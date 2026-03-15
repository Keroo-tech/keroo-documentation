# Migrações do Banco de Dados

O keroo-pdv usa um sistema de migrações **versionadas** para evoluir o schema sem perder dados dos clientes.

## Versão Atual

`schemaVersion = 5` (definido em `AppDatabase`)

## Princípio Fundamental

**Nunca altere uma tabela existente diretamente.** Toda alteração de schema deve:
1. Incrementar `schemaVersion`
2. Adicionar uma cláusula `if (from < N)` na estratégia de migração

## Histórico de Versões

| Versão | Alteração |
|---|---|
| 1 | Schema inicial: 14 tabelas base |
| 2 | `users`: adicionada coluna `operator_code TEXT DEFAULT '000'` |
| 3 | Nova tabela `store_configs` |
| 4 | `inventory_movements`: adicionada coluna `notes TEXT` (nullable) |
| 5 | `store_configs`: adicionada coluna `paper_width TEXT DEFAULT 'mm80'` |

## Como Criar uma Migração

### Passo 1: Alterar a tabela Drift

Em `packages/database/lib/src/tables/<tabela>.dart`, adicione a nova coluna:

```dart
// Exemplo: adicionando 'notes' a inventory_movements
TextColumn get notes => text().nullable()();
```

### Passo 2: Registrar a migração

Em `packages/database/lib/src/migrations/migration_strategy.dart`:

```dart
Future<void> _applyMigrations(Migrator m, int from, int to) async {
  // ...migrações anteriores...
  if (from < 5) {
    await m.addColumn(storeConfigs, storeConfigs.paperWidth);
  }
  // Nova migração:
  if (from < 6) {
    await m.addColumn(inventoryMovements, inventoryMovements.notes);
  }
}
```

### Passo 3: Incrementar schemaVersion

Em `packages/database/lib/src/database.dart`:

```dart
@override
int get schemaVersion => 6;  // era 5
```

### Passo 4: Re-gerar o código

```bash
melos run build_runner
```

## Triggers Append-Only

As tabelas `cash_movements` e `inventory_movements` são protegidas por triggers criados na migração:

```sql
-- Criado em migration_strategy.dart → onCreate
CREATE TRIGGER trg_cash_movements_no_update
BEFORE UPDATE ON cash_movements
BEGIN SELECT RAISE(ABORT, 'cash_movements is immutable'); END;

CREATE TRIGGER trg_cash_movements_no_delete
BEFORE DELETE ON cash_movements
BEGIN SELECT RAISE(ABORT, 'cash_movements is immutable'); END;

-- Idem para inventory_movements
```

## PRAGMAs de Configuração

Executados no `beforeOpen` (antes de qualquer query):

```dart
await db.customStatement('PRAGMA journal_mode = WAL');
await db.customStatement('PRAGMA synchronous = NORMAL');
await db.customStatement('PRAGMA foreign_keys = ON');
await db.customStatement('PRAGMA temp_store = MEMORY');
await db.customStatement('PRAGMA cache_size = -64000');
await db.customStatement('PRAGMA busy_timeout = 5000');
```

## O que NÃO Fazer

- ❌ `customStatement('ALTER TABLE ...')` fora do processo de migração
- ❌ Modificar uma tabela existente sem criar migração
- ❌ Usar `ALTER TABLE DROP COLUMN` — SQLite tem suporte limitado; prefira criar nova tabela
- ❌ Remover uma cláusula `if (from < N)` mesmo que pareça "antiga" — pode afetar upgrades de versões antigas
