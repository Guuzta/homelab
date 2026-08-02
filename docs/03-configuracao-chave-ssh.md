# Configuração de chave SSH

## Objetivo

Configurar autenticação SSH utilizando chave pública/privada para acessar o servidor sem depender de senha.

Nesse cenário:

- PC principal → cliente SSH
- Notebook Ubuntu Server → servidor SSH

---

## Como funciona

O SSH utiliza um par de chaves:

- **Chave privada:** fica no computador principal e nunca deve ser compartilhada.
- **Chave pública:** é enviada para o servidor e adicionada ao arquivo `authorized_keys`.

Durante a conexão, o servidor verifica se o computador principal possui a chave privada correspondente à chave pública cadastrada.

A chave privada nunca é enviada pela rede.

---

## Criando uma chave SSH

Foi criada uma chave exclusiva para o homelab:

```bash
ssh-keygen -t ed25519 -f .ssh/id_ed25519_homelab
```

Arquivos gerados:

```
~/.ssh/id_ed25519_homelab
~/.ssh/id_ed25519_homelab.pub
```

---

## Enviando a chave pública para o servidor

Comando utilizado:

```bash
ssh-copy-id -i ~/.ssh/id_ed25519_homelab.pub usuario@ip-do-servidor
```

A chave pública foi adicionada ao servidor em:

```
~/.ssh/authorized_keys
```

---

## Testando conexão

Acesso utilizando a chave privada:

```bash
ssh usuario@ip-do-servidor
```

Após a configuração, o acesso ao servidor foi realizado utilizando autenticação por chave SSH.

---
