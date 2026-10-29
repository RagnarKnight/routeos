## Desactivar dhcp-client

```bash
/ip dhcp-client disable [find where interface=WAN]
```

- Asignar ip 

```bash
/ip address add address=192.168.168.10/24 interface=WAN
```

- gateway

```bash 
/ip route add dst-address=0.0.0.0/0 gateway=192.168.168.1
```


## Setear dhcp-client

```bash
/ip dhcp-client print detail
/ip dhcp-client set [find where interface="WAN"] add-default-route=yes default-route-distance=1 use-peer-dns=yes use-peer-ntp=yes disabled=no
```

- Para borrarlo y crearlo de nuevo en esa interfaz:

```bash 
/ip dhcp-client remove [find where interface="WAN"]
/ip dhcp-client add interface=WAN add-default-route=yes default-route-distance=1 use-peer-dns=yes use-peer-ntp=yes disabled=no
```

## Si existe un bridg renómbralo y activa RSTP:

```bash 
/interface bridge set [find where name="name-bridge"] name=BR-LAN protocol-mode=rstp

```
- Quitar un port del bridge:

```bash 
/interface bridge port remove [find where interface="ether2"]
```
- Agregar un port al bridge 

```bash 
/interface bridge port add bridge=BR-LAN interface=ether2
```

## Cambiar una IP existente:

```bash 
/ip address set [find where address="192.168.88.1/24"] address=10.1.1.1/24 interface=BR-LAN
```

- Asignar ip si no hay:

```bash
/ip address add address=10.1.1.1/24 interface=BR-LAN

```

## Configurar el servidor DHCP existente en BR-LAN:

```bash 
/ip dhcp-server set [find where name="DHCP-LAN"] interface=BR-LAN address-pool=POOL-LAN lease-time=1d disabled=no
```

- modifica la entrada de red si ya existe

```bash 
/ip dhcp-server network set [find where address="192.168.10.0/24"] gateway=10.1.1.1 dns-server=10.1.1.1
```

## Quitar una interfaz de una lista

```bash 
/interface list member remove [find where list="WAN-LIST" and interface="WAN"]
```

- Agregar interfaz

```bash 
/interface list member add list=WAN-LIST interface=WAN
```


## Para eliminar una regla identificada por un comentario único:

```bash 
/ip firewall filter remove [find where comment="COMENTARIO-DE-LA-REGLA"]

```
- modificar 

```bash
/ip firewall filter set [find where comment="COMENTARIO-DE-LA-REGLA"] action=drop

```
