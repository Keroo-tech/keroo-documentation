# Endpoints de Sincronização

Documentação completa dos 5 endpoints da API HTTP do `MasterServer`.

Base URL: `http://<ip-do-master>:7890`

---

## GET `/api/v1/info`

Retorna a identidade do terminal master. Usado pelo cliente para verificar que encontrou o master correto (mesmo `store_id`).

### Request

```
GET /api/v1/info
```

Sem parâmetros, sem body.

### Response `200 OK`

```json
{
  "store_id": "a1b2c3d4e5f6a1b2c3d4e5f6a1b2c3d4",
  "role": "master",
  "server_time": "2026-03-12T14:30:00.123Z"
}
```

| Campo | Tipo | Descrição |
|---|---|---|
| `store_id` | string | UUID da loja (do `store_configs`) |
| `role` | string | Sempre `"master"` |
| `server_time` | string | Timestamp UTC do servidor |

---

## GET `/api/v1/sync/status`

Retorna a versão máxima de sync disponível no master. O cliente usa isso para saber se há mudanças novas antes de fazer o delta completo.

### Request

```
GET /api/v1/sync/status
```

Sem parâmetros, sem body.

### Response `200 OK`

```json
{
  "master_sync_version": 142,
  "server_time": "2026-03-12T14:30:01.456Z"
}
```

| Campo | Tipo | Descrição |
|---|---|---|
| `master_sync_version` | integer | Maior `sync_version` no `sync_log` do master |
| `server_time` | string | Timestamp UTC |

---

## GET `/api/v1/sync/delta`

Retorna todos os registros do `sync_log` com `sync_version > from_version`, opcionalmente filtrado por tabelas.

### Request

```
GET /api/v1/sync/delta?from_version=100&tables=products,inventory,customers
```

| Parâmetro | Tipo | Obrigatório | Descrição |
|---|---|---|---|
| `from_version` | integer | Não (default: 0) | Retorna entradas com `sync_version > from_version` |
| `tables` | string | Não | Tabelas separadas por vírgula; vazio = todas |

### Response `200 OK`

```json
{
  "entries": [
    {
      "id": "uuid",
      "terminal_id": "terminal-uuid",
      "table_name": "products",
      "record_id": "product-uuid",
      "operation": "update",
      "payload": "{\"name\":\"Café Especial\",\"sale_price\":1250,\"sync_version\":101}",
      "sync_version": 101,
      "synced": true,
      "created_at": "2026-03-12T14:25:00.000Z"
    }
  ],
  "current_version": 101
}
```

| Campo | Tipo | Descrição |
|---|---|---|
| `entries` | array | Lista de `SyncLogEntry` |
| `entries[].table_name` | string | Tabela afetada (`products`, `inventory`, etc.) |
| `entries[].operation` | string | `"insert"`, `"update"`, `"delete"` |
| `entries[].payload` | string | JSON serializado com os dados do registro |
| `entries[].sync_version` | integer | Versão desta entrada |
| `current_version` | integer | Maior `sync_version` retornado |

---

## POST `/api/v1/sync/push`

Envia registros pendentes do cliente (com `synced = false`) para o master. O master aplica os registros ao seu `sync_log` e retorna quantos foram aceitos.

### Request

```
POST /api/v1/sync/push
Content-Type: application/json
```

```json
{
  "entries": [
    {
      "id": "uuid",
      "terminal_id": "terminal-b-uuid",
      "table_name": "sales",
      "record_id": "sale-uuid",
      "operation": "insert",
      "payload": "{\"id\":\"sale-uuid\",\"total\":4500,\"status\":\"completed\",...}",
      "sync_version": 0,
      "synced": false,
      "created_at": "2026-03-12T14:28:00.000Z"
    }
  ]
}
```

### Response `200 OK`

```json
{
  "accepted": 1
}
```

| Campo | Tipo | Descrição |
|---|---|---|
| `accepted` | integer | Número de entradas aceitas com sucesso |

> Entradas que falham são ignoradas silenciosamente (o cliente pode retentar).

### Response `400 Bad Request`

```json
{
  "error": "invalid JSON"
}
```

---

## POST `/api/v1/stock/reserve`

Reserva de estoque em tempo real durante uma venda. Chamado pelo cliente de forma **fire-and-forget** após a venda ser confirmada localmente.

O master aplica a fórmula **aditiva** para calcular a nova quantidade:
```
new_qty = master_current + (client_final - client_initial)
```

### Request

```
POST /api/v1/stock/reserve
Content-Type: application/json
```

```json
{
  "product_id": "product-uuid",
  "client_initial": 10.0,
  "client_final": 7.0
}
```

| Campo | Tipo | Descrição |
|---|---|---|
| `product_id` | string | UUID do produto |
| `client_initial` | number | Quantidade que o cliente viu no início da venda |
| `client_final` | number | Quantidade após a venda (client_initial - qtd_vendida) |

### Response `200 OK`

```json
{
  "accepted": true,
  "new_quantity": 7.0,
  "alert": null
}
```

```json
{
  "accepted": true,
  "new_quantity": -1.0,
  "alert": "STOCK_EXHAUSTED"
}
```

| Campo | Tipo | Descrição |
|---|---|---|
| `accepted` | boolean | Sempre `true` (venda não é revertida) |
| `new_quantity` | number | Nova quantidade no master após delta |
| `alert` | string or null | `"STOCK_EXHAUSTED"` se `new_quantity <= 0`, senão `null` |

### Response `400 Bad Request`

```json
{
  "error": "invalid JSON"
}
```

---

## Formato de `payload` nas Entradas de Sync

O campo `payload` é um JSON **serializado como string** (string com escape de aspas), não um objeto JSON aninhado diretamente.

Exemplo para produto:
```json
{
  "id": "product-uuid",
  "name": "Café Especial 250g",
  "sale_price": 1250,
  "cost_price": 800,
  "unit": "UN",
  "active": true,
  "weighable": false,
  "sync_version": 101,
  "updated_at": "2026-03-12T14:25:00.000Z"
}
```

O campo `deleted_at` preenchido indica soft delete:
```json
{
  "id": "product-uuid",
  "deleted_at": "2026-03-12T15:00:00.000Z",
  "sync_version": 105
}
```
