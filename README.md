# Debian Laptop Server

Documentación y configuraciones del servidor doméstico en portátil con Debian.

## Configuración del Sistema

### 1. Gestión de Energía (Tapa cerrada)
Configurado `/etc/systemd/logind.conf` para evitar la suspensión al cerrar la tapa:

```ini
HandleLidSwitch=ignore
HandleLidSwitchExternalPower=ignore
HandleLidSwitchDocked=ignore
```

Comandos aplicados:

```bash
sudo systemctl restart systemd-logind
sudo systemctl mask sleep.target suspend.target hibernate.target hybrid-sleep.target
```

### 2. Paquetes y SSH
Soporte para terminal Kitty por SSH:

```bash
sudo apt install -y kitty-terminfo
```

## Seguridad y Endurecimiento (Hardening)

### 1. Autenticación SSH por Clave Pública (ed25519)
Se deshabilitó el login con contraseña y el acceso directo al usuario root.

1. Generar la clave en el equipo cliente y copiarla al servidor:

```bash
ssh-keygen -t ed25519 -C "servidor-debian"
ssh-copy-id fran@portatil.lan
```

2. Crear la configuración de hardening en `/etc/ssh/sshd_config.d/hardening.conf`:

```text
PermitRootLogin no
PasswordAuthentication no
KbdInteractiveAuthentication no
```

3. Validar sintaxis y reiniciar el servicio:

```bash
sudo sshd -t
sudo systemctl restart sshd
```

### 2. Cortafuegos (UFW)
Aislamiento de tráfico para permitir únicamente conexiones desde la red mesh de Tailscale:

```bash
sudo ufw default deny incoming
sudo ufw default allow outgoing
sudo ufw allow in on tailscale0 to any port 22 proto tcp
sudo ufw enable
```

### 3. Actualizaciones Automáticas (unattended-upgrades)
Instalación de parches de seguridad automatizados en Debian:

```bash
sudo apt update && sudo apt install -y unattended-upgrades apt-listchanges
sudo dpkg-reconfigure --priority=low unattended-upgrades
```

## Docker
Instalado desde el repositorio oficial de Docker CE (`docker-ce`, `docker-compose-plugin`).

## Servicios

### Pi-hole (DNS y AdBlock)
* Ubicación local: `~/services/pihole/`

Despliegue:

```bash
cd ~/services/pihole
cp .env.example .env
docker compose up -d
```
