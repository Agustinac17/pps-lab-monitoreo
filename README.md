# Laboratorio de monitoreo, respaldo y filtrado — PPS II

**Erika Agustina Cáceres** · Ingeniería en Sistemas de Información · Universidad de la Cuenca del Plata (Sede Posadas) · 2026

Este repositorio acompaña el paper de la Práctica Profesional Supervisada II (Actividad de Evaluación 4). En él se reproduce, **en un laboratorio propio**, el tipo de solución que implementé durante mi práctica en el Centro de Cómputos de un organismo público provincial.

> **Confidencialidad.** El protocolo de práctica prohíbe divulgar información de la organización. Por eso aquí no hay direcciones, credenciales, configuraciones ni capturas de la red real. Las direcciones de los ejemplos son genéricas.

---

## Caso de estudio

**Contexto.** Se trata de un organismo público con infraestructura crítica: si la red falla, el afiliado no se atiende.

**Problema.** No había monitoreo (las fallas se conocían por el reclamo de un usuario), los respaldos no tenían histórico, los sistemas nuevos no tenían un entorno aislado y no había control de navegación por grupos.

**Qué hice, sobre Proxmox VE y equipos de red de varias marcas (Ubiquiti, MikroTik, Cisco, HP Aruba):**

| Frente | Resultado |
| --- | --- |
| Monitoreo | Zabbix instalado desde cero, switches monitoreados por SNMP y alertas automáticas a un grupo de Telegram |
| Respaldo | VM de backup con rsync y systemd, copia diaria, 7 días de histórico y limpieza del origen solo tras un backup exitoso |
| Entorno de un sistema nuevo | Conectividad a la red institucional y acceso seguro: usuarios propios, 2FA y firewall |
| Filtrado DNS | AdGuard Home como contenedor en un MikroTik, con reglas de bloqueo y excepciones por grupo de clientes |
| Seguridad | Estudio y guía de Wazuh para su incorporación futura |

**Límite.** La reestructuración de la topología de red se diagnosticó, pero no se ejecutó durante la práctica.

---

## Contenido del repositorio

| Carpeta | Qué hay |
| --- | --- |
| [`lab/`](lab/) | `docker-compose.yml` para levantar Zabbix (servidor, web y base de datos) y los pasos para la demo |
| [`docs/guia-agregar-switches.md`](docs/guia-agregar-switches.md) | Guía para agregar un switch a Zabbix por SNMP |
| [`docs/alertas-telegram.md`](docs/alertas-telegram.md) | Configuración de alertas de Zabbix a un grupo de Telegram |
| [`docs/diagnostico-snmp.md`](docs/diagnostico-snmp.md) | Cómo se diagnosticó el SNMP: timeout → authorizationError → datos |
| [`mikrotik/adguard-container.md`](mikrotik/adguard-container.md) | AdGuard Home como contenedor en RouterOS |
| [`adguard/`](adguard/) | Listas de reglas: bloqueo por categorías y excepciones para clientes autorizados |

---

## Demo: de la falla a la alerta

```
Equipo de prueba ──SNMP──► Zabbix ──trigger──► Media type Telegram ──► Grupo de alertas
```

1. Levantar el laboratorio (ver [`lab/README.md`](lab/README.md)).
2. Agregar como host un equipo de prueba con SNMP (un switch propio o una VM con `snmpd`).
3. Configurar las alertas a Telegram (ver [`docs/alertas-telegram.md`](docs/alertas-telegram.md)).
4. **Prueba:** apagar la interfaz o el equipo de prueba, esperar a que se dispare el trigger y verificar que el mensaje llegue al grupo y quede en el historial de Zabbix.

**Plan B:** si el laboratorio no levanta en la defensa, se muestra el video grabado de la prueba o las capturas.

---

## Relación con mi Proyecto Final de Grado

Mi PFG, **Limbii**, es una plataforma de gestión y seguridad de redes domésticas sobre Raspberry Pi. Las reglas de bloqueo y excepciones por grupo de clientes que configuré en AdGuard son, a escala institucional, el mismo mecanismo de **perfiles y políticas** que Limbii propone para un hogar.
