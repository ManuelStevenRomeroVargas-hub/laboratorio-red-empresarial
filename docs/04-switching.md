# 4. Switching

## 4.1 Introducción

La infraestructura de switching de la sede de Tortosa está diseñada mediante una arquitectura basada en un switch de core y dos switches de acceso.

La red se encuentra segmentada mediante VLAN para separar los diferentes tipos de tráfico y facilitar la administración y seguridad de la infraestructura.

Además, se utilizan enlaces trunk 802.1Q y EtherChannel mediante LACP para proporcionar conectividad entre los switches y aumentar la disponibilidad de los enlaces.

---

## 4.2 VLAN implementadas

Se han definido las siguientes VLAN en la sede de Tortosa:

| VLAN | Nombre  | Uso                       |
| ---- | ------- | ------------------------- |
| 10   | ADMIN   | Equipos de administración |
| 20   | USERS   | Equipos de usuarios       |
| 30   | SERVERS | Servidores                |
| 40   | GUESTS  | Dispositivos invitados    |
| 50   | VOICE   | Telefonía IP              |
| 99   | NATIVE  | VLAN nativa de los trunks |

La segmentación mediante VLAN permite separar lógicamente los diferentes tipos de tráfico dentro de la misma infraestructura física.

---

## 4.3 Asignación de puertos

### SW-TOR-CORE

El switch `SW-TOR-CORE` actúa como switch central de la infraestructura de switching de Tortosa.

| Interfaz | Función                  |
| -------- | ------------------------ |
| Po1      | Trunk hacia SW-TOR-ACC01 |
| Po2      | Trunk hacia SW-TOR-ACC02 |
| Gig0/2   | Trunk                    |

### SW-TOR-ACC01

| Puertos     | VLAN             | Uso                           |
| ----------- | ---------------- | ----------------------------- |
| F0/1-F0/10  | 10               | Administración                |
| F0/11-F0/20 | 30               | Servidores                    |
| F0/2        | 10 + Voice 50    | Teléfono IP / equipo asociado |
| F0/21-F0/22 | EtherChannel Po1 | Enlace con SW-TOR-CORE        |
| F0/23-F0/24 | EtherChannel Po3 | Enlace con SW-TOR-ACC02       |

### SW-TOR-ACC02

| Puertos     | VLAN             | Uso                     |
| ----------- | ---------------- | ----------------------- |
| F0/1-F0/10  | 20               | Usuarios                |
| F0/11-F0/18 | 40               | Invitados               |
| F0/21-F0/22 | 40               | Invitados               |
| F0/19-F0/20 | EtherChannel Po2 | Enlace con SW-TOR-CORE  |
| F0/23-F0/24 | EtherChannel Po3 | Enlace con SW-TOR-ACC01 |

> Nota: la asignación exacta de puertos se ha comprobado mediante `show vlan brief` en los switches de acceso.

---

## 4.4 Enlaces Trunk

Los enlaces entre los switches utilizan trunking 802.1Q para transportar tráfico perteneciente a diferentes VLAN.

La VLAN 99 se utiliza como VLAN nativa de los enlaces trunk.

En `SW-TOR-CORE`, la comprobación mediante `show interfaces trunk` muestra los siguientes enlaces en funcionamiento como trunk:

* `Po1`
* `Po2`
* `Gig0/2`

Todos utilizan 802.1Q y tienen configurada la VLAN 99 como VLAN nativa.

Las VLAN permitidas en estos trunks son:

* VLAN 10
* VLAN 20
* VLAN 30
* VLAN 40
* VLAN 99

La VLAN 50 no aparece entre las VLAN permitidas en los trunks del `SW-TOR-CORE`.

---

## 4.5 EtherChannel

Para aumentar la capacidad y disponibilidad de determinados enlaces entre switches se ha implementado EtherChannel.

La agregación de enlaces se realiza utilizando LACP (Link Aggregation Control Protocol).

Se han configurado los siguientes Port-Channels:

