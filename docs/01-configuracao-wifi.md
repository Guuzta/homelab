# Configuração do WiFi no Ubuntu Server

Após a instalação do Ubuntu Server em um notebook para utilização como homelab, o próximo passo foi configurar a conexão WiFi. Como o equipamento não possui entrada Ethernet, a conexão sem fio foi configurada utilizando a interface WiFi integrada.

## Identificação da interface WiFi

Primeiramente foi verificada a interface de rede disponível:

```bash
ip a
```

A interface encontrada foi:

```text
wlp3s0
```

## Configuração do Netplan

O Ubuntu Server utiliza o Netplan para gerenciamento da rede.

O arquivo de configuração foi editado:

```bash
sudo nano /etc/netplan/00-installer-config.yaml
```

Configuração utilizada:

```yaml
network:
  version: 2
  WiFis:
    wlp3s0:
      dhcp4: true
      access-points:
        "NOME_DA_REDE":
          password: "SENHA_DA_REDE"
```

> **Observação**:

O campo `match` utilizando `macaddress` foi removido, pois causava incompatibilidade com a configuração WiFi:

```text
network backend does not support WiFi with match
```

A interface foi configurada diretamente pelo nome `wlp3s0`.

## Problema encontrado: RF-KILL

Ao tentar ativar a interface WiFi manualmente:

```bash
sudo ip link set wlp3s0 up
```

o sistema retornou:

```text
Operation not possible due to RF-KILL
```

## Entendimento do problema

O RF-KILL é um mecanismo do Linux responsável por bloquear ou liberar dispositivos de rádio, como:

- WiFi
- Bluetooth
- Redes móveis

Neste caso, o problema não estava relacionado a:

- senha do WiFi
- roteador
- configuração DHCP
- Netplan

A placa WiFi foi detectada corretamente, porém o RF-KILL estava bloqueado.

## Tentativa de desbloqueio

Foi realizada uma tentativa de alterar o estado do RF-KILL:

```bash
echo 1 | sudo tee /sys/class/rfkill/rfkill0/state
```

Após isso, o sistema foi reiniciado:

```bash
sudo reboot
```

Resultado

Após o reinício do sistema, a interface WiFi foi ativada corretamente.

A máquina conseguiu:

1. Ativar a interface `wlp3s0`
2. Conectar ao ponto de acesso WiFi
3. Receber um endereço IP via DHCP
4. Estabelecer conexão com a rede

Verificação:

```bash
ip a
```

e:

```bash
ip route
```

## Aprendizado

O problema mostrou a importância de diagnosticar problemas de rede seguindo uma ordem de camadas:

1. Verificar se o hardware foi reconhecido
2. Verificar se a interface existe
3. Verificar se a interface está ativa
4. Verificar bloqueios de hardware/software (RF-KILL)
5. Configurar conexão WiFi
6. Validar DHCP e conectividade

Nem todo problema de rede está relacionado ao roteador ou à configuração IP. Às vezes o dispositivo sequer chegou na etapa de comunicação com a rede.
