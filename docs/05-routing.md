# 5. Routing

## 5.1 Objetivo

El objetivo de esta parte del laboratorio es implementar el enrutamiento entre las diferentes sedes de la red empresarial mediante **OSPF**, permitiendo que las redes LAN de Tortosa, Amposta, Roquetes y L'Aldea sean accesibles entre sí.

La infraestructura utiliza varios enlaces WAN entre routers para proporcionar conectividad y caminos alternativos.

La configuración de routing se basa en:

- OSPF como protocolo de enrutamiento dinámico.
- Área única, **Area 0**.
- Router IDs definidos para cada router.
- Enlaces WAN con direccionamiento `/30`.
- Interfaces LAN anunciadas mediante OSPF.
- Interfaces LAN configuradas como pasivas.
- Uso de rutas alternativas mediante la topología redundante.

---

## 5.2 Topología de routing

La infraestructura está formada por cuatro routers:

- `RTR-TOR`
- `RTR-AMP`
- `RTR-ALD`
- `RTR-ROQ`

Los routers están conectados mediante cuatro enlaces WAN:

```text
                         209.165.200.0/30
                    RTR-TOR ───────── RTR-AMP
                     .1                  .2
                      │                    │
                      │                    │
          209.165.200.4/30        209.165.200.12/30
                      │                    │
                     .5                    .14
                      │                    │
                    RTR-ALD ───────── RTR-ROQ
                     .6                  .13
                         209.165.200.8/30
```

La topología permite disponer de más de un camino entre determinadas sedes.

Por ejemplo, `RTR-ROQ` dispone de conectividad hacia Tortosa a través de `RTR-AMP` y de `RTR-ALD`.

---

## 5.3 Direccionamiento de los enlaces WAN

Los enlaces entre routers utilizan redes `/30`, proporcionando dos direcciones utilizables por enlace.

| Enlace | Red | Router A | IP | Router B | IP |
|---|---|---|---|---|---|
| Tortosa - Amposta | `209.165.200.0/30` | RTR-TOR | `209.165.200.1` | RTR-AMP | `209.165.200.2` |
| Tortosa - L'Aldea | `209.165.200.4/30` | RTR-TOR | `209.165.200.5` | RTR-ALD | `209.165.200.6` |
| L'Aldea - Roquetes | `209.165.200.8/30` | RTR-ALD | `209.165.200.10` | RTR-ROQ | `209.165.200.9` |
| Roquetes - Amposta | `209.165.200.12/30` | RTR-ROQ | `209.165.200.13` | RTR-AMP | `209.165.200.14` |

---

## 5.4 Redes LAN anunciadas

Cada sede dispone de sus propias redes LAN.

### Tortosa

Tortosa utiliza varias VLAN, cada una con su propia subred:

| VLAN | Red | Gateway |
|---|---|---|
| VLAN 10 - ADMIN | `192.168.1.0/26` | `192.168.1.1` |
| VLAN 20 - USERS | `192.168.1.64/26` | `192.168.1.65` |
| VLAN 30 - SERVERS | `192.168.1.128/26` | `192.168.1.129` |
| VLAN 40 - GUESTS | `192.168.1.192/26` | `192.168.1.193` |

Estas cuatro redes aparecen como rutas OSPF en los routers remotos.

### Amposta

```text
Red:     192.168.10.0/24
Gateway: 192.168.10.1
```

### Roquetes

```text
Red:     172.16.1.0/24
Gateway: 172.16.1.1
```

### L'Aldea

```text
Red:     172.16.0.0/24
Gateway: 172.16.0.1
```

---

## 5.5 Configuración de OSPF

Se utiliza el proceso:

```text
OSPF 10
```

Todos los routers pertenecen a una única área:

```text
Area 0
```

Los Router ID utilizados son:

| Router | Router ID |
|---|---|
| RTR-TOR | `1.1.1.1` |
| RTR-ALD | `2.2.2.2` |
| RTR-ROQ | `3.3.3.3` |
| RTR-AMP | `4.4.4.4` |

La configuración de OSPF se ha verificado mediante:

```bash
show ip protocols
```

Los cuatro routers muestran el proceso OSPF 10, una única área normal y los correspondientes Router ID.

---

## 5.6 Redes incluidas en OSPF

### RTR-TOR

En `RTR-TOR` se anuncian las interfaces correspondientes a las redes LAN de Tortosa y los enlaces WAN:

