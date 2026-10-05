# 3. Plan de direccionamiento IP

## 3.1 Introducción

El direccionamiento IP de la infraestructura se ha diseñado utilizando
IPv4 y diferentes rangos privados para separar las redes LAN de las
diferentes sedes.

En la sede de Tortosa se utiliza segmentación mediante VLAN, mientras
que las sedes de Amposta, Roquetes y L'Aldea disponen de una red LAN
independiente.

Los enlaces entre routers utilizan redes /30, adecuadas para
conexiones punto a punto.

---

## 3.2 Direccionamiento de la sede de Tortosa

La sede de Tortosa utiliza la red `192.168.1.0/24`, dividida en
diferentes subredes /26 para separar los distintos tipos de tráfico.

| VLAN | Nombre | Red | Máscara | Gateway |
|---|---|---|---|---|
| 10 | ADMIN | 192.168.1.0/26 | 255.255.255.192 | 192.168.1.1 |
| 20 | USERS | 192.168.1.64/26 | 255.255.255.192 | 192.168.1.65 |
| 30 | SERVERS | 192.168.1.128/26 | 255.255.255.192 | 192.168.1.129 |
| 40 | GUESTS | 192.168.1.192/26 | 255.255.255.192 | 192.168.1.193 |
| 50 | VOICE | - | - | - |
| 99 | NATIVE | - | - | - |

Las VLAN 50 y 99 se utilizan para funciones específicas de switching.
La VLAN 50 está destinada al tráfico de voz y la VLAN 99 se utiliza
como VLAN nativa de los enlaces trunk.

---

## 3.3 Direccionamiento de la sede de Amposta

La sede de Amposta utiliza una única red LAN:

| Sede | Red | Máscara | Gateway |
|---|---|---|---|
| Amposta | 192.168.10.0/24 | 255.255.255.0 | 192.168.10.1 |

El router `RTR-AMP` actúa como gateway de la red y proporciona
direccionamiento DHCP a los equipos de la sede.

---

## 3.4 Direccionamiento de la sede de Roquetes

La sede de Roquetes utiliza la siguiente red LAN:

| Sede | Red | Máscara | Gateway |
|---|---|---|---|
| Roquetes | 172.16.1.0/24 | 255.255.255.0 | 172.16.1.1 |

El gateway de la red LAN es la interfaz `G0/0/2` del router
`RTR-ROQ`.

---

## 3.5 Direccionamiento de la sede de L'Aldea

La sede de L'Aldea utiliza la siguiente red LAN:

| Sede | Red | Máscara | Gateway |
|---|---|---|---|
| L'Aldea | 172.16.0.0/24 | 255.255.255.0 | 172.16.0.1 |

El gateway de la red LAN es la interfaz `G0/0/2` del router
`RTR-ALD`.

---

## 3.6 Direccionamiento de los enlaces WAN

Los enlaces entre routers utilizan subredes /30.

Cada enlace dispone de dos direcciones IP utilizables, una para cada
extremo del enlace.

| Red | Router 1 | IP | Router 2 | IP |
|---|---|---|---|---|
| 209.165.200.0/30 | RTR-TOR | 209.165.200.1 | RTR-AMP | 209.165.200.2 |
| 209.165.200.4/30 | RTR-TOR | 209.165.200.5 | RTR-ALD | 209.165.200.6 |
| 209.165.200.8/30 | RTR-ROQ | 209.165.200.9 | RTR-ALD | 209.165.200.10 |
| 209.165.200.12/30 | RTR-ROQ | 209.165.200.13 | RTR-AMP | 209.165.200.14 |

La distribución de los enlaces forma una topología con caminos
alternativos entre las diferentes sedes.

---

## 3.7 Direccionamiento de los routers

### RTR-TOR

| Interfaz | Dirección IP | Función |
|---|---|---|
| G0/1/0.10 | 192.168.1.1/26 | Gateway VLAN 10 |
| G0/1/0.20 | 192.168.1.65/26 | Gateway VLAN 20 |
| G0/1/0.30 | 192.168.1.129/26 | Gateway VLAN 30 |
| G0/1/0.40 | 192.168.1.193/26 | Gateway VLAN 40 |
| G0/0/1 | 209.165.200.1/30 | Enlace con RTR-AMP |
| G0/0/0 | 209.165.200.5/30 | Enlace con RTR-ALD |