| Port-Channel | Enlaces físicos | Conexión                    |
| ------------ | --------------- | --------------------------- |
| Po1          | F0/21-F0/22     | SW-TOR-CORE ↔ SW-TOR-ACC01  |
| Po2          | F0/19-F0/20     | SW-TOR-CORE ↔ SW-TOR-ACC02  |
| Po3          | F0/23-F0/24     | SW-TOR-ACC01 ↔ SW-TOR-ACC02 |

Cada grupo de enlaces físicos funciona lógicamente como un único enlace.

Esto proporciona:

* Mayor capacidad de transmisión.
* Redundancia frente al fallo de un enlace físico.
* Simplificación de la topología lógica.
* Mejor utilización de los enlaces.
* Integración con el funcionamiento de Spanning Tree.

---

## 4.6 LACP

LACP se utiliza para negociar y mantener los grupos EtherChannel.

La configuración permite que los enlaces físicos pertenecientes al mismo grupo funcionen como un único Port-Channel.

La utilización de LACP permite utilizar un protocolo estandarizado para la agregación de enlaces.

La configuración se ha verificado mediante `show etherchannel summary` en los tres switches.

En `SW-TOR-CORE` se encuentran operativos:

* `Po1`
* `Po2`

En `SW-TOR-ACC01` se encuentran operativos:

* `Po1`
* `Po3`

En `SW-TOR-ACC02` se encuentran operativos:

* `Po2`
* `Po3`

Los Port-Channels aparecen con estado `SU` y las interfaces físicas asociadas aparecen con estado `P`.

---

## 4.7 PortFast

PortFast se configura en los puertos destinados a dispositivos finales.

Su objetivo es permitir que un puerto conectado a un equipo final pase rápidamente al estado de forwarding, evitando la espera innecesaria de los estados tradicionales de Spanning Tree.

Se aplica principalmente a puertos de acceso donde se conectan ordenadores y otros dispositivos finales.

PortFast no debe utilizarse de forma indiscriminada en enlaces entre switches.

---

## 4.8 BPDU Guard

BPDU Guard se utiliza conjuntamente con PortFast para proteger los puertos destinados a dispositivos finales.

Si un dispositivo conectado a un puerto protegido comienza a enviar BPDUs, el switch puede considerar que existe un posible problema de topología y poner el puerto en estado de error.

Esto ayuda a evitar que un dispositivo no autorizado pueda introducir cambios en la topología de Spanning Tree.

---

## 4.9 Port Security

Se ha implementado Port Security en los puertos de acceso destinados a dispositivos finales.

El objetivo es limitar qué dispositivos pueden utilizar determinados puertos físicos del switch.

Esta medida permite reducir riesgos asociados a la conexión de dispositivos no autorizados.

En `SW-TOR-ACC01`, la configuración se ha comprobado mediante:

```text
show port-security
```

Los puertos configurados muestran un máximo de 3 direcciones MAC y el modo de violación configurado como `Shutdown`.

Como comprobación específica se ha utilizado:

```text
show port-security interface f0/1
```

El resultado muestra:

```text
Port Security              : Enabled
Port Status                : Secure-up
Violation Mode             : Shutdown
Aging Time                 : 0 mins
Aging Type                 : Absolute
SecureStatic Address Aging : Disabled
Maximum MAC Addresses      : 3
Total MAC Addresses        : 1
Configured MAC Addresses   : 0
Sticky MAC Addresses       : 1
Security Violation Count   : 0
```

El puerto `Fa0/1` se encuentra en estado `Secure-up`, con una dirección MAC aprendida mediante Sticky MAC y sin violaciones de seguridad registradas.

---

## 4.10 DHCP Snooping

En la infraestructura de switching de Amposta se ha implementado DHCP Snooping como mecanismo de seguridad de Capa 2.

DHCP Snooping permite distinguir entre puertos confiables y no confiables para el tráfico DHCP.

