# Guía para agregar switches a Zabbix por SNMP

El objetivo es ver el estado del equipo, sus interfaces, el tráfico y el estado de los puertos, según el modelo y la plantilla.

## 1. Datos necesarios

| Dato | Ejemplo | Para qué |
| --- | --- | --- |
| Nombre del host | `SW-PISO1-Aruba` | Identificar el equipo en Zabbix |
| IP de gestión | `10.0.10.14` | Desde ahí Zabbix consulta los datos |
| Versión SNMP | v2c | v3 pide parámetros de autenticación adicionales |
| Community | `lab_ro` | La "clave de lectura" que usa Zabbix (no es la contraseña de admin del switch) |
| Grupo de hosts | `Switches` | Organizar los equipos |
| Plantilla | Según el fabricante | Define qué métricas se recolectan |

## 2. En el switch

1. Habilitar SNMP.
2. Crear una **vista** con los objetos que podrá leer la community.
3. Crear la **community** con acceso de **solo lectura**, asociada a esa vista y restringida a la **IP del servidor Zabbix**.
4. Aplicar y guardar la configuración.

## 3. En Zabbix

1. *Data collection → Hosts → Create host*.
2. Nombre, grupo `Switches`, interfaz **SNMP** con la IP y el puerto 161, versión v2c y la community.
3. Elegir la plantilla del fabricante (por ejemplo, *HP Enterprise Switch by SNMP* para Aruba, *Mikrotik by SNMP* o *Cisco IOS by SNMP*).
4. Guardar y esperar a que la disponibilidad SNMP aparezca en verde.

## 4. Verificación

```bash
snmpwalk -v2c -c lab_ro 10.0.10.14 system
```

Si responde, Zabbix también debería poder leer el equipo. Si no, ver [`diagnostico-snmp.md`](diagnostico-snmp.md).
