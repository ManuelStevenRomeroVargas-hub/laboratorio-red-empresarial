# 4. Switching

## 4.1 Introducción

La infraestructura de switching de la sede de Tortosa está diseñada
mediante una arquitectura basada en un switch de core y dos switches
de acceso.

La red se encuentra segmentada mediante VLAN para separar los
diferentes tipos de tráfico y facilitar la administración y seguridad
de la infraestructura.

Además, se utilizan enlaces trunk 802.1Q y EtherChannel mediante LACP
para proporcionar conectividad entre los switches y aumentar la
disponibilidad de los enlaces.

---

## 4.2 VLAN implementadas

Se han definido las siguientes VLAN en la sede de Tortosa:

| VLAN | Nombre | Uso |
|---|---|---|
| 10 | ADMIN | Equipos de administración |
| 20 | USERS | Equipos de usuarios |
| 30 | SERVERS | Servidores |
| 40 | GUESTS | Dispositivos invitados |
| 50 | VOICE | Telefonía IP |
| 99 | NATIVE | VLAN nativa de los trunks |

La segmentación mediante VLAN permite separar lógicamente los
diferentes tipos de tráfico dentro de la misma infraestructura física.

---

## 4.3 Asignación de puertos

### SW-TOR-1

| Puertos | VLAN | Uso |
|---|---|---|
| F0/1-F0/10 | 10 | Administración |
| F0/11-F0/20 | 30 | Servidores |
| F0/2 | 10 + Voice 50 | Teléfono IP / equipo asociado |
| F0/21-F0/22 | Trunk / EtherChannel | Enlace con SW-TOR-CORE |

### SW-TOR-2

| Puertos | VLAN | Uso |
|---|---|---|
| F0/1-F0/10 | 20 | Usuarios |
| F0/11-F0/18 | 40 | Invitados |
| F0/21-F0/22 | 40 | Invitados |
| F0/19-F0/20 | Trunk / EtherChannel | Enlace con SW-TOR-CORE |
| F0/23-F0/24 | Trunk / EtherChannel | Enlace con SW-TOR-ACC01 |

> Nota: la asignación exacta de puertos debe mantenerse sincronizada
> con la configuración final del archivo `.pkt`.

---

## 4.4 Enlaces Trunk

Los enlaces entre los switches utilizan trunking 802.1Q para
transportar tráfico perteneciente a diferentes VLAN.

Los trunks permiten que las VLAN 10, 20, 30, 40 y 50 puedan
transportarse entre los diferentes switches cuando sea necesario.

La VLAN 99 se utiliza como VLAN nativa de los enlaces trunk.

El uso de una VLAN nativa dedicada permite evitar utilizar una VLAN
de usuarios como VLAN nativa.

---

## 4.5 EtherChannel

Para aumentar la capacidad y disponibilidad de determinados enlaces
entre switches se ha implementado EtherChannel.

La agregación de enlaces se realiza utilizando **LACP
(Link Aggregation Control Protocol)**.

Se han configurado los siguientes Port-Channels:

| Port-Channel | Enlaces físicos | Conexión |
|---|---|---|
| Po1 | F0/21-F0/22 | SW-TOR-CORE ↔ SW-TOR-ACC01 |
| Po2 | F0/19-F0/20 | SW-TOR-CORE ↔ SW-TOR-ACC02 |
| Po3 | F0/23-F0/24 | SW-TOR-ACC01 ↔ SW-TOR-ACC02 |

Cada grupo de enlaces físicos funciona lógicamente como un único
enlace.

Esto proporciona:

- Mayor capacidad de transmisión.
- Redundancia frente al fallo de un enlace físico.
- Simplificación de la topología lógica.
- Mejor utilización de los enlaces.
- Integración con el funcionamiento de Spanning Tree.

---

## 4.6 LACP

LACP se utiliza para negociar y mantener los grupos EtherChannel.

La configuración permite que los enlaces físicos pertenecientes al
mismo grupo funcionen como un único Port-Channel.

La utilización de LACP evita depender de una configuración
propietaria y permite una negociación estandarizada del
EtherChannel.

---

## 4.7 PortFast

PortFast se configura en los puertos destinados a dispositivos
finales.

Su objetivo es permitir que un puerto conectado a un equipo final
pase rápidamente al estado de forwarding, evitando la espera
innecesaria de los estados tradicionales de Spanning Tree.

Se aplica principalmente a puertos de acceso donde se conectan
ordenadores y otros dispositivos finales.

PortFast no debe utilizarse de forma indiscriminada en enlaces entre
switches.

---

## 4.8 BPDU Guard

BPDU Guard se utiliza conjuntamente con PortFast para proteger los
puertos destinados a dispositivos finales.

Si un dispositivo conectado a un puerto protegido comienza a enviar
BPDUs, el switch puede considerar que existe un posible problema de
topología y poner el puerto en estado de error.

Esto ayuda a evitar que un dispositivo no autorizado pueda introducir
cambios en la topología de Spanning Tree.

---

## 4.9 Port Security

Se ha implementado Port Security en los puertos de acceso destinados
a dispositivos finales.

El objetivo es limitar qué dispositivos pueden utilizar determinados
puertos físicos del switch.

Esta medida permite reducir riesgos asociados a la conexión de
dispositivos no autorizados.

La configuración de Port Security se aplica en los puertos de acceso
correspondientes y puede utilizar mecanismos como la limitación del
número de direcciones MAC permitidas.

---

## 4.10 DHCP Snooping

En la infraestructura de switching de Amposta se ha implementado
DHCP Snooping como mecanismo de seguridad de Capa 2.

DHCP Snooping permite distinguir entre puertos confiables y no
confiables para el tráfico DHCP.

El objetivo principal es evitar que un dispositivo no autorizado
pueda actuar como servidor DHCP dentro de la red.

Los puertos conectados hacia el servicio DHCP o hacia la infraestructura
que transporta tráfico DHCP legítimo se consideran de confianza,
mientras que los puertos destinados a usuarios permanecen como
no confiables.

---

## 4.11 Dynamic ARP Inspection

Dynamic ARP Inspection (DAI) se ha configurado en Amposta como
mecanismo adicional de seguridad de Capa 2.

DAI permite validar determinados mensajes ARP utilizando la
información obtenida por DHCP Snooping.

De esta forma se ayuda a prevenir ataques basados en la suplantación
ARP dentro de la red local.

La implementación de DHCP Snooping y DAI proporciona una capa
adicional de protección frente a ataques de tipo ARP spoofing.

---

## 4.12 Verificación de la configuración

Para verificar el funcionamiento de la infraestructura de switching
se pueden utilizar los siguientes comandos:

### VLAN

```text
show vlan brief
