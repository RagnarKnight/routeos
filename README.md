# MikroTik RB2011UiAS-RM

## Descripción

Router principal de la red LAN. Gestiona la conexión a Internet, DHCP, DNS, NAT y firewall.

## Topología

- **WAN:** `ether1`, recibe IP por DHCP del router superior.
- **LAN:** `BR-LAN`, dirección `10.1.1.1/24`.
- **Puertos LAN:** `ether2`, `ether3`, `ether4` y `ether5`.
- **DHCP:** rango `10.1.1.50` a `10.1.1.200`.
- **DNS de los clientes:** `10.1.1.1`.
- **Servidor SSH:** `10.1.1.122`, puerto interno `22`.

## Funciones

- Conexión a Internet mediante DHCP.
- Asignación automática de direcciones IP.
- Resolución DNS para los clientes LAN.
- NAT para la salida a Internet.
- Firewall para proteger el router y controlar el tráfico.
- Redirección del puerto TCP `23` hacia `10.1.1.122:22`.

## Acceso SSH externo

Las conexiones entrantes al puerto TCP `23` se redirigen al servidor SSH interno.

## Seguridad

- Utilizar una contraseña administrativa robusta.
- Restringir la administración del router a la LAN.
- Mantener RouterOS actualizado.
- Deshabilitar los servicios que no se utilicen.
- Guardar una copia de seguridad después de cada cambio importante.

## Backup

Desde la terminal de RouterOS:

```routeros
/system backup save name=rb2011-config
/export file=rb2011-config
```

Descargar ambos archivos desde WinBox y guardarlos en un lugar seguro.

## Mejoras futuras

- Implementar QoS para priorizar o limitar el tráfico.
- Revisar las reglas de firewall según las necesidades de la red.
- Monitorizar el consumo de ancho de banda y el estado del equipo.


## Interfaces físicas y conexiones

### WAN — TELEMATICA

La conexión a Internet se recibe desde TELEMATICA por fibra óptica. La instalación prevista utiliza una ONT o terminal de fibra que entrega Ethernet al puerto `ether1` del MikroTik.

- **Interfaz:** `ether1`
- **Función:** WAN
- **Dirección IP:** recibida por DHCP del equipo superior
- **Ruta predeterminada:** recibida mediante DHCP

La conexión directa de fibra al puerto SFP no está prevista por el momento.

### LAN — Producción

| Interfaz | Uso |
|---|---|
| `ether2` | Switch Ala Norte |
| `ether3` | Switch Ala Sur |
| `ether4` | Puerto LAN disponible |
| `ether5` | Puerto LAN disponible |
| `ether6`–`ether10` | Reservados; sin uso inicial |
| `sfp1` | Reservado; sin uso inicial |

Las interfaces `ether2`–`ether5` pertenecen al bridge `BR-LAN`, con dirección `10.1.1.1/24`.

Los puertos reservados quedan fuera del bridge LAN mientras no se necesiten. Su estado administrativo debe verificarse en RouterOS.

### Criterio de uso

Los puertos reservados se documentan para facilitar el mantenimiento y las ampliaciones. Antes de habilitar uno, verificar su configuración, incorporarlo al bridge o asignarle la función que corresponda y revisar las reglas de seguridad.

