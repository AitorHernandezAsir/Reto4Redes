# Mapa Lógico de la Red - Reto 4

Este documento describe la topología lógica, el enrutamiento y los servicios desplegados en la infraestructura virtualizada sobre Proxmox y protegida mediante pfSense.

## Diagrama de Topología (Mermaid)

```mermaid
flowchart TD
    EXT["🌐 Red Externa Aula<br/>Subred: 172.26.44.0/24"]

    subgraph PFSENSE ["🛡️ Router / Firewall pfSense"]
        WAN["Interfaz WAN<br/>IP: 172.26.46.52"]
        NAT["Reglas NAT (Port Forwarding) & Política Deny-All<br/>• TCP 2221 → 192.168.2.101:22 (SSH server1)<br/>• TCP 2222 → 192.168.2.102:22 (SSH server2)<br/>• TCP 2223 → 192.168.2.103:22 (SSH server3)<br/>• TCP 2224 → 192.168.2.104:22 (SSH server4)<br/>• TCP/UDP 53 → 192.168.2.101:53 (DNS)<br/>• TCP 8280 → 192.168.2.103:80 (Web Apache2)<br/>• TCP 8281 → 192.168.2.103:8080 (Monitorix)"]
        LAN["Interfaz LAN (Gateway)<br/>IP: 192.168.2.1/24"]
        WAN --- NAT --- LAN
    end

    EXT <-->|Tráfico Filtrado| WAN

    subgraph PROXMOX ["🔀 Red Virtual Proxmox (192.168.2.0/24 | Dominio: red-reto4.lan)"]
        S1["🖥️ server1-gX<br/>IP: 192.168.2.101<br/>• DNS Maestro (BIND9)<br/>• DHCP Primario (isc-dhcp-server)"]
        S2["🖥️ server2-gX<br/>IP: 192.168.2.102<br/>• DNS Esclavo (BIND9)<br/>• DHCP Secundario (isc-dhcp-server)"]
        S3["🖥️ server3-gX<br/>IP: 192.168.2.103<br/>• Servidor Web (Apache2)<br/>• Monitorización (Monitorix)"]
        S4["🖥️ server4-gX<br/>IP: 192.168.2.104<br/>• Cliente de Pruebas<br/>• IP Fija por Reserva DHCP (MAC)"]
    end

    LAN <--> S1
    LAN <--> S2
    LAN <--> S3
    LAN <--> S4
    S1 <-->|Transferencia de Zona DNS| S2
```

## Resumen de Flujos de Red y Enrutamiento

1. **Acceso desde el Exterior (`172.26.44.0/24`):**
   - Todas las peticiones entran a través de la interfaz WAN de **pfSense** (`172.26.46.52`).
   - El firewall aplica una política restrictiva (*Deny-All* por defecto) y únicamente permite el paso a los puertos redirigidos explícitamente mediante reglas **NAT Port Forwarding**.
   - El servidor DNS (`server1-gX`) autoriza consultas recursivas a la subred externa mediante su ACL (`172.26.44.0/24`).

2. **Comunicaciones Internas (`192.168.2.0/24`):**
   - **Resolución de nombres:** `server1-gX` (maestro) replica automáticamente las zonas directa (`red-reto4.lan`) e inversa (`2.168.192.in-addr.arpa`) hacia `server2-gX` (esclavo).
   - **Asignación de direcciones:** El servicio DHCP de pfSense está deshabilitado; `server1-gX` y `server2-gX` gestionan el rango dinámico (`192.168.2.150` - `.199`) y asignan la reserva estática por MAC a `server4-gX` (`192.168.2.104`).
   - **Administración remota:** Todos los nodos aceptan conexiones SSH exclusivamente mediante autenticación por clave pública RSA de 4096 bits para los 5 usuarios autorizados.