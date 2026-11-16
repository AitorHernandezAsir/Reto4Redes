graph TD
    EXT["🌐 Red Externa Aula<br/>172.26.44.0/24"] -->|Tráfico Entrante| FW["🛡️ Router Firewall pfSense<br/>WAN: 172.26.46.52 | LAN: 192.168.2.1<br/>NAT: 2221-2224 (SSH) | 53 (DNS) | 8280 (Web) | 8281 (Monitorix)"]
    
    FW --- SW["🔀 Red Virtual Proxmox (192.168.2.0/24)<br/>Dominio: red-reto4.lan"]
    
    SW --- S1["🖥️ server1-gX (192.168.2.101)<br/>DNS Maestro (BIND9)<br/>DHCP Primario"]
    SW --- S2["🖥️ server2-gX (192.168.2.102)<br/>DNS Esclavo (BIND9)<br/>DHCP Secundario"]
    SW --- S3["🖥️ server3-gX (192.168.2.103)<br/>Servidor Web (Apache2)<br/>Monitorix"]
    SW --- S4["🖥️ server4-gX (192.168.2.104)<br/>Cliente Pruebas<br/>IP Fija por DHCP (MAC)"]
