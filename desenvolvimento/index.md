# Desenvolvimento

Guia para desenvolvedores que contribuem com o keroo-pdv.

## Configuração do Ambiente

→ [Configuração do ambiente de desenvolvimento](configuracao-ambiente.md)

## Geração de Código

→ [Drift + Riverpod + Freezed](geracao-de-codigo.md)

## Testes

→ [Estratégias de teste e padrões](testes.md)

## Convenções

→ [Convenções de código e proibições](convencoes.md)

## Workflow

1. **Antes de codar**: leia o `CLAUDE.md` do diretório em que vai trabalhar
2. **Ao modificar `.dart`**: rode `melos run analyze` e corrija todos os problemas
3. **Ao alterar schema**: crie migração, incremente `schemaVersion`, re-rode `build_runner`
4. **Ao criar pacote**: crie `README.md` + `CLAUDE.md` imediatamente
5. **Ao criar provider**: use `@riverpod`, nunca manual constructor

## Arquivos Críticos

| Arquivo | Por que é crítico |
|---|---|
| `packages/database/lib/src/database.dart` | Entry point Drift: tabelas, DAOs, schemaVersion |
| `packages/domain/lib/src/models/` | Modelos centrais usados por todos os módulos |
| `apps/pdv/lib/features/pos/screens/pos_screen.dart` | Tela principal de venda |
| `packages/document_module/lib/src/comprovante_builder.dart` | Geração do comprovante ESC/POS |
| `packages/license_module/lib/src/feature_flags.dart` | Guard central de funcionalidades |
| `packages/sync_engine/lib/src/master_server.dart` | Servidor HTTP multi-terminal |

## Regras Invioláveis

- **Monetário**: sempre `INTEGER` em centavos, nunca `double`
- **IDs**: sempre UUID `TEXT`, nunca auto-increment
- **Fiscal**: verificar `featureFlags.fiscalNfce` antes de qualquer lógica fiscal
- **Schema**: sempre criar migração ao alterar tabelas, nunca `ALTER TABLE` direto
- **Testes**: sempre usar `NullPrinter`/`NullScale`, nunca assumir hardware presente
- **Sync**: `cash_movements` e `inventory_movements` são append-only
- **Providers**: usar `@riverpod` (code gen), nunca constructors manuais
