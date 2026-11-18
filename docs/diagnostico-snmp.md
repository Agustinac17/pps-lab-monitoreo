# Diagnóstico de SNMP: de timeout a datos

Caso real de la práctica (con datos anonimizados): un switch respondía a ping, pero Zabbix no podía leerlo por SNMP.

| Paso | Síntoma en Zabbix | Qué significaba | Acción |
| --- | --- | --- | --- |
| 1 | `Timeout while connecting to <IP>:161` | El equipo no respondía a SNMP: community incorrecta o acceso no permitido | Revisar la configuración SNMP real del switch |
| 2 | Se confundía el usuario admin del switch con la community | El admin sirve para entrar por web o consola, no para SNMP | Usar la community SNMP configurada |
| 3 | `authorizationError (access denied to that object)` | **Avance:** el equipo ya respondía, pero la community no tenía permiso sobre los objetos | Asociar la community a una vista que incluya esos objetos |
| 4 | Disponibilidad SNMP en verde | Zabbix lee interfaces, tráfico y estado | Verificar los ítems y los triggers |

**Lección:** cada cambio de error es información. Pasar de *timeout* a *authorizationError* demostró que el problema ya no era de conectividad, sino de permisos.
