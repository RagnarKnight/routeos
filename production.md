# MikroTik RB2011UiAS-RM — Configuración RouterOS

## 1. Datos de la configuración

| Parámetro | Valor |
|---|---|
| Equipo | MikroTik RB2011UiAS-RM |
| WAN | `ether1` |
| Dirección WAN | DHCP del router superior |
| Bridge LAN | `BR-LAN` |
| Puertos LAN | `ether2`–`ether5` |
| IP del router en LAN | `10.1.1.1/24` |
| Rango DHCP | `10.1.1.50-10.1.1.200` |
| Lista WAN | `WAN-LIST` |
| Lista LAN | `LAN` |
| Servidor SSH | `10.1.1.10` (sinc12)|
| Puerto externo | TCP `23` (ssh)|
| Puerto interno | TCP `22` (sinc12)|


> **Antes de empezar:** confirmar el estado del equipo.

---

## 2. Identificar el equipo y revisar la configuración actual

```routeros
/system identity print
/system resource print
/system package print
/system routerboard print
/interface print detail
/interface bridge print detail
/interface bridge port print detail
/interface list print
/interface list member print detail
/ip address print detail
/ip route print detail
/ip dhcp-client print detail
/ip dhcp-server print detail
/ip pool print detail
/ip firewall nat print detail
/ip firewall filter print detail
```


Verificar que no haya configuraciones
---

## 3. Identificar y renombrar la WAN

Si la interfaz física `ether1` todavía no tiene ese nombre:

```routeros
/interface ethernet set [find where default-name=ether1] name=WAN
```

Verificar:

```routeros
/interface print detail
```

La interfaz debería aparecer como `WAN`.

## 4. Configurar WAN por DHCP

Si no existe un cliente DHCP en `WAN`:

```routeros
/ip dhcp-client add interface=WAN add-default-route=yes default-route-distance=1 use-peer-dns=no disabled=no
```

- Solicite una dirección IP por DHCP.
- Instala la ruta predeterminada recibida del servidor DHCP.
- cuanto menor el número, más preferida la ruta para salir a internet
- No sustituye la configuración DNS que se va a establecer.


Verificar:

```routeros
/ip dhcp-client print detail
/ip address print detail
/ip route print detail
```

**Resultado esperado:**

- El cliente DHCP debe mostrar `status=bound`.
- La interfaz `WAN` debe tener una dirección asignada por el router superior.
- Debe existir una ruta predeterminada hacia el gateway recibido por DHCP.

Si ya hay un cliente DHCP sobre `WAN`, setealo en lugar de crear otro.

---

## 5. Crear el bridge LAN

Si `BR-LAN` no existe:

```routeros
/interface bridge add name=BR-LAN protocol-mode=rstp
```

Agregar los puertos LAN únicamente si todavía no están en otro bridge:

```routeros
/interface bridge port add bridge=BR-LAN interface=ether2
/interface bridge port add bridge=BR-LAN interface=ether3
/interface bridge port add bridge=BR-LAN interface=ether4
/interface bridge port add bridge=BR-LAN interface=ether5
```

Asignar la dirección IP al bridge, si todavía no existe:

```routeros
/ip address add address=10.1.1.1/24 interface=BR-LAN comment="GATEWAY-LAN"
```

Verificar:

```routeros
/interface bridge print detail
/interface bridge port print detail
/ip address print detail
```

Los puertos `ether2`–`ether5` deben pertenecer a `BR-LAN`. La dirección `10.1.1.1/24` debe estar asignada al bridge, no a cada puerto individual.

---

## 6. Configurar el servidor DHCP

Crear el pool si no existe:

```routeros
/ip pool add name=POOL-LAN ranges=10.1.1.50-10.1.1.200
```

Crear la red DHCP si no existe:

```routeros
/ip dhcp-server network add address=10.1.1.0/24 gateway=10.1.1.1 dns-server=10.1.1.1
```

Crear el servidor DHCP si no existe:

```routeros
/ip dhcp-server add name=DHCP-LAN interface=BR-LAN address-pool=POOL-LAN lease-time=1d disabled=no
```

Verificar:

```routeros
/ip pool print detail
/ip dhcp-server print detail
/ip dhcp-server network print detail
/ip dhcp-server lease print detail
```

El servidor DHCP debe estar habilitado sobre `BR-LAN` y entregar direcciones dentro del rango establecido.

---

## 7. Configurar DNS

```routeros
/ip dns set servers=1.1.1.1,8.8.8.8 allow-remote-requests=yes
```

Verificar:

```routeros
/ip dns print
```

El router podrá resolver consultas de los clientes de la red LAN. El firewall de entrada debe impedir que equipos externos usen el router como resolver.

---

## 8. Crear las listas de interfaces

Crear las listas si todavía no existen:

```routeros
/interface list add name=WAN-LIST
/interface list add name=LAN
```

Agregar la WAN:

```routeros
/interface list member add list=WAN-LIST interface=WAN
```

Agregar el bridge LAN:

```routeros
/interface list member add list=LAN interface=BR-LAN
```

Verificar:

```routeros
/interface list print
/interface list member print detail
```

Cada interfaz debe estar en la lista correspondiente.

---

## 9. Configurar NAT de salida

Permitir que los clientes LAN salgan a través de la WAN usando la dirección que recibe por DHCP:

```routeros
/ip firewall nat add chain=srcnat out-interface-list=WAN-LIST action=masquerade comment="NAT-LAN-a-WAN"
```

Verificar:

```routeros
/ip firewall nat print detail
```

Debe existir una sola regla equivalente de `masquerade` para la salida LAN hacia WAN.

---

## 10. Redirigir TCP 23 al servidor SSH

Crear la regla DST-NAT:

```routeros
/ip firewall nat add chain=dstnat in-interface-list=WAN-LIST protocol=tcp dst-port=23 action=dst-nat to-addresses=10.1.1.10 to-ports=22 comment="SSH-externo-a-servidor-LAN"
```

Verificar:

```routeros
/ip firewall nat print detail
```

Esta regla cambia el destino del tráfico entrante TCP `23` hacia `10.1.1.10:22`.

**Importante:** el servidor `SINC12` debe tener ena ip estatica.

---

## 11. Configurar el firewall de entrada

Estas reglas protegen el propio router.

```routeros
/ip firewall filter add chain=input action=accept connection-state=established,related,untracked comment="INPUT conexiones establecidas"
/ip firewall filter add chain=input action=drop connection-state=invalid comment="INPUT descartar invalidas"
/ip firewall filter add chain=input action=accept protocol=icmp comment="INPUT permitir ICMP"
/ip firewall filter add chain=input action=accept in-interface-list=LAN comment="INPUT permitir administracion LAN"
/ip firewall filter add chain=input action=drop in-interface-list=WAN-LIST comment="INPUT bloquear acceso desde WAN"
```

Verificar:

```routeros
/ip firewall filter print detail
```

Orden para `input`:

1. Aceptar conexiones establecidas y relacionadas.
2. Descartar conexiones inválidas.
3. Permitir ICMP.
4. Permitir acceso desde la LAN.
5. Bloquear el resto del tráfico entrante desde WAN.

---

## 12. Configurar el firewall de tránsito

Estas reglas controlan el tráfico que atraviesa el router hacia otros equipos.

```routeros
/ip firewall filter add chain=forward action=accept connection-state=established,related,untracked comment="FORWARD conexiones establecidas"
/ip firewall filter add chain=forward action=drop connection-state=invalid comment="FORWARD descartar invalidas"
/ip firewall filter add chain=forward action=accept in-interface-list=LAN out-interface-list=WAN-LIST comment="FORWARD LAN a WAN"
/ip firewall filter add chain=forward action=accept in-interface-list=WAN-LIST protocol=tcp connection-nat-state=dstnat dst-address=10.1.1.10 dst-port=22 comment="FORWARD SSH redirigido"
/ip firewall filter add chain=forward action=drop comment="FORWARD bloquear resto"
```

Verificar:

```routeros
/ip firewall filter print detail
```

Orden para `forward`:

1. Aceptar conexiones establecidas y relacionadas.
2. Descartar conexiones inválidas.
3. Permitir conexiones de LAN hacia WAN.
4. Permitir el tráfico redirigido hacia el servidor SSH.
5. Bloquear el resto del tráfico.

La regla del SSH redirigido debe estar antes del descarte final. Si ya existe una regla final de descarte, agregá la regla SSH antes de ella, no después.

---

## 13. Verificación general

Ejecutar los siguientes comandos de inspección:

```routeros
/system resource print
/interface print
/interface bridge print detail
/interface bridge port print detail
/interface list member print detail
/ip address print detail
/ip route print detail
/ip dhcp-client print detail
/ip dhcp-server print detail
/ip dhcp-server network print detail
/ip dhcp-server lease print detail
/ip dns print
/ip firewall nat print detail
/ip firewall filter print detail
```

Probar conectividad desde el router a internet:

```routeros
/ping 1.1.1.1 count=4
```

Si responde, probar resolución DNS:

```routeros
:put [:resolve example.com]
```

Si falla alguna prueba revisar primero la WAN y la ruta predeterminada.

---

## 14. Guardar un backup

Una vez revisada y comprobada la configuración:

```routeros
/system backup save name=rb2011-config-inicial
/export file=rb2011-config-inicial
```

Verificar los archivos:

```routeros
/file print
```

El backup binario y el archivo exportado se guardan en el almacenamiento del MikroTik.

---

## 15. Resumen de comandos de inspeccion 

```routeros
/interface print detail
/interface bridge print detail
/interface bridge port print detail
/interface list print
/interface list member print detail
/ip address print detail
/ip route print detail
/ip dhcp-client print detail
/ip dhcp-server print detail
/ip dhcp-server network print detail
/ip dns print
/ip firewall nat print detail
/ip firewall filter print detail
```

Estos comandos no modifican la configuración, solo muestran el estado actual.
