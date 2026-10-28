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

