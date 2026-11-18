# AdGuard Home como contenedor en MikroTik (RouterOS 7)

Direcciones de ejemplo: LAN `192.168.50.0/24`, router `192.168.50.1`, AdGuard `192.168.50.2`.

## 1. Base del router

1. Partir de la configuración de fábrica eliminada. **Lección aprendida:** los puertos del bridge de fábrica impedían que funcionara el DHCP.
2. Configurar la IP WAN (y la ruta por defecto, si es estática).
3. Crear un **bridge** con los puertos LAN (ether2 a ether5) y asignarle `192.168.50.1/24`.
4. Configurar el **servidor DHCP** sobre el bridge, el **NAT** (masquerade hacia la WAN) y el DNS con *allow remote requests*.

> Antes de borrar algo, asegurarse de no quedar sin acceso al equipo.

## 2. Habilitar contenedores

```
/system/package/update/check-for-updates
/system/package/update/install
/system/routerboard/upgrade
/system/reboot
# instalar el paquete extra "container" de la misma versión (desde mikrotik.com, arrastrándolo a Files) y reiniciar
/system/device-mode/update container=yes
/system/device-mode/print
```

Si la memoria interna no alcanza: conectar un **USB**, formatearlo en ext4 desde *System → Disks* y usarlo para el contenedor.

## 3. Red del contenedor

```
/interface/veth/add name=veth-adguard address=192.168.50.2/24 gateway=192.168.50.1
/interface/bridge/port/add bridge=bridge interface=veth-adguard
```

## 4. Almacenamiento y contenedor

```
/container/config/set registry-url=https://registry-1.docker.io tmpdir=usb1-part1/tmp
/file add type=directory name=usb1-part1/adguard/work
/file add type=directory name=usb1-part1/adguard/conf
/container/mounts/add name=adguard_work src=usb1-part1/adguard/work dst=/opt/adguardhome/work
/container/mounts/add name=adguard_conf src=usb1-part1/adguard/conf dst=/opt/adguardhome/conf
/container/add remote-image=adguard/adguardhome:latest interface=veth-adguard \
    root-dir=usb1-part1/adguard/root mounts=adguard_conf,adguard_work start-on-boot=yes
/container/print
/container/start [find where name~"adguard"]
```

## 5. Entregar AdGuard como DNS

```
/ip/dhcp-server/network/set [find] dns-server=192.168.50.2
```

## 6. Políticas por grupo

En AdGuard, crear un cliente persistente llamado `autorizados` con las IPs o MACs habilitadas y cargar:

- [`../adguard/reglas_bloqueo.txt`](../adguard/reglas_bloqueo.txt): bloqueo por categorías (redes sociales, mensajería y streaming) para todos los clientes;
- [`../adguard/reglas_permisos.txt`](../adguard/reglas_permisos.txt): excepciones `@@...$client=autorizados` que habilitan esos servicios solo para ese grupo.
