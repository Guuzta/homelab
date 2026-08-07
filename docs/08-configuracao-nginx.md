# Nginx — Proxy Reverso

## Objetivo

Utilizar o Nginx como ponto de entrada dos serviços do homelab, permitindo controlar quais serviços podem ser acessados e como eles serão encaminhados.

## Estrutura

```
homelab/
└── services/
    ├── netdata/
    │   └── docker-compose.yml
    │
    └── nginx/
        ├── docker-compose.yml
        └── conf.d/
            └── default.conf
```

## Rede Docker

Foi criada uma rede compartilhada entre os serviços:

```bash
docker network create homelab
```

Os containers do Nginx e Netdata utilizam essa rede para se comunicarem internamente.

```yaml
networks:
  homelab:
    external: true
```

## Netdata

A porta do Netdata deixou de ser publicada diretamente:

```yaml
# Removido
ports:
  - "19999:19999"
```

Dessa forma, o Netdata não fica diretamente exposto na rede. O acesso passa pelo Nginx.

## Nginx

O Nginx publica a porta 80:

```yaml
ports:
  - "80:80"
```

A configuração do proxy reverso foi definida em:

```yaml
conf.d/default.conf
```

Configuração:

```yaml
server {
    listen 80;

    location /netdata/ {
        proxy_pass http://netdata:19999/;
    }
}
```

## Funcionamento

Ao acessar:

```
http://IP_DO_SERVIDOR/netdata/
```

O fluxo da requisição é:

```
Cliente
   |
   ▼
Nginx :80
   |
   ▼
netdata:19999
   |
   ▼
Netdata
```

O Nginx utiliza o nome `netdata` para localizar o container através da rede Docker compartilhada.

## Controle de acesso

Uma das principais vantagens dessa arquitetura é poder controlar o acesso aos serviços através do Nginx.

Uma rota pode:

- permitir acesso;
- bloquear completamente o serviço;
- permitir apenas determinados endereços ou redes;
- encaminhar a requisição para um serviço específico.

Exemplo permitindo acesso somente à rede local:

```yaml
location /netdata/ {
    allow 192.168.18.0/24;
    deny all;

    proxy_pass http://netdata:19999/;
}
```

## Observação sobre `location` e `proxy_pass`

A barra `/` no final possui importância no comportamento do `proxy_pass`.

Foi utilizado:

```yaml
location /netdata/ {
    proxy_pass http://netdata:19999/;
}
```

Isso permite que o prefixo `/netdata/` seja removido antes da requisição ser encaminhada ao Netdata.

## Aprendizados

- Nginx pode atuar como proxy reverso.
- Serviços podem permanecer internos na rede Docker.
- O Nginx pode funcionar como ponto central de entrada.
- `location` define quais caminhos serão atendidos.
- `proxy_pass` define para qual serviço a requisição será encaminhada.
- Containers podem se comunicar utilizando os nomes dos serviços dentro de uma rede Docker.
- O acesso aos serviços pode ser controlado de forma mais granular através do Nginx.