El objetivo principal es evitar que un dispositivo no autorizado pueda actuar como servidor DHCP dentro de la red.

En `SW-AMP-01` se ha utilizado:

```text
show ip dhcp snooping
```

El resultado confirma que DHCP Snooping está habilitado.

Las VLAN configuradas para DHCP Snooping son:

* VLAN 10
* VLAN 1000

El puerto `GigabitEthernet0/2` aparece como `Trusted`, mientras que los puertos FastEthernet aparecen como `Untrusted`.

También se ha utilizado:

```text
show ip dhcp snooping binding
```

En el momento de la comprobación no existen bindings registrados:

```text
Total number of bindings: 0
```

---

## 4.11 Dynamic ARP Inspection

Dynamic ARP Inspection (DAI) se ha configurado en `SW-AMP-01` como mecanismo adicional de seguridad de Capa 2.

DAI permite validar determinados mensajes ARP utilizando la información obtenida por DHCP Snooping.

En `SW-AMP-01` se ha utilizado:

```text
show ip arp inspection
```

El resultado muestra DAI configurado para las VLAN 10 y 1000.

En la comprobación realizada, ambas VLAN aparecen con:

```text
Configuration: Enabled
Operation: Inactive
```

También se ha comprobado el estado de confianza de las interfaces mediante:

```text
show ip arp inspection interfaces
```

Los puertos FastEthernet aparecen como `Untrusted`, mientras que `GigabitEthernet0/2` aparece como `Trusted`.

---

## 4.12 Verificación de la configuración

Para verificar el funcionamiento de la infraestructura de switching se han realizado diferentes comprobaciones mediante comandos `show` en los switches.

### VLAN

En `SW-TOR-CORE` se ha utilizado:

```text
show vlan brief
```

El resultado confirma la existencia de las VLAN:

```text
10   ADMIN
20   USERS
30   SERVERS
40   GUESTS
99   NATIVE
```

En `SW-TOR-ACC01` se ha comprobado la asignación de puertos mediante el mismo comando.

La salida muestra:

```text
10   ADMIN      Fa0/1 - Fa0/10
30   SERVERS    Fa0/11 - Fa0/20
50   VOICE      Fa0/2
```

En `SW-TOR-ACC02` se ha comprobado:

```text
20   USERS      Fa0/1 - Fa0/10
40   GUESTS     Fa0/11 - Fa0/18, Fa0/21 - Fa0/22
```

### Trunks

En `SW-TOR-CORE` se ha utilizado:

```text
show interfaces trunk
```

La salida muestra:

```text
Po1         802.1q         trunking      99
Po2         802.1q         trunking      99
Gig0/2      802.1q         trunking      99
```

Las VLAN permitidas son:

```text
10,20,30,40,99
```

Los tres enlaces aparecen en estado `trunking`.

### EtherChannel

En `SW-TOR-CORE`:

```text
show etherchannel summary
```

Resultado:

```text
1      Po1(SU)           LACP   Fa0/21(P) Fa0/22(P)
2      Po2(SU)           LACP   Fa0/19(P) Fa0/20(P)
```

En `SW-TOR-ACC01`:

```text
1      Po1(SU)           LACP   Fa0/21(P) Fa0/22(P)
3      Po3(SU)           LACP   Fa0/23(P) Fa0/24(P)
```

En `SW-TOR-ACC02`:

```text
2      Po2(SU)           LACP   Fa0/19(P) Fa0/20(P)
3      Po3(SU)           LACP   Fa0/23(P) Fa0/24(P)
```

Los Port-Channels aparecen operativos y las interfaces físicas aparecen integradas en sus respectivos grupos.

### Spanning Tree

En `SW-TOR-CORE` se ha utilizado:

```text
show spanning-tree
```

Para la VLAN 10 se observa:

```text
Root ID    Priority    32778
           Address     000C.8596.C21E
           This bridge is the root
```

Esto indica que `SW-TOR-CORE` actúa como Root Bridge para la VLAN 10.

