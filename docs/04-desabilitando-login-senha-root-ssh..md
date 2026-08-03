# Desabilitando login por senha e acesso Root via SSH

Após configurar o acesso via chave SSH, realizei algumas alterações para aumentar a segurança do servidor.

## Desabilitar autenticação por senha

Editei a configuração do OpenSSH:

```bash
sudo nano /etc/ssh/sshd_config
```

Alterei:

```text
PasswordAuthentication no
```

### O que isso faz?

Com essa configuração, o servidor deixa de aceitar autenticação por senha.

Antes:

- ✔️ Login com senha
- ✔️ Login com chave SSH

Depois:

- ❌ Login com senha
- ✔️ Apenas login utilizando chave SSH

---

## Desabilitar login do usuário root

No mesmo arquivo de configuração, alterei:

```text
PermitRootLogin no
```

### O que isso faz?

Impede que o usuário `root` faça login diretamente via SSH.

## Aplicando as alterações

Após modificar a configuração:

Verificar se a configuração está válida:

```bash
sudo sshd -t
```

Reiniciar o serviço SSH:

```bash
sudo systemctl restart ssh
```

Verificar se o serviço está ativo:

```bash
sudo systemctl status ssh
```

---

## Observação importante

Durante essa etapa percebi que minhas alterações não estavam sendo aplicadas.

Descobri que o arquivo:

```text
/etc/ssh/sshd_config.d/50-cloud-init.conf
```

estava sobrescrevendo parte da configuração definida em:

```text
/etc/ssh/sshd_config
```

Isso acontece porque o arquivo principal do SSH carrega automaticamente outros arquivos de configuração que ficam dentro do diretório:

```text
Include /etc/ssh/sshd_config.d/*.conf
```

Ou seja, além do arquivo principal, o OpenSSH também carrega todos os arquivos presentes em `sshd_config.d/`, e configurações desses arquivos podem substituir configurações anteriores.

Para verificar qual configuração estava sendo realmente utilizada pelo serviço SSH, utilizei:

```bash
sudo sshd -T | grep passwordauthentication
```

Esse comando mostra o valor final aplicado pelo OpenSSH após carregar todos os arquivos de configuração.

Depois, para identificar em quais arquivos essa configuração estava definida, utilizei:

```bash
sudo grep -R "PasswordAuthentication" /etc/ssh/
```

Com isso, foi possível identificar que o arquivo:

```bash
/etc/ssh/sshd_config.d/50-cloud-init.conf
```

possuía uma configuração que estava sobrescrevendo o valor definido anteriormente.

Após identificar o arquivo responsável pela sobrescrita, ajustei a configuração no local correto e validei:

```bash
sudo sshd -t
```

Depois reiniciei o serviço SSH:

```bash
sudo systemctl restart ssh
```

Após a correção, as configurações passaram a ser aplicadas corretamente:

```text
PasswordAuthentication no
PermitRootLogin no
```

---
