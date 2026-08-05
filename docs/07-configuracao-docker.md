# Docker e Primeiro Serviço (Netdata)

## Objetivo

Instalar o Docker no servidor Ubuntu e utilizá-lo para executar o primeiro serviço do homelab, entendendo como os containers facilitam a instalação, isolamento e gerenciamento de aplicações.

---

## Instalação do Docker

Adicionar o repositório oficial do Docker e instalar os pacotes necessários.

Verificar a instalação:

```bash
docker --version
```

Verificar se o serviço está em execução:

```bash
sudo systemctl status docker
```

Testar o funcionamento:

```bash
docker run hello-world
```

---

## Configuração do usuário

Para utilizar o Docker sem `sudo`, adicionar o usuário ao grupo `docker`:

```bash
sudo usermod -aG docker $USER
```

Aplicar a alteração:

```bash
newgrp docker
```

---

## Instalação do Netdata

Foi utilizado o Netdata como primeiro serviço do homelab para monitoramento do servidor.

Exemplo de `docker-compose.yml`:

```yaml
services:
  netdata:
    image: netdata/netdata
    container_name: netdata
    restart: unless-stopped
    ports:
      - "19999:19999"
```

Iniciar o serviço:

```bash
docker compose up -d
```

Verificar os containers em execução:

```bash
docker ps
```

Acessar o dashboard:

```text
http://IP_DO_SERVIDOR:19999
```

---

## Observação sobre o Firewall

Durante os testes foi observado que, mesmo com apenas a porta do SSH liberada no UFW, o Netdata continuou acessível pela rede local.

Isso ocorre porque o Docker publica as portas utilizando regras próprias de rede, que não seguem o comportamento esperado apenas com as regras do UFW.

Esse comportamento será revisitado futuramente durante a configuração do Nginx como proxy reverso e no estudo da integração entre Docker e firewall.

---
