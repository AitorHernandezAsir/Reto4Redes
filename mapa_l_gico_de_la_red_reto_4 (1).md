# Mapa Lógico de la Red - Reto 4

Este documento describe la topología lógica, el enrutamiento y los servicios desplegados en la infraestructura virtualizada sobre Proxmox y protegida mediante pfSense.

## Diagrama de Topología (Mermaid - Horizontal)

```mermaid
flowchart LR
    EXT["🌐 Red Externa Aula<br/>172.26.44.0/24"] -->|Tráfico Filtrado| WAN

    subgraph PFSENSE ["🛡️ Router / Firewall pfSense"]
        direction TB
        WAN["Interfaz WAN<br/>IP: 172.26.46.52"]
        NAT["Reglas NAT & Deny-All<br/>• 2221-2224 → SSH (.101-.104)<br/>• 53 → DNS (.101)<br/>• 8280 → Web (.103:80)<br/>• 8281 → Monitorix (.103:8080)"]
        LAN["Interfaz LAN<br/>IP: 192.168.2.1"]
        WAN --- NAT --- LAN
    end

    LAN ==> SW["🔀 Switch Virtual / Red LAN<br/>192.168.2.0/24 (red-reto4.lan)"]

    SW --> S1
    SW --> S2
    SW --> S3
    SW --> S4

    subgraph PROXMOX ["🖥️ Nodos Red Proxmox"]
        direction TB
        S1["server1-gX (.101)<br/>• DNS Maestro<br/>• DHCP Primario"]
        S2["server2-gX (.102)<br/>• DNS Esclavo<br/>• DHCP Secundario"]
        S3["server3-gX (.103)<br/>• Web (Apache2)<br/>• Monitorix"]
        S4["server4-gX (.104)<br/>• Cliente Pruebas<br/>• IP Fija por MAC"]
    end

    S1 <-->|Transferencia Zona| S2
```

## Resumen de Flujos de Red y Enrutamiento

1. **Acceso desde el Exterior (`172.26.44.0/24`):**
   * Las peticiones entran a través de la interfaz WAN de **pfSense** (`172.26.46.52`).
   * El firewall aplica una política restrictiva (*Deny-All*) y permite el acceso mediante **NAT Port Forwarding**.
   * El DNS (`server1-gX`) procesa las consultas desde la red externa gracias a la ACL ampliada.

2. **Comunicaciones Internas (`192.168.2.0/24`):**
   * **Resolución de nombres:** `server1-gX` (maestro) sincroniza las zonas directa e inversa con `server2-gX` (esclavo).
   * **Asignación de IP:** DHCP gestionado por los servidores 1 y 2 con reserva estática por MAC para el servidor 4.
   * **Acceso SSH:** Restringido mediante pares de claves RSA de 4096 bits para los 5 usuarios autorizados.