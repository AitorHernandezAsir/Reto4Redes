# Mapa Lógico de la Red - Reto 4

```mermaid
flowchart TD
    EXT["🌐 Red Externa Aula<br/>Subred: 172.26.44.0/24"]

    subgraph PFSENSE ["🛡️ Router / Firewall pfSense"]
        WAN["Interfaz WAN<br/>IP: 172.26.46.52"]
        NAT["Reglas NAT (Port Forwarding)<br/>• TCP 2221 → .101:22 (SSH server1)<br/>• TCP 2222 → .102:22 (SSH server2)<br/>• TCP 2223 → .103:22 (SSH server3)<br/>• TCP 2224 → .104:22 (SSH server4)<br/>• TCP/UDP 53 → .101:53 (DNS)<br/>• TCP 8280 → .103:80 (Web Apache2)<br/>• TCP 8281 → .103:8080 (Monitorix)"]
        LAN["Interfaz LAN (Gateway)<br/>IP: 192.168.2.1/24"]
        WAN --- NAT --- LAN
    end

    EXT <-->|Tráfico Filtrado / Deny-All| WAN

    subgraph PROXMOX ["🔀 Red Virtual Proxmox (192.168.2.0/24 | Dominio: red-reto4.lan)"]
        S1["🖥️ server1-gX<br/>IP: 192.168.2.101<br/>• DNS Maestro (BIND9)<br/>• DHCP Primario (isc-dhcp-server)"]
        S2["🖥️ server2-gX<br/>IP: 192.168.2.102<br/>• DNS Esclavo (BIND9)<br/>• DHCP Secundario (isc-dhcp-server)"]
        S3["🖥️ server3-gX<br/>IP: 192.168.2.103<br/>• Servidor Web (Apache2)<br/>• Monitorix"]
        S4["🖥️ server4-gX<br/>IP: 192.168.2.104<br/>• Cliente Pruebas<br/>• IP Fija por Reserva DHCP (MAC)"]
    end

    LAN <--> S1
    LAN <--> S2
    LAN <--> S3
    LAN <--> S4
    S1 <-->|Transferencia de Zona| S2
