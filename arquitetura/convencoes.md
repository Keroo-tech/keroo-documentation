# Convenções Globais

Regras que se aplicam a **todo o projeto** — banco de dados, código Dart, UI e API.

## Monetário

**Sempre `INTEGER` em centavos. Nunca `double` ou `REAL`.**

```
R$ 10,50 → 1050
R$ 0,99  → 99
R$ 100,00 → 10000
```

- Armazenamento: `INTEGER NOT NULL` no SQLite
- Dart: `int` (nunca `double`)
- Formatação para exibição: `intl.NumberFormat.currency(locale: 'pt_BR', symbol: 'R\$').format(cents / 100.0)`
- **Exceção**: quantidades de estoque (KG com decimais) usam `REAL`/`double`

## IDs

**Sempre `TEXT` UUID gerado no cliente. Nunca auto-increment.**

```sql
-- SQL
id TEXT PRIMARY KEY DEFAULT (lower(hex(randomblob(16))))
```

```dart
// Dart (gerado pelo Drift via withDefault)
TextColumn get id => text()
    .withDefault(const CustomExpression("lower(hex(randomblob(16)))"))();
```

- UUIDs são gerados no cliente (terminal), não no servidor
- Permite inserção offline e sync posterior sem conflito de chaves
- Formato: 32 caracteres hexadecimais em minúsculas (sem hifens)

## Datas e Timestamps

**Sempre `TEXT` no formato ISO-8601 UTC.**

```
2026-02-25T14:30:00.000Z
```

```sql
-- Geração automática no SQLite
created_at TEXT NOT NULL DEFAULT (strftime('%Y-%m-%dT%H:%M:%fZ','now'))
updated_at TEXT NOT NULL DEFAULT (strftime('%Y-%m-%dT%H:%M:%fZ','now'))
```

- Sempre UTC no armazenamento
- Conversão para horário local somente na camada de UI (para exibição)
- Milissegundos incluídos (`%f` no strftime)

## Soft Delete

**Nunca deletar fisicamente registros. Usar `deleted_at`.**

```sql
deleted_at TEXT  -- NULL = ativo, preenchido = deletado
```

```dart
TextColumn get deletedAt => text().nullable()();
```

- `NULL` = registro ativo
- Valor preenchido = registro deletado (o valor é o timestamp da deleção)
- Queries devem filtrar por `WHERE deleted_at IS NULL` para listar apenas ativos
- A propagação de soft delete via sync sempre vence sobre updates (`delete wins`)

## Sync Version

**Contador monotônico para controle de conflitos no sync.**

```sql
sync_version INTEGER NOT NULL DEFAULT 0
```

- Incrementado a cada mutation no master
- Usado pelo cliente para determinar quais registros receber no delta
- `SELECT * FROM sync_log WHERE sync_version > :last_known`

## Código Fonte

- **Idioma do código**: inglês (funções, classes, variáveis, comentários técnicos)
- **Idioma da UI**: português brasileiro (labels, mensagens, tooltips)
- **Idioma da documentação**: português brasileiro

## Campos Fiscais

Os campos fiscais existem no schema mas são controlados por feature flag:

```dart
// Sempre verificar antes de renderizar campos fiscais
if (featureFlags.fiscalNfce) {
  // mostrar NCM, CFOP, CST, ICMS
}
```

**Nunca usar na UI** os termos: "Nota Fiscal", "NF-e", "NFC-e", "Cupom Fiscal", "SAT", "DANFE" — estes são termos fiscais proibidos no MVP.

## Segurança

| Dado | Regra |
|---|---|
| PINs de operadores | Armazenados como bcrypt hash em `pin_hash`. Nunca em plain text. |
| CNPJ da licença | Nunca incluir em logs, analytics ou mensagens visíveis ao usuário |
| Email de clientes | Dado sensível — nunca logar |
| Chave privada RSA | Nunca no repositório — somente no servidor de licenciamento |
| XMLs fiscais | `fiscal_queue.xml_request/xml_response` — nunca sync sem criptografia |

## Tabelas Imutáveis (Append-Only)

`cash_movements` e `inventory_movements` são **append-only**:
- Protegidas por triggers SQLite que bloqueiam UPDATE e DELETE
- Nunca usar UPDATE ou DELETE nestas tabelas
- O saldo do caixa é sempre calculado: `SELECT SUM(amount) FROM cash_movements`
