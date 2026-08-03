# Atualizações Automáticas com Unattended Upgrades

## Objetivo

Configurar atualizações automáticas no Ubuntu para manter o servidor recebendo correções de segurança sem depender de atualizações manuais.

---

## Instalação

Verificar se o pacote está instalado:

```bash
dpkg -l | grep unattended-upgrades
```

Caso não esteja instalado:

```bash
sudo apt install unattended-upgrades
```

---

## Habilitando atualizações automáticas

Executar:

```bash
sudo dpkg-reconfigure --priority=low unattended-upgrades
```

Selecionar:

```text
Yes
```

---

## Testando

Executar uma simulação:

```bash
sudo unattended-upgrades --dry-run --debug
```

Durante o teste foi identificado:

```text
System is on battery power, stopping
```

O Ubuntu bloqueia a execução quando o sistema está utilizando bateria para evitar que uma atualização seja interrompida por falta de energia.

Após conectar o carregador, a execução ocorreu normalmente.

---

## Verificando o agendamento

O Ubuntu utiliza timers do systemd para executar as atualizações automaticamente.

Verificar:

```bash
systemctl status apt-daily-upgrade.timer
```

Listar timers relacionados ao APT:

```bash
systemctl list-timers | grep apt
```

---

## Logs

Os registros das atualizações ficam em:

```bash
/var/log/unattended-upgrades/
```

---
