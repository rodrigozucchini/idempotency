# Idempotencia: de cero a implementación

## 1. Qué es

Una operación es **idempotente** si ejecutarla **una vez** o **N veces** deja el sistema en **el mismo estado final**.

```
f(x) = f(f(x))
```

Ejemplos cotidianos:

| Operación | ¿Idempotente? | Por qué |
|---|---|---|
| Apretar el botón "apagar luz" | Sí | Apagada queda apagada, aunque lo aprietes 5 veces |
| Apretar el botón del ascensor | Sí | Ya está llamado, más clicks no cambian nada |
| "Sumar $10 al saldo" | **No** | Cada ejecución cambia el resultado |
| "Poner el saldo en $110" | Sí | Da igual cuántas veces se ejecute |
| "Crear un pedido nuevo" | **No** | Cada llamada crea otro pedido |
| "Cancelar el pedido 42" | Sí | Cancelado queda cancelado |

**Idea clave:** idempotente no significa "no hace nada la segunda vez" ni "devuelve siempre la misma respuesta". Significa que **el efecto sobre el estado** no se acumula. (Una respuesta puede variar: un `DELETE` devuelve `204` la primera vez y `404` la segunda, y sigue siendo idempotente porque el estado final es el mismo: el recurso no existe.)

### Conceptos que se confunden

- **Seguro (safe):** no modifica nada (`GET`). Todo método seguro es idempotente, pero no al revés.
- **Idempotente:** puede modificar, pero repetir no cambia el resultado final.
- **Función pura:** misma entrada → misma salida, sin efectos secundarios. Es otro concepto.

## 2. Por qué existe el problema

En sistemas distribuidos **las llamadas fallan de formas ambiguas**:

```
Cliente ──POST /pagos──▶ Servidor ──cobra──▶ ✅
Cliente ◀──── timeout ──── (la respuesta se perdió)
```

El cliente **no sabe** si el cobro se hizo o no. Si reintenta y la operación no es idempotente → **doble cobro**.

Situaciones reales donde pasa lo mismo:

- Timeout de red o el usuario hace doble click en "Pagar".
- Un cliente HTTP / gateway / balanceador **reintenta automáticamente**.
- Un **webhook** (Stripe, MercadoPago, etc.) se envía de nuevo porque no respondiste a tiempo.
- Una cola entrega el mensaje **"at-least-once"**: garantiza que llega, pero puede llegar dos veces.
- Un job falla a la mitad y se reintenta desde el principio.

No podés evitar los reintentos (son necesarios para la confiabilidad). Lo que sí podés es que **reintentar sea seguro**. Por eso se dice: *"exactly-once" en la práctica = at-least-once + idempotencia*.

## 3. Métodos HTTP y su idempotencia

| Método | Seguro | Idempotente | Nota |
|---|---|---|---|
| GET, HEAD, OPTIONS | Sí | Sí | Solo lectura |
| PUT | No | Sí | Reemplaza el recurso completo |
| DELETE | No | Sí | El estado final es "no existe" |
| POST | No | **No** | Es el que necesita ayuda (idempotency key) |
| PATCH | No | Depende | `{status:"paid"}` sí; `{increment: 1}` no |

Esto es lo que dice la especificación; **tu implementación puede romperlo** (un `PUT` que además manda un mail cada vez ya no es idempotente en la práctica).

## 4. Tipos / patrones para lograrla

### Tipo A: Idempotente por diseño (la mejor opción cuando se puede)

Diseñá la operación como **"establecer un estado"** en vez de **"aplicar un cambio"**.

```sql
-- ❌ No idempotente
UPDATE cuentas SET saldo = saldo + 10 WHERE id = 1;

-- ✅ Idempotente (estado absoluto)
UPDATE cuentas SET saldo = 110 WHERE id = 1;
-- (ojo: puede pisar cambios concurrentes, ver Tipo D)
```

Otras formas: `PUT` en vez de `POST`, `SET` en vez de `INCREMENT`, `ensure_exists` en vez de `create`.

