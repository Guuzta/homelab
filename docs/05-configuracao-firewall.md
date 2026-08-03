# Configuração do Firewall com UFW

## Objetivo

Configurar um firewall no servidor Ubuntu para controlar quais serviços podem ser acessados pela rede.

O objetivo dessa etapa foi entender como o firewall trabalha junto com os serviços do servidor, permitindo ou bloqueando conexões através de portas específicas.

---

# Conceito

Um servidor pode possuir vários serviços rodando ao mesmo tempo, como:

- SSH → porta 22
- HTTP (Nginx) → porta 80
- HTTPS → porta 443

Mesmo que um serviço esteja funcionando corretamente, ele não necessariamente estará acessível externamente.

O firewall atua como uma camada entre a rede e os serviços do servidor, decidindo quais conexões podem chegar até determinadas portas.

Exemplo:

```
Cliente
   |
   | Tentativa de conexão HTTP (porta 80)
   |
Firewall

   |
   |-- ALLOW → conexão permitida
   |
   |-- DENY → conexão bloqueada
   |
Serviço (Nginx)
```

---

# Instalação e verificação do UFW

O Ubuntu utiliza o **UFW (Uncomplicated Firewall)** como uma interface simplificada para gerenciamento das regras de firewall.

Verificar o status atual:

```bash
sudo ufw status
```

Caso esteja desativado:

```
Status: inactive
```

---

# Configuração do acesso SSH

Como o servidor é administrado remotamente através de SSH, a primeira regra adicionada foi permitindo o acesso pela porta 22.

```bash
sudo ufw allow OpenSSH
```

Também seria possível liberar diretamente:

```bash
sudo ufw allow 22/tcp
```

Essa etapa é importante para evitar perder o acesso remoto ao ativar o firewall.

---

# Ativando o firewall

Após adicionar as regras necessárias:

```bash
sudo ufw enable
```

Verificando as regras configuradas:

```bash
sudo ufw status
```

Resultado inicial:

```
To                         Action      From
--                         ------      ----
OpenSSH                    ALLOW       Anywhere
```

Neste momento, somente conexões SSH estavam permitidas.

---

# Teste utilizando o Nginx

Para entender o funcionamento do firewall, foi utilizado o Nginx como serviço de teste.

Primeiro foi verificado se o serviço estava funcionando:

```bash
sudo systemctl status nginx
```

O serviço continuou ativo normalmente.

---

## Bloqueando a porta HTTP

A porta padrão do HTTP é a porta 80.

Foi adicionada uma regra bloqueando essa porta:

```bash
sudo ufw deny 80/tcp
```

Verificando as regras:

```bash
sudo ufw status
```

Resultado:

```
To                         Action      From
--                         ------      ----
OpenSSH                    ALLOW       Anywhere
80/tcp                     DENY        Anywhere
OpenSSH (v6)               ALLOW       Anywhere (v6)
80/tcp (v6)                DENY        Anywhere (v6)
```

---

# Resultado do teste

Mesmo com a porta 80 bloqueada pelo firewall:

- O Nginx continuou em execução.
- O processo do Nginx não foi encerrado.
- O servidor continuou funcionando normalmente.

Porém:

- O acesso externo ao Nginx foi bloqueado.

Isso demonstra que o firewall não controla o funcionamento do serviço, mas sim o acesso de rede até ele.

---

# Liberando novamente o acesso HTTP

Removendo a regra de bloqueio:

```bash
sudo ufw delete deny 80/tcp
```

Liberando a porta:

```bash
sudo ufw allow 80/tcp
```

Ou utilizando o perfil do Nginx:

```bash
sudo ufw allow "Nginx HTTP"
```

Verificando novamente:

```bash
sudo ufw status
```

Resultado:

```
To                         Action      From
--                         ------      ----
OpenSSH                    ALLOW       Anywhere
80/tcp                     ALLOW       Anywhere
```

Agora conexões HTTP podem chegar até o Nginx.

---