```text
192.168.1.1/32
192.168.1.65/32
192.168.1.129/32
192.168.1.193/32
209.165.200.1/32
209.165.200.5/32
```

Las interfaces correspondientes a las VLAN de Tortosa están configuradas como pasivas.

### RTR-ALD

En `RTR-ALD` se anuncian:

```text
172.16.0.1/32
209.165.200.6/32
209.165.200.10/32
```

La interfaz LAN `GigabitEthernet0/0/2` está configurada como pasiva.

### RTR-ROQ

En `RTR-ROQ` se anuncian:

```text
172.16.1.1/32
209.165.200.13/32
209.165.200.9/32
```

La interfaz LAN `GigabitEthernet0/0/2` está configurada como pasiva.

### RTR-AMP

En `RTR-AMP` se anuncian:

```text
192.168.10.1/32
209.165.200.2/32
209.165.200.14/32
```

La interfaz LAN `GigabitEthernet0/0/2` está configurada como pasiva.

---

## 5.7 Vecinos OSPF

El estado de los vecinos se comprueba mediante:

```bash
show ip ospf neighbor
```

### RTR-TOR

`RTR-TOR` mantiene dos adyacencias OSPF:

| Router ID vecino | Dirección | Estado |
|---|---|---|
| `4.4.4.4` | `209.165.200.2` | FULL/DR |
| `2.2.2.2` | `209.165.200.6` | FULL/DR |

### RTR-ALD

`RTR-ALD` mantiene dos adyacencias:

| Router ID vecino | Dirección | Estado |
|---|---|---|
| `1.1.1.1` | `209.165.200.5` | FULL/BDR |
| `3.3.3.3` | `209.165.200.9` | FULL/DR |

### RTR-ROQ

`RTR-ROQ` mantiene dos adyacencias:

| Router ID vecino | Dirección | Estado |
|---|---|---|
| `2.2.2.2` | `209.165.200.10` | FULL/BDR |
| `4.4.4.4` | `209.165.200.14` | FULL/DR |

### RTR-AMP

`RTR-AMP` mantiene dos adyacencias:

| Router ID vecino | Dirección | Estado |
|---|---|---|
| `1.1.1.1` | `209.165.200.1` | FULL/BDR |
| `3.3.3.3` | `209.165.200.13` | FULL/BDR |

Todas las adyacencias verificadas se encuentran en estado `FULL`.

---

## 5.8 Parámetros OSPF de los enlaces WAN

Los enlaces WAN utilizados para OSPF aparecen operativos y asociados al área 0.

Por ejemplo, en `RTR-TOR`:

```text
GigabitEthernet0/0/1
IP: 209.165.200.1/30
Area: 0
Process ID: 10
Router ID: 1.1.1.1
Network Type: BROADCAST
Cost: 1
```

El vecino `4.4.4.4` aparece como Designated Router y `RTR-TOR` como Backup Designated Router.

En el segundo enlace WAN de `RTR-TOR`:

```text
GigabitEthernet0/0/0
IP: 209.165.200.5/30
Area: 0
Process ID: 10
Router ID: 1.1.1.1
Network Type: BROADCAST
Cost: 1
```

El vecino `2.2.2.2` aparece como Designated Router.

Los enlaces restantes presentan igualmente coste OSPF `1` en las evidencias obtenidas.

---

## 5.9 Rutas aprendidas mediante OSPF

Las tablas de routing muestran las rutas aprendidas dinámicamente mediante OSPF con el código:

```text
O
```

La distancia administrativa utilizada para OSPF es:

```text
110
```

### RTR-TOR

`RTR-TOR` aprende mediante OSPF:

```text
172.16.0.0/24
172.16.1.0/24
192.168.10.0/24
209.165.200.8/30
209.165.200.12/30
```

Para `172.16.1.0/24` existen dos siguientes saltos:

```text
209.165.200.2
209.165.200.6
```

Esto demuestra la existencia de dos caminos disponibles hacia la red de Roquetes.

### RTR-ALD

`RTR-ALD` aprende mediante OSPF las redes de:

- Roquetes.
- Tortosa.
- Amposta.
- Otros enlaces WAN.

La red `192.168.10.0/24` dispone de dos siguientes saltos:

```text
209.165.200.9
209.165.200.5
```

### RTR-ROQ

