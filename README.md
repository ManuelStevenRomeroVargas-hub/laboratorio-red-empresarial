# Laboratorio de Red Empresarial — 4 sedes con OSPF

Red empresarial simulada en Cisco Packet Tracer: cuatro sedes
interconectadas, segmentación por VLANs, routing dinámico con OSPF,
seguridad de capa 2 y acceso remoto seguro por SSH.

## Índice
- [Escenario](#escenario)
- [Topología](#topología)
- [Direccionamiento](#direccionamiento)
- [Tecnologías y decisiones de diseño](#tecnologías-y-decisiones-de-diseño)
- [Seguridad de capa 2](#seguridad-de-capa-2)
- [Pruebas realizadas](#pruebas-realizadas)
- [Problemas encontrados y solución](#problemas-encontrados-y-solución)
- [Cómo abrir el laboratorio](#cómo-abrir-el-laboratorio)
- [Qué mejoraría](#qué-mejoraría)

## Escenario

Empresa con cuatro sedes (Tortosa, Amposta, Roquetes y L'Aldea) que
necesita:

- Comunicación entre todas las sedes.
- Segmentación del tráfico por departamentos dentro de cada sede.
- Que cada sede tenga su propio servidor DHCP para sus VLANs.
- Acceso remoto seguro a los routers (SSH, no Telnet).
- Protección frente a bucles, tormentas de broadcast y ataques
  básicos de capa 2 (MAC flooding, ARP spoofing, DHCP rogue).

> Nota: el escenario es inventado por mí para dar contexto realista
> al laboratorio. Los requisitos son los que yo mismo definí antes
> de configurar nada.

## Topología

Diagrama: [`topologia.png`](./topologia.png)

- **Tortosa** es la sede principal y actúa como núcleo de la red.
- Cada sede tiene un router que la conecta al resto vía OSPF.
- Dentro de cada sede hay switches de acceso con VLANs por
  departamento y un switch de distribución/core con EtherChannel
  hacia los de acceso.

(Ver `topologia.png` para el detalle de enlaces y equipos.)

## Direccionamiento

Cada sede usa un bloque /16 propio, subdividido en /24 por VLAN:

| Sede      | Bloque          | VLAN | Red             | Gateway        |
|-----------|-----------------|------|-----------------|----------------|
| Tortosa   | 10.10.0.0/16    | 10   | 10.10.10.0/24   | 10.10.10.1     |
|           |                 | 20   | 10.10.20.0/24   | 10.10.20.1     |
|           |                 | 30   | 10.10.30.0/24   | 10.10.30.1     |
| Amposta   | 10.20.0.0/16    | 10   | 10.20.10.0/24   | 10.20.10.1     |
|           |                 | 20   | 10.20.20.0/24   | 10.20.20.1     |
| Roquetes  | 10.30.0.0/16    | 10   | 10.30.10.0/24   | 10.30.10.1     |
|           |                 | 20   | 10.30.20.0/24   | 10.30.20.1     |
| L'Aldea   | 10.40.0.0/16    | 10   | 10.40.10.0/24   | 10.40.10.1     |
|           |                 | 20   | 10.40.20.0/24   | 10.40.20.1     |

- **Enlaces entre routers**: /30 dedicados.
- **VLAN nativa**: 99 (no usada por usuarios).
- **VLAN de gestión**: 999.

> Ajusta esta tabla a tus rangos reales. Es la sección que más te van
> a mirar: si los números no cuadran con el .pkt, se nota.

## Tecnologías y decisiones de diseño

- **VLANs + 802.1Q**: una VLAN por departamento y sede, para aislar
  dominios de broadcast y aplicar políticas por segmento.
- **Routing inter-VLAN (router-on-a-stick / SVIs)**: cada sede enruta
  entre sus VLANs en su router.
- **OSPF**: elegido frente a RIP por ser estándar abierto, de
  convergencia rápida y escalable; frente a EIGRP por no depender de
  Cisco. Una sola área (área 0) porque el tamaño del lab no justifica
  jerarquía.
- **EtherChannel / LACP**: agrupa enlaces entre switches para sumar
  ancho de banda y tolerar el fallo de un cable, evitando que STP
  bloquee uno de los enlaces.
- **DHCP por sede**: cada router es servidor DHCP de sus VLANs; en
  las VLANs sin servidor local se usa `ip helper-address` hacia el
  router correspondiente.
- **SSH con ACL**: acceso de gestión solo por SSH (Telnet deshabilitado)
  y ACL que limita el origen de las conexiones a la VLAN de gestión.

## Seguridad de capa 2

- **Port Security**: limita el número de MACs por puerto de acceso y
  define acción ante violación. Objetivo: evitar MAC flooding y
  conexión de switches no autorizados en puertos de usuario.
- **PortFast**: solo en puertos de acceso finales, para que el host
  no espere los estados de STP. **Nunca en trunks ni en puertos de
  switch**.
- **BPDU Guard**: en los mismos puertos con PortFast, para que si
  alguien conecta un switch ahí, el puerto se bloquee. Complementa a
  PortFast.
- **DHCP Snooping**: marca como *trusted* solo el uplink hacia el
  router/servidor DHCP y como *untrusted* los puertos de usuario, para
  bloquear servidores DHCP rogue.
- **Dynamic ARP Inspection (DAI)**: usa la tabla de DHCP Snooping
  para validar ARP y frenar ARP spoofing.

## Pruebas realizadas

- `ping` entre VLANs de la misma sede y entre sedes (todo el rango).
- `show ip ospf neighbor` en los 4 routers: adyacencias FULL.
- `show etherchannel summary`: canal en uso (SU).
- `show port-security interface`: MACs aprendidas y violaciones.
- `show ip dhcp snooping binding`: bindings coherentes.
- Cliente DHCP obtiene IP en cada VLAN.
- SSH funciona desde la VLAN de gestión; Telnet rechazado.

> Añade capturas en `capturas/` y enlázalas aquí. Sin capturas,
> esta sección es una promesa; con capturas, es una prueba.

## Problemas encontrados y solución

(Esta sección la escribes tú, sin IA. 5 problemas reales, con
"síntoma → causa → solución". Ejemplos de estructura:)

1. **Síntoma**: OSPF no formaba adyacencia entre dos sedes.
   **Causa**: las interfaces estaban en áreas distintas / máscara
   distinta / MTU distinta.
   **Solución**: unifiqué área y revisé `show ip ospf interface`.

2. **Síntoma**: los clientes de una VLAN no recibían IP.
   **Causa**: faltaba `ip helper-address` en la SVI / el DHCP
   snooping bloqueaba el OFFER porque el uplink no era trusted.
   **Solución**: ...

3. **Síntoma**: Port Security ponía el puerto en err-disabled.
   **Causa**: más MACs de las permitidas / MAC del switch vecino.
   **Solución**: ...

4. **Síntoma**: un enlace del EtherChannel no se agregaba.
   **Causa**: configuración distinta en los dos extremos (modo,
   VLAN permitidas, velocidad).
   **Solución**: ...

5. **Síntoma**: DAI bloqueaba ARP legítimo.
   **Causa**: puerto no trusted / binding no aprendido.
   **Solución**: ...

## Cómo abrir el laboratorio

1. Instalar **Cisco Packet Tracer** (versión usada: X.X).
2. Abrir `red-empresarial.pkt`.
3. Esperar a que STP y OSPF converjan (~1 min).
4. Comprobar con `show ip ospf neighbor` en cada router.

## Qué mejoraría

- Añadir **HSRP/VRRP** para redundancia de gateway.
- **IPv6** (SLAAC + OSPFv3).
- **NTP y syslog** centralizados.
- **Automatización** de backups de configuración con Python + Netmiko
  (fuera de Packet Tracer).
- Migrar a **GNS3/EVE-NG** para trabajar con imágenes reales.

## Autor

[Tu nombre] — Grado Medio SMR, Grado Superior ASIR y DAM, CCNA.
[LinkedIn] · [GitHub]
