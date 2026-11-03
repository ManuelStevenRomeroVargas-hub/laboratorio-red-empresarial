# Laboratorio de Red Empresarial --- 4 sedes con OSPF

Red empresarial multisede simulada en Cisco Packet Tracer: cuatro sedes
interconectadas (Tortosa, Amposta, Roquetes y L'Aldea), segmentación por
VLAN, routing dinámico con OSPF, servicios de red (DHCP, SSH) y
mecanismos de seguridad de capa 2.

El proyecto está documentado en `docs/` y las evidencias de verificación
(salidas de comandos `show`) están referenciadas en cada sección.

## Índice

-   [Escenario](#escenario)
-   [Topología](#topología)
-   [Direccionamiento](#direccionamiento)
-   [Routing: OSPF](#routing-ospf)
-   [Switching: VLANs, trunks y
    EtherChannel](#switching-vlans-trunks-y-etherchannel)
-   [Servicios de red](#servicios-de-red)
-   [Seguridad de capa 2](#seguridad-de-capa-2)
-   [Verificación y estado real del
    laboratorio](#verificación-y-estado-real-del-laboratorio)
-   [Qué mejoraría / próximos pasos](#qué-mejoraría--próximos-pasos)
-   [Cómo abrir el laboratorio](#cómo-abrir-el-laboratorio)
-   [Estructura del repositorio](#estructura-del-repositorio)
-   [Autor](#autor)

## Escenario

Empresa con cuatro sedes (Tortosa, Amposta, Roquetes y L'Aldea) que
necesita:

-   Comunicación entre todas las sedes.
-   Segmentación del tráfico por función dentro de Tortosa (sede
    principal).
-   DHCP en las redes donde se decide usar direccionamiento dinámico.
-   Acceso remoto seguro por SSH, restringido por ACL.
-   Protección frente a riesgos básicos de capa 2 (MAC flooding, DHCP
    rogue, ARP spoofing y bucles).

> El escenario es inventado por mí para dar un contexto realista al
> laboratorio. Los requisitos los definí antes de configurar nada.

## Topología

Diagrama: [`diagramas/topologia-red.png`](./diagramas/topologia-red.png)

-   **Tortosa** es la sede principal y concentra la infraestructura de
    switching segmentada por VLAN y el routing inter-VLAN en `RTR-TOR`.
-   **Amposta, Roquetes y L'Aldea** tienen una red LAN única cada una.
-   Las sedes están conectadas mediante **enlaces punto a punto /30** y
    participan en un dominio **OSPF de área única (Area 0)**.
-   La topología WAN forma un **anillo de cuatro routers**, lo que
    proporciona caminos alternativos entre sedes.

``` text
                         209.165.200.0/30
                    RTR-TOR ───────── RTR-AMP
                     .1                  .2
                      │                    │
          209.165.200.4/30        209.165.200.12/30
                      │                    │
                     .5                    .14
                      │                    │
                    RTR-ALD ───────── RTR-ROQ
                     .6                  .13
                         209.165.200.8/30
```

## Direccionamiento

### Tortosa

Segmentación en subredes `/26` sobre `192.168.1.0/24`.

    VLAN Nombre    Red                  Gateway
  ------ --------- -------------------- -----------------
      10 ADMIN     `192.168.1.0/26`     `192.168.1.1`
      20 USERS     `192.168.1.64/26`    `192.168.1.65`
      30 SERVERS   `192.168.1.128/26`   `192.168.1.129`
      40 GUESTS    `192.168.1.192/26`   `192.168.1.193`
      50 VOICE     (voz)                ---
      99 NATIVE    (trunks)             ---

### Resto de sedes

  Sede       Red                 Gateway          Direccionamiento
  ---------- ------------------- ---------------- ------------------
  Amposta    `192.168.10.0/24`   `192.168.10.1`   DHCP (`RTR-AMP`)
  Roquetes   `172.16.1.0/24`     `172.16.1.1`     Estático
  L'Aldea    `172.16.0.0/24`     `172.16.0.1`     Estático

### Enlaces WAN (/30)

  -------------------------------------------------------------------------------------------------------
  Enlace              Red                   Router A    IP                 Router B    IP
  ------------------- --------------------- ----------- ------------------ ----------- ------------------
  Tortosa--Amposta    `209.165.200.0/30`    `RTR-TOR`   `209.165.200.1`    `RTR-AMP`   `209.165.200.2`

  Tortosa--L'Aldea    `209.165.200.4/30`    `RTR-TOR`   `209.165.200.5`    `RTR-ALD`   `209.165.200.6`

  L'Aldea--Roquetes   `209.165.200.8/30`    `RTR-ALD`   `209.165.200.10`   `RTR-ROQ`   `209.165.200.9`

  Roquetes--Amposta   `209.165.200.12/30`   `RTR-ROQ`   `209.165.200.13`   `RTR-AMP`   `209.165.200.14`
  -------------------------------------------------------------------------------------------------------

La tabla completa por interfaz y por equipo está en
`docs/03-direccionamiento.md`.

## Routing: OSPF

-   **Protocolo:** OSPFv2, proceso 10, área única (Area 0).
-   **Router IDs:**
    -   `RTR-TOR`: `1.1.1.1`
    -   `RTR-ALD`: `2.2.2.2`
    -   `RTR-ROQ`: `3.3.3.3`
    -   `RTR-AMP`: `4.4.4.4`
-   Las interfaces LAN están configuradas como pasivas: se anuncian las
    redes sin intentar formar adyacencias con equipos finales.
-   Adyacencias verificadas en estado **FULL** en los cuatro routers
    (`show ip ospf neighbor`).
-   **Redundancia:** la topología en anillo permite rutas de coste
    equivalente. Por ejemplo, `RTR-ROQ` aprende las cuatro redes de
    Tortosa con dos siguientes saltos (`209.165.200.10` y
    `209.165.200.14`) y métrica `[110/3]`.

**Decisión de diseño:** OSPF se eligió por ser un estándar abierto y
ofrecer convergencia rápida, frente a EIGRP (propietario de Cisco) o RIP
(limitado). Se utiliza una sola área porque el tamaño del laboratorio no
justifica una jerarquía.

Detalle completo y tablas de rutas: `docs/05-routing.md`.

## Switching: VLANs, trunks y EtherChannel

Infraestructura de Tortosa: un core (`SW-TOR-CORE`) y dos switches de
acceso (`SW-TOR-ACC01`, `SW-TOR-ACC02`).

-   **VLANs:** 10 ADMIN, 20 USERS, 30 SERVERS, 40 GUESTS, 50 VOICE y 99
    NATIVE.
-   **Trunks:** 802.1Q con VLAN 99 como nativa.
-   **EtherChannel:** LACP (Po1, Po2 y Po3) entre core y acceso, y entre
    los dos switches de acceso. Estado `SU` verificado con
    `show etherchannel summary`; interfaces físicas en estado `P`.
-   **Spanning Tree:** `SW-TOR-CORE` es Root Bridge para VLAN 10; para
    VLAN 20 y 40 el root es otro switch y Po2 aparece como puerto root.

> **Nota técnica:** en `SW-TOR-CORE`, la VLAN 50 (VOICE) no aparece
> entre las VLAN permitidas en los trunks (10, 20, 30, 40 y 99). Está
> documentado así de forma deliberada: es una incoherencia real del
> laboratorio, no un error de la documentación.

Detalle de asignación de puertos y verificaciones:
`docs/04-switching.md`.

## Servicios de red

### DHCP

-   **Tortosa:** `RTR-TOR` actúa como servidor DHCP para las VLANs de
    usuario; el host `DHCP-SERVER` (`192.168.1.130/26`) forma parte de
    la VLAN 30 con direccionamiento estático.
-   **Amposta:** `RTR-AMP` proporciona DHCP a la LAN `192.168.10.0/24`.
-   **Roquetes y L'Aldea:** direccionamiento estático en los equipos.

### SSH

-   Administración remota cifrada en routers y switches, con Telnet
    deshabilitado.
-   **ACL en VTY:** en Tortosa se restringe el acceso a la VLAN 10
    (`192.168.1.0/26`); en el resto de sedes, a la LAN local.

Detalle: `docs/06-servicios.md`.

## Seguridad de capa 2

  -----------------------------------------------------------------------
  Mecanismo               Ámbito                  Estado real documentado
  ----------------------- ----------------------- -----------------------
  SSH + ACL en VTY        Routers/switches        Configurado

  Port Security           `SW-TOR-ACC01`          Habilitado, máximo 3
                                                  MACs, violación
                                                  `Shutdown`

  Sticky MAC              `SW-TOR-ACC01`          Evidenciado en `Fa0/1`

  PortFast + BPDU Guard   Puertos de acceso       En diseño; falta
                                                  evidencia por interfaz

  DHCP Snooping           `SW-AMP-01`             Habilitado en VLAN 10 y
                                                  1000; 0 bindings en la
                                                  captura

  DAI                     `SW-AMP-01`             Configurado en VLAN 10
                                                  y 1000; estado
                                                  `Inactive`
  -----------------------------------------------------------------------

**Honestidad técnica:** DHCP Snooping está habilitado pero sin bindings
observados; DAI está configurado pero inactivo. El README no presenta
estas funciones como plenamente operativas. Es una decisión consciente:
prefiero documentar el estado real y dejar claro qué falta por validar.

Detalle y buenas prácticas: `docs/07-seguridad.md`.

## Verificación y estado real del laboratorio

Comprobaciones realizadas y evidenciadas en `docs/`:

  -------------------------------------------------------------------------
  Comando                               Resultado documentado
  ------------------------------------- -----------------------------------
  `show ip ospf neighbor`               Adyacencias FULL en los 4 routers

  `show ip route`                       Rutas O con distancia
                                        administrativa 110 y caminos
                                        alternativos documentados

  `show etherchannel summary`           Po1, Po2 y Po3 en estado SU

  `show interfaces trunk`               Trunks 802.1Q con VLAN nativa 99

  `show spanning-tree`                  Root Bridge identificado por VLAN

  `show port-security interface f0/1`   `Secure-up`, 1 sticky MAC y 0
                                        violaciones

  `show ip dhcp snooping`               Habilitado, Gi0/2 trusted y
                                        FastEthernet untrusted

  `show ip dhcp snooping binding`       0 bindings

  `show ip arp inspection`              Configurado, operación `Inactive`
  -------------------------------------------------------------------------

## Cómo abrir el laboratorio

1.  Instalar Cisco Packet Tracer (versión usada: `9.0.0.0810`).
2.  Abrir `red-empresarial.pkt`.
3.  Esperar a que STP y OSPF converjan (aproximadamente 1 minuto).
4.  Comprobar con `show ip ospf neighbor` en cada router.
5.  Revisar las verificaciones listadas en `docs/`.

## Estructura del repositorio

``` text
.
├── README.md
├── red-empresarial.pkt
├── diagramas/
│   └── topologia-red.png
├── docs/
│   ├── 01-descripcion.md
│   ├── 02-diseno.md
│   ├── 03-direccionamiento.md
│   ├── 04-switching.md
│   ├── 05-routing.md
│   ├── 06-servicios.md
│   └── 07-seguridad.md
└── capturas/
    └── (salidas de verificación)
```

## Autor

Manuel Steven Romero Vargas --- Grado Medio SMR, Grado Superior ASIR y DAM, CCNA.

www.linkedin.com/in/manuel-steven-romero-vargas-6a72b820b - github.com/ManuelStevenRomeroVargas-hub/laboratorio-red-empresarial
