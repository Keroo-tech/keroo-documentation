# Módulo: database

O pacote `database` é a camada de persistência do keroo-pdv. Implementa o schema SQLite via **Drift ORM** e expõe repositórios concretos para o restante do sistema.

## Responsabilidades

- Definição de todas as **15 tabelas** do schema
- **DAOs** (Data Access Objects) com queries type-safe
- **Migrações** de schema versionadas
- **Implementações de repositório** (interfaces definidas em `domain`)
- Configuração de PRAGMAs SQLite (WAL, foreign keys, cache)

## Dependências

```
domain  → interfaces de repositório (ProductRepository, SaleRepository, etc.)
drift   → ORM + code generation
path_provider → localização do arquivo .db no sistema de arquivos
```

## SQLite WAL Mode

O banco opera em **WAL (Write-Ahead Logging)** mode, configurado via PRAGMA no `beforeOpen`:

```sql
PRAGMA journal_mode = WAL;      -- performance de escrita
PRAGMA synchronous = NORMAL;    -- equilíbrio entre segurança e velocidade
PRAGMA foreign_keys = ON;       -- integridade referencial
PRAGMA temp_store = MEMORY;     -- tabelas temporárias em RAM
PRAGMA cache_size = -64000;     -- 64 MB de cache
PRAGMA busy_timeout = 5000;     -- aguarda 5s antes de retornar BUSY
```

## Arquivo do Banco

O banco é armazenado em:
- **Windows**: `%USERPROFILE%\Documents\keroo_pdv.db`
- **Linux**: `~/Documents/keroo_pdv.db`

(via `getApplicationDocumentsDirectory()` do `path_provider`)

## Versão do Schema

A versão atual é **5**. Cada alteração de schema gera uma nova versão e uma migração correspondente em `migration_strategy.dart`.

## Estrutura do Pacote

```
packages/database/lib/src/
├── database.dart          → AppDatabase (@DriftDatabase, schemaVersion)
├── tables/                → Definições das 15 tabelas
├── daos/                  → 9 DAOs (um por agregado de domínio)
├── repositories/          → Implementações concretas das interfaces domain
└── migrations/
    └── migration_strategy.dart → Estratégia de migração + PRAGMAs
```

## Seções Relacionadas

- [Schema completo — 15 tabelas](schema.md)
- [Migrações e versionamento](migracoes.md)
- [Referência de tabelas](../../banco-de-dados/tabelas.md)
