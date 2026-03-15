# Convenções do Banco de Dados

Regras obrigatórias que se aplicam a todas as tabelas do schema.

## Identificadores (PKs)

**Sempre `TEXT` UUID gerado no cliente. Nunca `INTEGER AUTO_INCREMENT`.**

```sql
id TEXT PRIMARY KEY DEFAULT (lower(hex(randomblob(16))))
```

```dart
// Drift
TextColumn get id => text()
    .withDefault(const CustomExpression("lower(hex(randomblob(16)))"))();
@override
Set<Column> get primaryKey => {id};
```

**Por quê?**
- Permite geração de IDs offline (sem servidor)
- Facilita a sincronização entre terminais (sem colisão de IDs)
- Escala para múltiplos terminais sem coordenação central

**Formato**: 32 caracteres hexadecimais em minúsculas, sem hifens
Exemplo: `a1b2c3d4e5f6a1b2c3d4e5f6a1b2c3d4`

---

## Valores Monetários

**Sempre `INTEGER` em centavos. Nunca `REAL` ou `DOUBLE`.**

```sql
sale_price INTEGER NOT NULL     -- R$ 10,50 → 1050
cost_price INTEGER DEFAULT 0    -- R$ 0,00 → 0
```

**Por quê?**
- Aritmética de ponto flutuante causa erros de arredondamento em dinheiro
- `0.1 + 0.2 ≠ 0.3` em ponto flutuante
- `INTEGER` é exato, sem surpresas

**Conversão para exibição**:
```dart
final formatted = NumberFormat.currency(
  locale: 'pt_BR',
  symbol: 'R\$',
).format(cents / 100.0);
```

**Exceção**: quantidades físicas (KG, L, m) usam `REAL` pois precisam de precisão decimal.

---

## Datas e Timestamps

**Sempre `TEXT` no formato ISO-8601 UTC.**

```sql
created_at TEXT NOT NULL DEFAULT (strftime('%Y-%m-%dT%H:%M:%fZ','now'))
updated_at TEXT NOT NULL DEFAULT (strftime('%Y-%m-%dT%H:%M:%fZ','now'))
deleted_at TEXT  -- NULL = ativo
```

**Formato**: `2026-02-25T14:30:00.000Z`
- `%Y-%m-%dT` — data
- `%H:%M:%f` — hora com milissegundos
- `Z` — indica UTC explicitamente

**Por quê?**
- SQLite não tem tipo nativo `DATETIME`
- ISO-8601 é ordenável lexicograficamente (ORDER BY funciona corretamente)
- UTC evita problemas de fuso horário em terminais configurados diferente

**Conversão para exibição** (fuso horário local):
```dart
final local = DateTime.parse(row.createdAt).toLocal();
```

---

## Soft Delete

**Nunca deletar fisicamente registros. Usar `deleted_at`.**

```sql
deleted_at TEXT  -- NULL = ativo, preenchido = deletado
```

```dart
TextColumn get deletedAt => text().nullable()();
```

**Queries devem sempre filtrar:**
```sql
WHERE deleted_at IS NULL  -- lista apenas ativos
```

**Por quê?**
- Mantém histórico para auditoria
- Permite reverter deleções acidentais
- Necessário para propagação correta no sync (delete wins)

**Propagação no sync**: quando `deleted_at` é preenchido, esta mudança **sempre prevalece** sobre qualquer update com sync_version anterior.

---

## Sync Version

**Contador monotônico para controle de versão no sync.**

```sql
sync_version INTEGER NOT NULL DEFAULT 0
```

- Incrementado pelo master a cada operação sincronizada
- Usado pelo cliente: `GET /sync/delta?from_version=X`
- Estratégia LWW: aceita se `incoming.sync_version > local.sync_version`

**Tabelas com sync_version**: `products`, `categories`, `customers`, `inventory`, `sales`
**Tabelas sem sync_version**: `cash_movements`, `inventory_movements` (append-only, não conflitam)

---

## Tabelas Append-Only

`cash_movements` e `inventory_movements` são **imutáveis após inserção**.

Protegidas por triggers SQLite criados na migração:

```sql
CREATE TRIGGER trg_cash_movements_no_update
BEFORE UPDATE ON cash_movements
BEGIN SELECT RAISE(ABORT, 'cash_movements is immutable'); END;

CREATE TRIGGER trg_cash_movements_no_delete
BEFORE DELETE ON cash_movements
BEGIN SELECT RAISE(ABORT, 'cash_movements is immutable'); END;
```

**Por quê?**
- Garante auditabilidade total do caixa
- Previne manipulação de dados financeiros
- O saldo é sempre calculado, nunca editado
