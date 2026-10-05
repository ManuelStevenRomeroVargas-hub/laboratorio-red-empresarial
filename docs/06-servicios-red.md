# 6 - Servicios de red

## 6.1 Objetivo

En esta sección se documentan los principales servicios de red implementados en el laboratorio empresarial y su relación con la infraestructura de switching y routing.

El objetivo es mostrar no solo la conectividad entre sedes, sino también los servicios necesarios para que los equipos puedan obtener configuración de red y para que la administración de los dispositivos se realice de forma segura.

Los servicios tratados en este laboratorio son:

- DHCP.
- Acceso remoto mediante SSH.
- Control de acceso a las líneas VTY mediante ACL.
- Servicios de seguridad asociados al acceso de los dispositivos.
- DHCP Snooping y Dynamic ARP Inspection como mecanismos de protección de acceso.

---

## 6.2 DHCP

El direccionamiento de los equipos no se gestiona de la misma forma en todas las sedes. El laboratorio combina direccionamiento dinámico mediante DHCP con direccionamiento estático.

### 6.2.1 Tortosa

En Tortosa se utiliza el router `RTR-TOR` como infraestructura de routing entre VLAN y como servidor DHCP para las redes de usuarios.

Las redes LAN de Tortosa están segmentadas mediante VLAN:

| VLAN | Nombre | Red | Gateway |
|---:|---|---|---|
| 10 | ADMIN | `192.168.1.0/26` | `192.168.1.1` |
| 20 | USERS | `192.168.1.64/26` | `192.168.1.65` |
| 30 | SERVERS | `192.168.1.128/26` | `192.168.1.129` |
| 40 | GUESTS | `192.168.1.192/26` | `192.168.1.193` |

La red de servidores utiliza direccionamiento estático. Por ejemplo, el servidor `DHCP-SERVER` tiene la dirección:

```text
IP:       192.168.1.130/26
Gateway:  192.168.1.129
```

Por tanto, el servicio DHCP no debe confundirse con la red de servidores: el servidor identificado como `DHCP-SERVER` forma parte de la infraestructura del laboratorio, mientras que la función DHCP también está asociada al router de Tortosa según el diseño realizado.

### 6.2.2 Amposta

En Amposta el router `RTR-AMP` proporciona el gateway de la LAN:

```text
Red:      192.168.10.0/24
Gateway:  192.168.10.1
```

Los equipos de usuario de esta sede utilizan direccionamiento dinámico.

La infraestructura de switching de Amposta incorpora además DHCP Snooping para controlar las respuestas DHCP procedentes de los puertos de acceso.

### 6.2.3 Roquetes y L'Aldea

Las sedes de Roquetes y L'Aldea utilizan direccionamiento estático en los equipos representados en el laboratorio.

#### Roquetes

```text
Red:      172.16.1.0/24
Gateway:  172.16.1.1
```

Ejemplos:

```text
PC-ROQ-USR01 → 172.16.1.10/24
PC-ROQ-USR02 → 172.16.1.11/24
```

#### L'Aldea

```text
Red:      172.16.0.0/24
Gateway:  172.16.0.1
```

Ejemplos:

```text
PC-ALD-USR01 → 172.16.0.10/24
PC-ALD-USR02 → 172.16.0.11/24
```

---

## 6.3 Acceso remoto mediante SSH

Los dispositivos de red del laboratorio disponen de acceso administrativo remoto mediante SSH.

El uso de SSH permite administrar routers y switches sin recurrir a Telnet, evitando el envío de las credenciales de administración en texto plano.

El acceso remoto forma parte de la configuración básica de administración de:

- `RTR-TOR`
- `RTR-AMP`
- `RTR-ROQ`
- `RTR-ALD`
- `SW-TOR-CORE`
- `SW-TOR-ACC01`
- `SW-TOR-ACC02`
- `SW-AMP-01`

La administración se combina con restricciones mediante ACL sobre las líneas VTY.

---

## 6.4 Restricción del acceso SSH mediante ACL

El acceso administrativo no queda abierto de forma indiscriminada a todas las redes.

En Tortosa, las líneas VTY están restringidas para permitir administración desde la VLAN de administración:

```text
VLAN 10 - ADMIN
192.168.1.0/26
```

En las demás sedes, el acceso administrativo se limita a la LAN local correspondiente.

Este diseño separa el tráfico de administración del tráfico normal de usuarios y reduce la superficie de exposición de los dispositivos de red.

La idea general es:

```text
                RED DE ADMINISTRACIÓN
                         │
                         ▼
                  ┌─────────────┐
                  │   ACL VTY    │
                  └──────┬──────┘
                         │
                         ▼
                  Acceso SSH permitido
                         │
                         ▼
                 Router / Switch
```

---

## 6.5 DHCP Snooping

En `SW-AMP-01` se ha configurado DHCP Snooping.

La configuración está aplicada a:

```text
VLAN 10
VLAN 1000
```

La interfaz conectada hacia la infraestructura superior está marcada como confiable:

```text
GigabitEthernet0/2 → Trusted
```

Los puertos de acceso de los usuarios permanecen como no confiables.