`RTR-ROQ` aprende mediante OSPF las cuatro redes de Tortosa.

Cada una dispone de dos siguientes saltos:

```text
209.165.200.14
209.165.200.10
```

Por ejemplo:

```text
O 192.168.1.0/26 [110/3]
  via 209.165.200.14
  via 209.165.200.10
```

El mismo comportamiento aparece para:

```text
192.168.1.64/26
192.168.1.128/26
192.168.1.192/26
```

### RTR-AMP

`RTR-AMP` aprende mediante OSPF:

```text
172.16.0.0/24
172.16.1.0/24
192.168.1.0/26
192.168.1.64/26
192.168.1.128/26
192.168.1.192/26
209.165.200.4/30
209.165.200.8/30
```

La red `172.16.0.0/24` dispone de dos siguientes saltos:

```text
209.165.200.13
209.165.200.1
```

---

## 5.10 Redundancia de routing

Una de las características principales de la topología es la existencia de caminos alternativos entre las sedes.

Un ejemplo claro aparece en `RTR-ROQ`.

Para llegar a las redes de Tortosa, el router dispone de dos siguientes saltos OSPF:

```text
                         ┌── RTR-ALD ── RTR-TOR
                         │
RTR-ROQ ─────────────────┤
                         │
                         └── RTR-AMP ── RTR-TOR
```

Los siguientes saltos utilizados son:

```text
209.165.200.10
209.165.200.14
```

Las rutas hacia las redes de Tortosa aparecen con una métrica OSPF de:

```text
[110/3]
```

Esto permite disponer de múltiples caminos de coste equivalente hacia los destinos.

También se observa un comportamiento equivalente desde `RTR-ALD` hacia la red de Amposta, donde existen dos siguientes saltos OSPF.

---

## 5.11 Interfaces pasivas

Las interfaces LAN se han configurado como **Passive Interface**.

El objetivo es anunciar las redes LAN mediante OSPF sin intentar establecer adyacencias OSPF con los dispositivos finales.

En `RTR-TOR` aparecen como pasivas:

```text
GigabitEthernet0/0/2
GigabitEthernet0/0/2.10
GigabitEthernet0/0/2.20
GigabitEthernet0/0/2.30
GigabitEthernet0/0/2.40
```

En `RTR-ALD`:

```text
GigabitEthernet0/0/2
```

En `RTR-ROQ`:

```text
GigabitEthernet0/0/2
```

En `RTR-AMP`:

```text
GigabitEthernet0/0/2
```

De esta forma, OSPF mantiene las adyacencias únicamente a través de los enlaces WAN.

---

## 5.12 Verificación del routing

Los principales comandos utilizados para verificar el funcionamiento del routing son:

### Tabla de routing

```bash
show ip route
```

Permite comprobar las redes directamente conectadas y las rutas aprendidas mediante OSPF.

### Vecinos OSPF

```bash
show ip ospf neighbor
```

Permite comprobar las adyacencias entre routers.

El estado esperado de una adyacencia operativa es:

```text
FULL
```

### Configuración del protocolo

```bash
show ip protocols
```

Permite comprobar:

- Proceso OSPF.
- Router ID.
- Áreas.
- Redes anunciadas.
- Interfaces pasivas.
- Fuentes de información de routing.

### Estado de una interfaz OSPF

```bash
show ip ospf interface <interfaz>
```

Permite comprobar:

- Dirección IP.
- Área.
- Proceso OSPF.
- Coste.
- Tipo de red.
- Estado DR/BDR.
- Vecinos.
- Temporizadores.

---

## 5.13 Resultado final

La configuración de routing queda operativa con los siguientes resultados:

- Los cuatro routers ejecutan **OSPF proceso 10**.
- Todos los routers trabajan en **Area 0**.
- Cada router dispone de un Router ID diferente.
- Las adyacencias OSPF verificadas se encuentran en estado `FULL`.
- Las redes LAN remotas aparecen como rutas `O` en las tablas de routing.
- Las interfaces LAN están configuradas como pasivas.
- La topología dispone de caminos alternativos entre diferentes sedes.
- Se observan rutas de coste equivalente hacia determinados destinos.
- Las rutas OSPF utilizan una distancia administrativa de `110`.

Por tanto, el routing dinámico permite la comunicación entre las diferentes redes LAN del laboratorio y proporciona redundancia mediante múltiples caminos OSPF.
