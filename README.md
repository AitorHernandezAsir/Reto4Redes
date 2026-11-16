# Reto 4: Configuración de Infraestructura de Red y Servicios Centralizados

Este repositorio contiene la documentación, los archivos de configuración, scripts y esquemas lógicos implementados para la puesta en marcha de una infraestructura de red multinodo sobre Proxmox, gestionada y asegurada mediante un cortafuegos **pfSense**.

---

## 📌 Visión General del Proyecto

El objetivo principal de este proyecto es desplegar una red interna segmentada que ofrece servicios centralizados esenciales (**DNS**, **DHCP** y **Web**) garantizando un control de acceso estricto a través del router pfSense y autenticación segura por claves SSH.

### Componentes Clave:
* **Router / Firewall (pfSense):** Gestión del perímetro de red, enrutamiento, políticas restrictivas (*Deny All*) y reenvío de puertos (NAT/Port Forwarding).
* **Servidores Linux (Ubuntu Server 22.04 LTS):** Despliegue de nodos para la distribución de servicios esenciales (`server1` a `server4`).
* **DNS Centralizado (BIND9):** Arquitectura Maestro-Esclavo para resolución de nombres directa e inversa.
* **DHCP Alta Disponibilidad (ISC-DHCP-Server):** Asignación dinámica de direccionamiento e IPs fijas por dirección MAC.
* **Servidor Web y Monitorización (Apache2 + Monitorix):** Aloja la plataforma web del reto y métricas del sistema expuestas a través de NAT.
* **Seguridad SSH:** Autenticación por pares de claves RSA de 4096 bits para 5 usuarios con permisos estrictos.

---

## 💻 Tabla de Direccionamiento IP y Servicios

| Equipo / Nodo | Interfaz / Rol | Dirección IP (LAN) | IP Externa / NAT (WAN) | Servicios Alojados |
| :--- | :--- | :--- | :--- | :--- |
| **pfSense** | Gateway / WAN / LAN | `192.168.2.1` | `172.26.46.52` | Firewall, NAT, Routing |
| **server1** | DNS Maestro / DHCP 1 | `192.168.2.101` | Port Forwarding 2221 (SSH), 53 (DNS) | BIND9 (Master), ISC-DHCP-Server |
| **server2** | DNS Esclavo / DHCP 2 | `192.168.2.102` | Port Forwarding 2222 (SSH) | BIND9 (Slave), ISC-DHCP-Server |
| **server3** | Servidor Web / Monitorix | `192.168.2.103` | Port Forwarding 2223 (SSH), 8280 (HTTP), 8281 (Monitorix) | Apache2, Monitorix |
| **server4** | Cliente / Nodo Dinámico | `192.168.2.104` | Port Forwarding 2224 (SSH) | Reserva DHCP por MAC (`bc:24:11:b1:8e:87`) |

---

## 🗺️ Mapa Lógico de la Red

Below is the logical structure of the environment implemented for Reto 4:

```text
                                  [ RED DEL CENTRO / WAN ]
                                      (172.26.46.x/24)
                                             │
                                             ▼
                                  ┌─────────────────────┐
                                  │   Router pfSense    │
                                  │   LAN: 192.168.2.1  │
                                  └──────────┬──────────┘
                                             │
                                  [ RED INTERNA / LAN ]
                                    (192.168.2.0/24)
                                             │
      ┌───────────────────┬──────────────────┴───────────────────┬───────────────────┐
      │                   │                                      │                   │
┌─────┴─────────────┐ ┌───┴───────────────┐              ┌───────┴───────────┐ ┌─────┴─────────────┐
│     server1       │ │     server2       │              │     server3       │ │     server4       │
│  192.168.2.101    │ │  192.168.2.102    │              │  192.168.2.103    │ │  192.168.2.104    │
├───────────────────┤ ├───────────────────┤              ├───────────────────┤ ├───────────────────┤
│ • BIND9 (Master)  │ │ • BIND9 (Slave)   │              │ • Apache2 Web     │ │ • DHCP Client     │
│ • ISC-DHCP        │ │ • ISC-DHCP        │              │ • Monitorix       │ │ • IP Fija por MAC │
└───────────────────┘ └───────────────────┘              └───────────────────┘ └───────────────────┘
```

---

## 📁 Estructura del Repositorio

```text
.
├── README.md               # Descripción general, matriz de IPs y esquema de red
├── gestion.md              # Organización de tareas y reparto entre colaboradores
├── .gitignore              # Filtro de seguridad (excluye claves privadas y datos sensibles)
├── informe/                # Memoria técnica completa en formato Markdown y capturas
├── config/                 # Ficheros de configuración originales del sistema
│   ├── netplan/            # Configuraciones YAML de red
│   ├── bind/               # Zonas directas e inversas de BIND9
│   ├── dhcp/               # Fichero dhcpd.conf
│   └── apache/             # VirtualHosts y puertos de Apache
├── scripts/                # Scripts de automatización y comprobación de red
└── diagramas/              # Esquemas y diagramas de topología de red
```

---

## 🔒 Seguridad e Identidad SSH

El acceso de administración remota a todos los nodos de la red está restringido y configurado mediante pares de claves RSA privadas de 4096 bits asociadas a los integrantes del equipo:
* `clave_aitor`
* `clave_gari`
* `clave_antonio`
* `clave_serrato`
* `clave_endika`

*Las claves privadas han sido excluidas intencionadamente del control de versiones mediante `.gitignore`.*