### RTR-AMP

| Interfaz | Dirección IP | Función |
|---|---|---|
| G0/0/2 | 192.168.10.1/24 | Gateway LAN Amposta |
| G0/0/1 | 209.165.200.2/30 | Enlace con RTR-TOR |
| G0/0/0 | 209.165.200.14/30 | Enlace con RTR-ROQ |

### RTR-ROQ

| Interfaz | Dirección IP | Función |
|---|---|---|
| G0/0/2 | 172.16.1.1/24 | Gateway LAN Roquetes |
| G0/0/0 | 209.165.200.13/30 | Enlace con RTR-AMP |
| G0/0/1 | 209.165.200.9/30 | Enlace con RTR-ALD |

### RTR-ALD

| Interfaz | Dirección IP | Función |
|---|---|---|
| G0/0/2 | 172.16.0.1/24 | Gateway LAN L'Aldea |
| G0/0/0 | 209.165.200.6/30 | Enlace con RTR-TOR |
| G0/0/1 | 209.165.200.10/30 | Enlace con RTR-ROQ |

---

## 3.8 Direccionamiento de los equipos

### Sede de Tortosa

| Equipo | Dirección IP | Máscara | Gateway |
|---|---|---|---|
| PC-TOR-ADM01 | 192.168.1.10 | /26 | 192.168.1.1 |
| PC-TOR-ADM02 | 192.168.1.11 | /26 | 192.168.1.1 |
| PC-TOR-USR01 | 192.168.1.74 | /26 | 192.168.1.65 |
| DHCP-SERVER | 192.168.1.130 | /26 | 192.168.1.129 |
| PC-TOR-GST01 | 192.168.1.202 | /26 | 192.168.1.193 |

### Sede de Amposta

| Equipo | Dirección IP | Máscara | Gateway |
|---|---|---|---|
| PC-AMP-USR01 | 192.168.10.10 | /24 | 192.168.10.1 |
| PC-AMP-USR02 | 192.168.10.11 | /24 | 192.168.10.1 |
| PC-AMP-USR03 | 192.168.10.12 | /24 | 192.168.10.1 |
| PC-AMP-USR04 | 192.168.10.13 | /24 | 192.168.10.1 |

### Sede de Roquetes

| Equipo | Dirección IP | Máscara | Gateway |
|---|---|---|---|
| PC-ROQ-USR01 | 172.16.1.10 | /24 | 172.16.1.1 |
| PC-ROQ-USR02 | 172.16.1.11 | /24 | 172.16.1.1 |

### Sede de L'Aldea

| Equipo | Dirección IP | Máscara | Gateway |
|---|---|---|---|
| PC-ALD-USR01 | 172.16.0.10 | /24 | 172.16.0.1 |
| PC-ALD-USR02 | 172.16.0.11 | /24 | 172.16.0.1 |

---

## 3.9 Servidor DHCP

El servidor DHCP de la sede de Tortosa se encuentra dentro de la
VLAN 30 (SERVERS):

- Dirección IP: `192.168.1.130`
- Máscara: `/26`
- Gateway: `192.168.1.129`

El servidor proporciona direccionamiento dinámico a las VLAN de
Tortosa que utilizan DHCP.

La VLAN 30 utiliza direccionamiento estático para el servidor y no
depende del servicio DHCP para su funcionamiento.

---

## 3.10 Resumen del direccionamiento

| Zona | Red | Uso |
|---|---|---|
| Tortosa VLAN 10 | 192.168.1.0/26 | Administración |
| Tortosa VLAN 20 | 192.168.1.64/26 | Usuarios |
| Tortosa VLAN 30 | 192.168.1.128/26 | Servidores |
| Tortosa VLAN 40 | 192.168.1.192/26 | Invitados |
| Amposta | 192.168.10.0/24 | Usuarios |
| Roquetes | 172.16.1.0/24 | Usuarios |
| L'Aldea | 172.16.0.0/24 | Usuarios |
| WAN 1 | 209.165.200.0/30 | RTR-TOR ↔ RTR-AMP |
| WAN 2 | 209.165.200.4/30 | RTR-TOR ↔ RTR-ALD |
| WAN 3 | 209.165.200.8/30 | RTR-ROQ ↔ RTR-ALD |
| WAN 4 | 209.165.200.12/30 | RTR-ROQ ↔ RTR-AMP |
