# Levantar el laboratorio

Requisitos: una máquina o VM (por ejemplo, en Proxmox) con Docker y Docker Compose.

```bash
cd lab
cp .env.example .env        # editar DB_PASSWORD
docker compose up -d
```

Abrir `http://<IP-del-lab>:8080` e ingresar con `Admin` / `zabbix`. **Cambiar la contraseña de inmediato** desde *User settings → Change password*.

## Equipo de prueba con SNMP

Opción A, un switch propio: habilitar SNMP v2c con una community de **solo lectura** restringida a la IP del servidor Zabbix (ver [`../docs/guia-agregar-switches.md`](../docs/guia-agregar-switches.md)).

Opción B, una VM Linux:

```bash
sudo apt install snmpd
# en /etc/snmp/snmpd.conf:
#   agentAddress udp:161
#   rocommunity lab_ro <IP-del-servidor-Zabbix>
sudo systemctl restart snmpd
# prueba desde el servidor Zabbix:
snmpwalk -v2c -c lab_ro <IP-de-la-VM> system
```

Después, en Zabbix: *Data collection → Hosts → Create host*, con una interfaz SNMP (puerto 161, v2c, community `lab_ro`) y la plantilla *Linux by SNMP* (o la del fabricante del switch).

## Lección aprendida en la práctica

Si la VM de Zabbix está en una red de gestión sin salida a internet, se le puede agregar una **segunda interfaz en una red NAT interna de Proxmox** (bridge con `ip_forward` y `MASQUERADE` en el host) solo para actualizar paquetes. El monitoreo SNMP local no necesita internet.
