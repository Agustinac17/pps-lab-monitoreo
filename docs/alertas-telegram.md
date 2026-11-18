# Alertas de Zabbix a un grupo de Telegram

Flujo: **Zabbix detecta un problema → Media type Telegram → bot → grupo de alertas.**

| Elemento | Qué es | Dónde se usa |
| --- | --- | --- |
| Bot | El "mensajero" que envía las alertas | Telegram |
| Token del bot | La llave para usar el bot | Media type de Zabbix |
| Chat ID | El destino de los mensajes | Media del usuario de Zabbix |

## Pasos

1. **Crear el bot:** en Telegram, abrir `@BotFather`, enviar `/newbot` y elegir un nombre y un usuario terminado en `bot`. Guardar el token **sin publicarlo nunca**.
2. **Crear el grupo de alertas** y agregar el bot. Usar un grupo exclusivo para alertas, separado de las conversaciones cotidianas.
3. **Obtener el Chat ID del grupo:** enviar un mensaje en el grupo y consultar `https://api.telegram.org/bot<TOKEN>/getUpdates`. El Chat ID de un grupo suele ser negativo.
4. **Probar desde la terminal:**
   ```bash
   curl -s -X POST "https://api.telegram.org/bot<TOKEN>/sendMessage" \
        -d chat_id="<CHAT_ID>" -d text="Prueba desde Zabbix"
   ```
5. **Media type en Zabbix:** *Alerts → Media types → Telegram*, completar `api_token` con el token y probar con *Test*.
6. **Media del usuario:** *Users → Admin → Media → Add*, tipo Telegram, *Send to* = Chat ID del grupo, severidad desde *Warning*.
7. **Acción:** *Alerts → Actions → Trigger actions*, con la condición *Trigger severity ≥ Warning* y la operación de enviar al usuario por Telegram.
8. **Prueba de punta a punta:** provocar una falla en un host de prueba y verificar que llegue el mensaje.
