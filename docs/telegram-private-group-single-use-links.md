# Telegram: Enlace único (1 uso) a grupo privado por ticket

## Objetivo
Cuando el sistema **emita un ticket**, debe:
- Generar un **enlace único** para un **grupo privado de Telegram**.
- El enlace debe **“expirar al usarse”**.
- Enviar al usuario un **correo** con su enlace.

> En Telegram, “expirar al entrar” se implementa como **enlace de 1 solo uso**: una vez que alguien entra, el enlace ya **no permite** más ingresos.

---

## Limitaciones importantes (Telegram)
- **No existe** un “expire al hacer click”.
  - Telegram no notifica “click”; solo eventos como “se unió” o “solicitud de unirse”.
  - Por lo tanto, el comportamiento real es: **expira tras el primer ingreso exitoso**.
- Si el usuario abre el link pero **no completa el ingreso**, el link seguirá válido hasta que alguien entre (o lo revoques).

---

## Enfoque recomendado (sin expiración por fecha)

### Enlace de 1 solo uso (sin `expire_date`)
Usar `createChatInviteLink` con:
- `member_limit = 1`
- **NO enviar** `expire_date`
- (opcional) `name` para trazabilidad (ticket/usuario)

Esto hace que el link quede “consumido” automáticamente al primer ingreso.

### Endurecimiento opcional: revocar luego del ingreso
Para asegurar que el link quede inválido incluso ante edge-cases, puedes:
- Detectar el evento de ingreso al grupo (webhook / polling de updates).
- Llamar `revokeChatInviteLink` inmediatamente.

---

## Requisitos previos
1. Crear un bot con `@BotFather` y obtener `BOT_TOKEN`.
2. El grupo debe ser **privado** y preferiblemente **supergrupo**.
3. Agregar el bot al grupo como **admin**, con permiso para:
   - Crear invitaciones (invites).
   - (Opcional) Gestionar miembros/solicitudes si usas join requests.
4. Obtener el `chat_id` del grupo (ej: `-1001234567890`).

Documentación oficial:
- Bot API: `https://core.telegram.org/bots/api`
- `createChatInviteLink`: `https://core.telegram.org/bots/api#createchatinvitelink`
- `revokeChatInviteLink`: `https://core.telegram.org/bots/api#revokechatinvitelink`

---

## Flujo funcional (high level)

### 1) Ticket emitido → crear invite link
Cuando se emite un ticket:
1. El backend llama a Telegram `createChatInviteLink` con `member_limit=1`.
2. Guarda en DB el `invite_link` asociado a `ticketId` y `userId`.
3. Envía el email al usuario con el link.

### 2) (Opcional) Marcar usado / revocar
Si tienes webhook de Telegram:
- Al detectar que alguien entró, marca el invite como **usado** y (opcional) **revócalo**.

---

## Llamadas a la Telegram Bot API
Base URL:
`https://api.telegram.org/bot<BOT_TOKEN>/<METHOD>`

### Crear enlace (1 uso, sin fecha)
`POST /createChatInviteLink`

Payload sugerido:
```json
{
  "chat_id": "-1001234567890",
  "member_limit": 1,
  "name": "ticket-<ticketId>-user-<userId>"
}
```

Respuesta relevante:
- `result.invite_link`: URL lista para enviar por email.

### Revocar enlace (opcional)
`POST /revokeChatInviteLink`

```json
{
  "chat_id": "-1001234567890",
  "invite_link": "https://t.me/+AbCdEf..."
}
```

---

## Persistencia recomendada (DB)
Colección/tabla: `telegram_invites`

Campos mínimos:
- `ticketId`
- `userId`
- `chatId`
- `inviteLink`
- `memberLimit` (1)
- `createdAt`
- `usedAt` (nullable)
- `revokedAt` (nullable)

Índices:
- Unique: `inviteLink`
- Index: `(ticketId, userId)`

---

## Webhook de Telegram (opcional, para “revocar al entrar”)
Si deseas automatizar “revocar al entrar”, configura un webhook para recibir updates del bot.

### Updates de interés
Dependiendo de la configuración y del tipo de chat, puedes recibir:
- `my_chat_member` / `chat_member`: cambios de membresía.
- `message.new_chat_members`: en algunos escenarios, el bot puede ver miembros nuevos (no siempre aplica igual en grupos privados).

> Nota: La disponibilidad exacta de estos updates puede variar por tipo de chat/permisos. Si no recibes eventos de ingreso, igual `member_limit=1` garantiza el “single-use”.

### Seguridad
Al configurar webhook, usa `secret_token` y valida el header:
`X-Telegram-Bot-Api-Secret-Token`

---

## Reenvío / recuperación (casos reales)
Casos comunes:
- El usuario no usó el link y lo perdió → reemitir un nuevo link.
- El link fue consumido por otra persona (reenvío) → reemitir y revocar el anterior.

Regla simple:
- Si el invite está **usado** o **revocado** → generar uno nuevo y enviar nuevamente.

---

## Recomendación final
Para cumplir “expira al entrar” **sin fecha**:
- Implementa **`member_limit=1`** sin `expire_date`.
- (Opcional) Agrega webhook para marcar `usedAt` y **revocar** por limpieza/auditoría.