### Tipo B: Upsert / constraint único (clave natural)

Si el recurso tiene un **identificador natural o externo** (id del pago en el proveedor, email, número de orden), dejá que la base lo garantice.

```sql
CREATE UNIQUE INDEX ux_pagos_external_id ON pagos(external_id);

INSERT INTO pagos (external_id, monto, estado)
VALUES ($1, $2, 'pendiente')
ON CONFLICT (external_id) DO NOTHING;      -- o DO UPDATE SET ...
```

La segunda ejecución no inserta nada. La unicidad la garantiza **la base de datos**, no tu código (que sufre race conditions).

### Tipo C: Idempotency Key (para POST y operaciones no naturalmente idempotentes)

El **cliente** genera un identificador único (UUID) por *intención de operación* y lo manda en un header. El servidor guarda el resultado asociado a esa clave y, si la ve de nuevo, **devuelve el resultado guardado en vez de volver a ejecutar**.

```http
POST /pagos
Idempotency-Key: 8f14e45f-ceea-4e5c-9d8b-1a2b3c4d5e6f
Content-Type: application/json

{ "monto": 5000, "moneda": "ARS" }
```

Flujo del servidor:

```
Llega request con Idempotency-Key K
│
├─ ¿K no existe?        → registrar K como "en_progreso", ejecutar, guardar respuesta, marcar "completada"
├─ ¿K existe y completada, mismo body?   → devolver la MISMA respuesta guardada (sin re-ejecutar)
├─ ¿K existe y en_progreso?              → 409 Conflict (ya se está procesando)
└─ ¿K existe pero body distinto?         → 422 (reutilizaron la clave con otro request: error del cliente)
```

Es el patrón que usan Stripe, Adyen, PayPal, etc.

### Tipo D: Control de concurrencia / condición de estado

Hacés que la operación **solo aplique si el estado es el esperado**. Si ya se aplicó, no hace nada.

```sql
-- Máquina de estados: solo se puede pagar un pedido pendiente
UPDATE pedidos SET estado = 'pagado'
WHERE id = $1 AND estado = 'pendiente';
-- filas afectadas = 1 → se aplicó; = 0 → ya estaba pagado (o cancelado): no repetir efectos
```

Variante con **versión (optimistic locking)**:

```sql
UPDATE productos SET stock = $1, version = version + 1
WHERE id = $2 AND version = $3;
```

### Tipo E: Deduplicación de mensajes (consumidores de colas / webhooks)

Cada mensaje trae un **id único**. El consumidor registra los ids ya procesados (patrón *Inbox* / *processed messages*) y descarta repetidos. Lo importante: **registrar el id y aplicar el efecto en la misma transacción**.

```sql
CREATE TABLE mensajes_procesados (
  message_id text PRIMARY KEY,
  processed_at timestamptz NOT NULL DEFAULT now()
);
```

### Resumen: ¿cuál elijo?

| Situación | Patrón |
|---|---|
| Podés redefinir la operación como "setear estado" | A |
| Hay un id natural/externo (pago del proveedor, email) | B |
| `POST` que crea algo o cobra, cliente propio / API pública | **C** |
| Transiciones de estado (pendiente → pagado) | D |
| Webhooks y consumidores de colas / eventos | E (+ B) |

Se combinan: una API de pagos suele usar **C** hacia afuera y **B + D** por dentro.

## 5. Cómo implementarlo (NestJS + PostgreSQL)

### 5.1 Tabla de claves

```sql
CREATE TABLE idempotency_keys (
  user_id         uuid        NOT NULL,
  key             text        NOT NULL,
  request_hash    text        NOT NULL,           -- hash del método + path + body
  status          text        NOT NULL CHECK (status IN ('en_progreso','completada')),
  response_status int,
  response_body   jsonb,
  created_at      timestamptz NOT NULL DEFAULT now(),
  PRIMARY KEY (user_id, key)                      -- la clave se scopea por usuario
);
```

