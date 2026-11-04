# Debian Laptop Server

Documentación y configuración del servidor doméstico sobre portátil con Debian GNU/Linux.

---

## 1. Configuración del Sistema

### Gestión de Energía (Tapa cerrada)
Prevención de suspensión al cerrar la tapa en `/etc/systemd/logind.conf`:

```ini
HandleLidSwitch=ignore
HandleLidSwitchExternalPower=ignore
HandleLidSwitchDocked=ignore
```

Aplicar cambios y enmascarar objetivos de suspensión:

```bash
sudo systemctl restart systemd-logind
sudo systemctl mask sleep.target suspend.target hibernate.target hybrid-sleep.target
```

### Soporte de Terminal
Instalación de definiciones de terminal para Kitty por SSH:

```bash
sudo apt install -y kitty-terminfo
```

---

## 2. Seguridad y Endurecimiento (Hardening)

### Autenticación SSH por Clave Pública (ed25519)
Acceso por contraseña y login directo de `root` deshabilitados.

1. Generar la clave en el equipo cliente y copiarla al servidor:
   ```bash
   ssh-keygen -t ed25519 -C "servidor-debian"
   ssh-copy-id fran@portatil.lan
   ```

2. Configuración en `/etc/ssh/sshd_config.d/hardening.conf`:
   ```text
   PermitRootLogin no
   PasswordAuthentication no
   KbdInteractiveAuthentication no
   ```

3. Validar sintaxis y reiniciar SSH:
   ```bash
   sudo sshd -t && sudo systemctl restart sshd
   ```

### Cortafuegos (UFW)
Tráfico entrante restringido a la interfaz mesh de Tailscale:

```bash
sudo ufw default deny incoming
sudo ufw default allow outgoing
sudo ufw allow in on tailscale0 to any port 22 proto tcp
sudo ufw enable
```

### Actualizaciones Automáticas
Parches de seguridad gestionados con `unattended-upgrades`:

```bash
sudo apt update && sudo apt install -y unattended-upgrades apt-listchanges
sudo dpkg-reconfigure --priority=low unattended-upgrades
```

---

## 3. Infraestructura Docker

### Red Integrada (`proxy-net`)
Red `bridge` externa dedicada para la comunicación directa inter-contenedor sin exponer puertos innecesarios al host:

```bash
docker network create proxy-net
```

Todos los desplegables conectan a la red en sus respectivos `docker-compose.yml`:

```yaml
networks:
  default:
  proxy-net:
    external: true
```

---

## 4. Servicios y Proxy Inverso

Dominio base: `portatil-lan.duckdns.org`  
Certificado: **SSL Wildcard** (`*.portatil-lan.duckdns.org`) gestionado mediante DNS Challenge en Let's Encrypt (DuckDNS token).

| Servicio | Ubicación Local | Acceso HTTPS / Dirección |
| :--- | :--- | :--- |
| **Homepage** | `~/services/homepage/` | `https://home.portatil-lan.duckdns.org` |
| **Nginx Proxy Manager** | `~/services/npm/` | `https://npm.portatil-lan.duckdns.org` *(Rescate: `http://100.72.28.89:81`)* |
| **Pi-hole** | `~/services/pihole/` | `https://pihole.portatil-lan.duckdns.org/admin/` |
| **Portainer** | `~/services/portainer/` | `https://portainer.portatil-lan.duckdns.org` |
| **Uptime Kuma** | `~/services/uptime-kuma/` | `https://kuma.portatil-lan.duckdns.org` |

### Detalles de Configuración por Servicio

* **Nginx Proxy Manager (`~/services/npm/`)**
  * Punto de entrada HTTP (80) y HTTPS (443).
  * Proxy host para sí mismo configurado apuntando a `http://127.0.0.1:81`.
  * Opción *Force SSL* y *HTTP/2 Support* habilitados globalmente.

* **Pi-hole (`~/services/pihole/`)**
  * Servidor DNS primario para la red local.
  
