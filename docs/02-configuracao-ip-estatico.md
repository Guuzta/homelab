# Configuração IP Estático no Ubuntu Server

## Por que configurar um IP estático?

Por padrão, dispositivos em uma rede doméstica recebem um endereço IP automaticamente através do `DHCP` do roteador. O problema é que esse endereço pode mudar após uma reinicialização do servidor ou do roteador.

> Exemplo:

```text
Antes:

Servidor → 192.168.18.159

Depois de uma reinicialização:

Servidor → 192.168.18.42
```

Essa mudança causa problemas principalmente em servidores, pois serviços e conexões dependem de saber onde encontrar aquela máquina.

No caso do homelab, o principal impacto é no acesso via SSH.

Sem um IP fixo, seria necessário descobrir novamente o endereço da máquina toda vez que ele mudasse:

```bash
ssh usuario@192.168.18.x
```

Com um IP estático, o servidor permanece sempre no mesmo endereço:

```bash
ssh usuario@192.168.18.10
```

## Configuração usando Netplan

O Ubuntu Server utiliza o Netplan para configurar interfaces de rede.

Arquivo utilizado:

```text
/etc/netplan/00-installer-config.yaml
```

### Configuração original utilizando DHCP:

```yaml
network:
  version: 2
  wifis:
    wlp3s0:
      dhcp4: true
      access-points:
        "NOME_DA_REDE":
          password: "SENHA_DA_REDE"
```

O parâmetro:

`dhcp4: true`

significa que o endereço IPv4 será entregue automaticamente pelo roteador.

### Configuração com IP estático

Exemplo:

```yaml
network:
  version: 2
  wifis:
    wlp3s0:
      dhcp4: false
      addresses:
        - 192.168.18.10/24
      routes:
        - to: default
          via: 192.168.18.1
      nameservers:
        addresses:
          - 1.1.1.1
          - 8.8.8.8
      access-points:
        "NOME_DA_REDE":
          password: "SENHA_DA_REDE"
```

Alterações realizadas:

- `dhcp4: false`
  - Desativa a atribuição automática de IP.
- `addresses`
  - Define o endereço fixo da máquina.
- `routes`
  - Define o gateway utilizado para acessar outras redes.
- `nameservers`
  - Define os servidores DNS responsáveis por traduzir nomes como google.com em endereços IP.

### Aplicando configuração

```bash
sudo netplan apply
```

### Validação

Verificar o IP:

```bash
ip a
```

Verificar a rota:

```bash
ip route
```

Testar conexão:

```bash
ping 8.8.8.8
```

## Resultado esperado

Após essa configuração, o servidor do homelab possui um endereço fixo dentro da rede:

```text
Ubuntu Server
      |
      |
192.168.18.10
```

Isso permite acessar serviços e administrar a máquina de forma previsível.