La configuración permite que el switch diferencie entre:

- tráfico DHCP procedente de puertos de confianza;
- tráfico DHCP procedente de puertos de acceso.

Además, está habilitada la inserción de Option 82 y no se permite Option 82 en puertos no confiables.

La verificación disponible muestra:

```text
Switch DHCP snooping is enabled

DHCP snooping is configured on following VLANs:
10,1000

Option 82 on untrusted port is not allowed

GigabitEthernet0/2 → Trusted
Fa0/1-Fa0/4        → Untrusted
```

La tabla de bindings mostrada en la evidencia contiene actualmente:

```text
Total number of bindings: 0
```

Por tanto, la documentación debe distinguir entre **DHCP Snooping configurado** y **bindings DHCP observados en la captura**.

---

## 6.6 Dynamic ARP Inspection

También se ha configurado Dynamic ARP Inspection (DAI) en `SW-AMP-01`.

Las VLAN configuradas para DAI son:

```text
VLAN 10
VLAN 1000
```

La evidencia disponible indica:

```text
Source Mac Validation      : Disabled
Destination Mac Validation : Disabled
IP Address Validation      : Disabled
```

Y muestra:

```text
VLAN 10    Configuration: Enabled    Operation: Inactive
VLAN 1000  Configuration: Enabled    Operation: Inactive
```

Por este motivo, el resultado debe documentarse con precisión:

> DAI está configurado para las VLAN indicadas, pero la evidencia disponible muestra la operación como **Inactive**.

No se debe presentar esta función como una protección DAI plenamente operativa en el estado capturado del laboratorio.

Los puertos aparecen definidos con un estado de confianza coherente con la separación entre infraestructura y acceso:

```text
GigabitEthernet0/2 → Trusted
Puertos de acceso  → Untrusted
```

---

## 6.7 Relación entre servicios y segmentación

Los servicios se integran con la segmentación de la red.

```text
                         RED EMPRESARIAL
                               │
             ┌─────────────────┼─────────────────┐
             │                 │                 │
          Tortosa           Amposta          Roquetes/L'Aldea
             │                 │                 │
       VLAN 10/20/30/40     VLAN 10            LAN local
             │                 │                 │
       ┌─────┴─────┐         │                 │
       │           │         │                 │
     DHCP        SSH/ACL    DHCP            IP estática
       │           │         │                 │
       └───────────┴─────────┴─────────────────┘
                         │
                       OSPF
                         │
                 Conectividad entre sedes
```

La separación de servicios permite que cada función tenga un ámbito definido:

| Servicio / función | Ubicación | Objetivo |
|---|---|---|
| DHCP | Tortosa / Amposta | Asignación dinámica de parámetros IP |
| SSH | Routers y switches | Administración remota segura |
| ACL VTY | Routers y switches | Restringir origen del acceso administrativo |
| DHCP Snooping | SW-AMP-01 | Controlar tráfico DHCP por confianza de puertos |
| DAI | SW-AMP-01 | Protección ARP configurada, actualmente inactiva |

---

## 6.8 Verificación

Para comprobar los servicios de red se pueden utilizar, entre otros, los siguientes comandos:

### DHCP

En routers:

```text
show ip dhcp pool
show ip dhcp binding
show running-config
```

En el switch de Amposta:

```text
show ip dhcp snooping
show ip dhcp snooping binding
```

### SSH y acceso administrativo

```text
show running-config
show access-lists
```

La comprobación debe centrarse en verificar las líneas VTY, el método de acceso y las ACL asociadas.

### DHCP Snooping y DAI

```text
show ip dhcp snooping
show ip dhcp snooping binding
show ip arp inspection
show ip arp inspection interfaces
```

---

## 6.9 Resultado final

La infraestructura incorpora servicios de red orientados tanto a la operación como a la administración segura:

- DHCP para automatizar el direccionamiento en las redes donde se utiliza.
- Direccionamiento estático para determinados equipos y sedes.
- SSH para administración remota.
- ACL para limitar el origen del acceso administrativo.
- DHCP Snooping en Amposta.
- Dynamic ARP Inspection configurado en Amposta, con operación mostrada como inactiva en la evidencia disponible.

La combinación de estos servicios con la segmentación por VLAN, el routing OSPF y los mecanismos de seguridad de los switches permite que el laboratorio represente una infraestructura empresarial pequeña pero con separación de funciones y controles básicos de seguridad.

---

## 6.10 Evidencias recomendadas

Para la documentación final del proyecto, las evidencias más representativas de esta sección son:

1. `show ip dhcp pool`
2. `show ip dhcp binding`
3. `show ip dhcp snooping`
4. `show ip dhcp snooping binding`
5. `show ip arp inspection`
6. `show ip arp inspection interfaces`
7. `show access-lists`
8. `show running-config` de las líneas VTY

No es necesario incluir configuraciones completas si ya existe documentación específica de cada tecnología. Es preferible mostrar una salida breve que demuestre que el servicio está configurado y funcionando, o indicar explícitamente cuando la evidencia muestra una función configurada pero inactiva.
