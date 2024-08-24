
# Configuración de VPN con WireGuard y Pi-hole
![Banner](https://github.com/oswaldom-code/docker-resources/docker/vpn/assets/banner.png)
El este `docker-compose.yml` configuramos una VPN utilizando [WireGuard](https://www.wireguard.com/) para el tunelado seguro y [Pi-hole](https://pi-hole.net/) como un bloqueador de anuncios a nivel de red. Esta configuración permite proteger tu privacidad mientras navegas y bloquea anuncios no deseados.


## 🛠️ Servicios

### 1. WireGuard

WireGuard es un protocolo VPN ligero y de alto rendimiento. En esta configuración, se encarga de manejar las conexiones VPN.

- **Imagen:** `linuxserver/wireguard`
- **Nombre del contenedor:** `wireguard`
- **Red:** `vpn_networks`, con la IP fija `172.21.0.6`
- **Puertos expuestos:** `51820/udp` (puerto de WireGuard)
- **Volúmenes:**
  - `/root/wireguard:/config`: Directorio de configuración de WireGuard.
  - `/lib/modules:/lib/modules`: Necesario para cargar módulos del kernel.
  - `/usr/src:/usr/src`: Necesario para compilar módulos del kernel si es necesario.
- **Variables de entorno:**
  - `PUID` y `PGID`: IDs de usuario y grupo para el contenedor.
  - `TZ`: Zona horaria.
  - `SERVERURL`: Dirección IP del servidor anfitrión.
  - `SERVERPORT`: Puerto en el que escuchará WireGuard (por defecto `51820`).
  - `PEERS`: Número de pares (clientes) que se configurarán automáticamente.
  - `PEERDNS`: DNS para los clientes (por defecto `auto`).
  - `INTERNAL_SUBNET`: Subred interna que usará WireGuard.

### 2. Pi-hole

Pi-hole es un bloqueador de anuncios a nivel DNS que actúa como servidor DNS y DHCP opcional para tu red.

- **Imagen:** `pihole/pihole`
- **Nombre del contenedor:** `pihole`
- **Red:** `vpn_networks`, con la IP fija `172.21.0.7`
- **Puertos expuestos:**
  - `8080:80`: Interfaz web de Pi-hole.
  - `442:443`: Interfaz segura (HTTPS).
  - Puertos expuestos adicionales:
    - `53`: DNS
    - `67`: DHCP
    - `80`: HTTP
    - `443`: HTTPS
- **Volúmenes:**
  - `./etc-pihole/:/etc/pihole/`: Configuración de Pi-hole.
  - `./etc-dnsmasq.d/:/etc/dnsmasq.d/`: Configuración de dnsmasq.
- **Variables de entorno:**
  - `TZ`: Zona horaria.
  - `WEBPASSWORD`: Contraseña para acceder a la interfaz web de Pi-hole.

## 🔧 Configuración de Red

La red `vpn_networks` se utiliza para la comunicación entre los contenedores y asigna IPs estáticas a cada uno.

- **Subred:** `172.21.0.0/24`

## 🚀 Cómo Levantar la VPN

1. **Configura las variables de entorno**:
   Asegúrate de definir las variables `HOST_SERVER_IP` y `PIHOLE_PASSWORD` en un archivo `.env` en el mismo directorio que el archivo `docker-compose.yml`.

   Ejemplo de archivo `.env`:
   ```
   HOST_SERVER_IP=your.server.ip
   PIHOLE_PASSWORD=your-pihole-password
   ```

2. **Levanta los servicios**:
   ```bash
   sudo docker-compose up -d
   ```

4. **Accede a Pi-hole**: Una vez levantado el servicio, puedes acceder a la interfaz web de Pi-hole desde tu navegador en `http://your.server.ip:8080` usando la contraseña definida en `PIHOLE_PASSWORD`.

## 🛑 Parar y Remover

Para detener los contenedores, ejecuta:
```bash
sudo docker-compose down
```

Esto parará y eliminará los contenedores pero conservará las configuraciones en los volúmenes montados.

## 📝 Notas Adicionales

- Asegúrate de que el puerto `51820/udp` esté abierto en tu firewall para permitir el tráfico de WireGuard.
- Puedes ajustar las configuraciones de los servicios editando el archivo `docker-compose.yml` según tus necesidades.