Los interfaces `Gig0/2`, `Po1` y `Po2` aparecen en estado `FWD`.

Para la VLAN 20 y la VLAN 40, el Root Bridge es otro switch y `Po2` aparece como puerto Root.

### Port Security

En `SW-TOR-ACC01` se ha utilizado:

```text
show port-security
```

El resultado muestra Port Security configurado en los puertos de acceso.

Como comprobación específica se ha utilizado:

```text
show port-security interface f0/1
```

El puerto `Fa0/1` presenta:

```text
Port Security              : Enabled
Port Status                : Secure-up
Violation Mode             : Shutdown
Maximum MAC Addresses      : 3
Total MAC Addresses        : 1
Sticky MAC Addresses       : 1
Security Violation Count   : 0
```

### DHCP Snooping

En `SW-AMP-01` se ha utilizado:

```text
show ip dhcp snooping
```

El resultado confirma que DHCP Snooping está habilitado para las VLAN 10 y 1000.

El puerto `GigabitEthernet0/2` aparece como `Trusted`, mientras que los puertos FastEthernet aparecen como `Untrusted`.

También se ha utilizado:

```text
show ip dhcp snooping binding
```

En el momento de la comprobación no existen bindings registrados:

```text
Total number of bindings: 0
```

### Dynamic ARP Inspection

En `SW-AMP-01` se ha utilizado:

```text
show ip arp inspection
```

El resultado muestra DAI configurado para las VLAN 10 y 1000, aunque en el momento de la comprobación aparece como `Inactive`.

También se ha utilizado:

```text
show ip arp inspection interfaces
```

Los puertos FastEthernet aparecen como `Untrusted`, mientras que `GigabitEthernet0/2` aparece como `Trusted`.

---

## 4.13 Resumen de las comprobaciones

| Comprobación           | Equipo       | Comando                             | Resultado                  |
| ---------------------- | ------------ | ----------------------------------- | -------------------------- |
| VLAN                   | SW-TOR-CORE  | `show vlan brief`                   | VLAN configuradas          |
| VLAN                   | SW-TOR-ACC01 | `show vlan brief`                   | VLAN y puertos comprobados |
| VLAN                   | SW-TOR-ACC02 | `show vlan brief`                   | VLAN y puertos comprobados |
| Trunks                 | SW-TOR-CORE  | `show interfaces trunk`             | Trunks operativos          |
| EtherChannel           | SW-TOR-CORE  | `show etherchannel summary`         | Po1 y Po2 operativos       |
| EtherChannel           | SW-TOR-ACC01 | `show etherchannel summary`         | Po1 y Po3 operativos       |
| EtherChannel           | SW-TOR-ACC02 | `show etherchannel summary`         | Po2 y Po3 operativos       |
| Spanning Tree          | SW-TOR-CORE  | `show spanning-tree`                | Estado comprobado          |
| Port Security          | SW-TOR-ACC01 | `show port-security`                | Configurado                |
| DHCP Snooping          | SW-AMP-01    | `show ip dhcp snooping`             | Habilitado                 |
| DHCP Snooping bindings | SW-AMP-01    | `show ip dhcp snooping binding`     | 0 bindings                 |
| Dynamic ARP Inspection | SW-AMP-01    | `show ip arp inspection`            | Configurado / Inactive     |
| DAI interfaces         | SW-AMP-01    | `show ip arp inspection interfaces` | Interfaces comprobadas     |

---

## 4.14 Conclusión

Las comprobaciones realizadas permiten verificar la configuración de los principales mecanismos de switching implementados en la infraestructura.

Se ha comprobado la configuración de las VLAN, los enlaces trunk, los Port-Channels mediante LACP, el estado de Spanning Tree y los mecanismos de seguridad de Capa 2.

Las pruebas también permiten identificar el estado actual de DHCP Snooping y Dynamic ARP Inspection, incluyendo la ausencia de bindings DHCP Snooping y el estado `Inactive` de DAI en el momento de la comprobación.