Scopear por usuario evita que alguien "pise" la clave de otro. Programá una limpieza (TTL típico: 24 h a 7 días).

### 5.2 Servicio

```ts
import { Injectable, ConflictException, UnprocessableEntityException } from '@nestjs/common';
import { DataSource } from 'typeorm';
import { createHash } from 'crypto';

type Guardada = { status: number; body: unknown };

@Injectable()
export class IdempotencyService {
  constructor(private readonly ds: DataSource) {}

  hash(payload: unknown): string {
    return createHash('sha256').update(JSON.stringify(payload)).digest('hex');
  }

  /** Devuelve la respuesta guardada si ya se completó; null si hay que ejecutar. */
  async begin(userId: string, key: string, requestHash: string): Promise<Guardada | null> {
    // Intento reservar la clave de forma atómica
    const inserted = await this.ds.query(
      `INSERT INTO idempotency_keys (user_id, key, request_hash, status)
       VALUES ($1, $2, $3, 'en_progreso')
       ON CONFLICT (user_id, key) DO NOTHING
       RETURNING key`,
      [userId, key, requestHash],
    );
    if (inserted.length > 0) return null; // la reservé yo → ejecutar

    const [row] = await this.ds.query(
      `SELECT request_hash, status, response_status, response_body
         FROM idempotency_keys WHERE user_id = $1 AND key = $2`,
      [userId, key],
    );

    if (row.request_hash !== requestHash) {
      throw new UnprocessableEntityException('Idempotency-Key reutilizada con otro request');
    }
    if (row.status === 'en_progreso') {
      throw new ConflictException('Request en progreso, reintentá en unos segundos');
    }
    return { status: row.response_status, body: row.response_body };
  }

  async complete(userId: string, key: string, status: number, body: unknown) {
    await this.ds.query(
      `UPDATE idempotency_keys
          SET status = 'completada', response_status = $3, response_body = $4
        WHERE user_id = $1 AND key = $2`,
      [userId, key, status, JSON.stringify(body)],
    );
  }

  /** Si la operación falla de forma que querés permitir reintento, liberá la clave. */
  async release(userId: string, key: string) {
    await this.ds.query(
      `DELETE FROM idempotency_keys WHERE user_id = $1 AND key = $2 AND status = 'en_progreso'`,
      [userId, key],
    );
  }
}
```

### 5.3 Uso en un controller

```ts
@Post('pagos')
async crearPago(
  @Headers('idempotency-key') key: string,
  @Body() dto: CrearPagoDto,
  @Req() req,
  @Res({ passthrough: true }) res: Response,
) {
  if (!key) throw new BadRequestException('Falta el header Idempotency-Key');

  const userId = req.user.id;
  const hash = this.idem.hash({ m: 'POST', p: '/pagos', b: dto });

  const guardada = await this.idem.begin(userId, key, hash);
  if (guardada) {                       // repetición: devolvemos lo mismo
    res.status(guardada.status);
    return guardada.body;
  }

  try {
    const pago = await this.pagosService.crear(userId, dto);
    await this.idem.complete(userId, key, 201, pago);
    res.status(201);
    return pago;
  } catch (e) {
    await this.idem.release(userId, key); // permitir reintento si fue un error transitorio
    throw e;
  }
}
```

Más adelante podés moverlo a un **Interceptor** o decorator (`@Idempotent()`) para no repetirlo en cada endpoint.

### 5.4 Consumidor de cola con deduplicación (BullMQ / eventos)

```ts
async procesar(msg: { id: string; data: any }) {
  await this.ds.transaction(async (tx) => {
    const r = await tx.query(
      `INSERT INTO mensajes_procesados (message_id) VALUES ($1)
       ON CONFLICT DO NOTHING RETURNING message_id`,
      [msg.id],
    );
    if (r.length === 0) return;           // ya procesado → ignorar

    await this.aplicarEfecto(tx, msg.data); // efecto + registro: misma transacción
  });
}
```

