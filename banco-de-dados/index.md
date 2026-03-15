# Banco de Dados

O keroo-pdv usa **SQLite** em modo **WAL (Write-Ahead Logging)** como banco de dados local, gerenciado via **Drift ORM**.

## Arquivo do Banco

```
keroo_pdv.db
```

Localização:
- **Windows**: `%USERPROFILE%\Documents\keroo_pdv.db`
- **Linux**: `~/Documents/keroo_pdv.db`

## Por que SQLite?

- **Offline-first**: sem servidor, sem dependência de rede
- **Confiabilidade**: ACID, transações atômicas
- **Performance**: WAL mode permite leituras concorrentes sem bloquear escritas
- **Portabilidade**: um único arquivo fácil de fazer backup

## Modo WAL

O WAL (Write-Ahead Logging) foi escolhido porque:
- Leituras nunca bloqueiam escritas
- Escritas nunca bloqueiam leituras
- Melhor performance em PDV (muitas leituras pequenas + escritas de venda)

## Convenções do Schema

| Convenção | Regra |
|---|---|
| **IDs** | `TEXT` UUID gerado no cliente |
| **Dinheiro** | `INTEGER` em centavos |
| **Quantidades físicas** | `REAL` (suporte a KG) |
| **Datas** | `TEXT` ISO-8601 UTC |
| **Soft delete** | `deleted_at TEXT NULL` |
| **Sync** | `sync_version INTEGER` |

## Versão do Schema

Versão atual: **5**

Cada versão representa uma migração de schema. Veja [../modulos/database/migracoes.md](../modulos/database/migracoes.md).

## Seções

- [Referência completa das tabelas](tabelas.md)
- [Convenções de schema](convencoes.md)
