# Debian Laptop Server

Documentación y configuraciones del servidor doméstico en portátil con Debian.

## 💻 Configuración del Sistema

### 1. Gestión de Energía (Tapa cerrada)
Configurado `/etc/systemd/logind.conf` para evitar la suspensión al cerrar la tapa.

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
- Soporte para terminal Kitty por SSH:
  ```bash
  sudo apt install -y kitty-terminfo
  ```

---

## 🐳 Docker

Instalado desde el repositorio oficial de Docker CE (`docker-ce`, `docker-compose-plugin`).

---

## 🚀 Servicios

### Pi-hole (DNS & AdBlock)
- **Ubicación local:** `~/services/pihole/`
- **Despliegue:**
  ```bash
  cd ~/services/pihole
  cp .env.example .env
  docker compose up -d
  ```