En **BullMQ** podés pasar un `jobId` determinístico al encolar para evitar duplicados mientras el job exista:

```ts
await queue.add('enviar-mail', payload, { jobId: `mail:${pedidoId}` });
```

Ojo: eso deduplica el *encolado*, no reemplaza la idempotencia del *worker* (los jobs pueden reintentarse).

### 5.5 Redis como alternativa liviana

Para locks o dedupe de corta vida (no como única fuente de verdad en pagos):

```ts
// SET key valor NX EX 86400  → solo setea si no existe
const ok = await redis.set(`idem:${userId}:${key}`, 'en_progreso', 'EX', 86400, 'NX');
if (ok !== 'OK') { /* ya existe: leer resultado guardado o devolver 409 */ }
```

Si Redis se cae o evicta la clave, perdés la garantía. Para dinero, preferí Postgres.

## 6. Errores comunes

1. **Chequear antes de insertar** (`if (!existe) crear()`): tiene race condition. Usá constraint único / `ON CONFLICT`.
2. **Efectos secundarios fuera de la transacción:** guardás la clave pero el mail/cobro se manda dos veces si falla en el medio. Aplicá efecto + registro juntos, o hacé el efecto externo también idempotente (pasándole la misma clave al proveedor).
3. **No validar el hash del body:** la misma clave con otro payload devolvería una respuesta que no corresponde.
4. **Clave generada por el servidor:** tiene que generarla el **cliente** *una vez por intención*, y reutilizarla en cada reintento. Si el cliente genera una nueva en cada reintento, no sirve de nada.
5. **Claves globales sin scope de usuario.**
6. **Nunca expirar las claves:** la tabla crece sin límite.
7. **Cachear errores permanentes o transitorios sin criterio:** definí qué guardás (normalmente éxitos y errores de validación) y cuándo liberás la clave (errores 5xx transitorios).
8. **Confundir idempotencia con "ignorar el segundo request":** el segundo request debería devolver un resultado coherente (idealmente el mismo del primero).

## 7. Checklist para diseñar una operación idempotente

- [ ] ¿Qué pasa si esta operación se ejecuta 2 veces seguidas? ¿Y en paralelo?
- [ ] ¿Se puede expresar como "setear estado" (Tipo A)?
- [ ] ¿Hay un id natural/externo para una constraint única (Tipo B)?
- [ ] Si es un `POST` crítico (pagos, pedidos, envíos): ¿hay Idempotency-Key (Tipo C)?
- [ ] ¿Las transiciones de estado están protegidas con condición (`WHERE estado = ...`) (Tipo D)?
- [ ] ¿Los consumidores de colas/webhooks deduplican por id (Tipo E)?
- [ ] ¿El efecto y el registro de "ya hecho" están en la misma transacción?
- [ ] ¿Hay TTL/limpieza de las claves?
- [ ] ¿Probé: doble click, timeout + reintento, dos requests concurrentes con la misma clave, misma clave con otro body?

## 8. Cómo probarlo

Test mínimo (Jest + supertest): mandar el mismo request dos veces con la misma clave y verificar que **se creó un solo registro** y que **ambas respuestas son iguales**. Agregar un caso de concurrencia con `Promise.all([req(), req()])`: uno debe ganar y el otro recibir `409` (o la respuesta guardada).

```ts
it('no duplica el pago con la misma Idempotency-Key', async () => {
  const body = { monto: 5000, moneda: 'ARS' };
  const headers = { 'Idempotency-Key': 'test-key-1' };

  const r1 = await request(app.getHttpServer()).post('/pagos').set(headers).send(body);
  const r2 = await request(app.getHttpServer()).post('/pagos').set(headers).send(body);

  expect(r1.status).toBe(201);
  expect(r2.body).toEqual(r1.body);
  expect(await pagosRepo.count()).toBe(1);
});
```

## 9. Regla de oro

> **Asumí que todo se va a ejecutar más de una vez.** Diseñá cada operación que cambia estado para que repetirla sea inofensivo.
