# Roadmap

Visão geral do que está implementado no MVP e o que vem na v1.0.

## MVP — Gestão e Comprovante (Sprints 1–10)

**Status**: Em desenvolvimento ativo

### O que está implementado

| Funcionalidade | Status |
|---|---|
| Setup inicial (loja + admin) | ✅ |
| Login por PIN 3 dígitos | ✅ |
| Gestão de usuários (admin) | ✅ |
| Cadastro de produtos e categorias | ✅ |
| Controle de estoque + histórico | ✅ |
| PDV — venda com carrinho | ✅ |
| Múltiplas formas de pagamento | ✅ |
| Comprovante ESC/POS (não fiscal) | ✅ |
| Abertura e fechamento de caixa | ✅ |
| Sangria e suprimento de caixa | ✅ |
| Relatório de fechamento impresso | ✅ |
| Diretório de clientes (read-only) | ✅ |
| Validação de licença RSA offline | ✅ |
| Sincronização multi-terminal (LAN) | ✅ |
| Configurações de hardware | ✅ |
| Relatórios por período | ✅ |
| Balança serial (pesáveis) | ✅ |

### Filosofia MVP

> "Gestão Primeiro, Fiscal Depois"

O MVP valida:
- Velocidade de registro de vendas
- Controle de estoque em tempo real
- Controle de caixa por turno
- Operação multi-terminal offline-first

**O que NÃO existe no MVP** (intencionalmente):
- Emissão de NF-e, NFC-e ou SAT
- Sincronização com nuvem (Supabase)
- Painel administrativo web
- Exportação de relatórios (PDF/CSV)
- Cadastro completo de clientes pela UI

---

## v1.0 — Emissor Fiscal (Sprints 11–16)

**Status**: Planejado

### Fiscal

| Funcionalidade | Observação |
|---|---|
| Emissão NF-e (Nota Fiscal Eletrônica) | Ativado via `featureFlags.fiscalNfce` |
| Emissão SAT (São Paulo) | Ativado via `featureFlags.fiscalSat` |
| Contingência fiscal | Armazenamento offline, emissão posterior |
| Cancelamento de NF-e | Prazo de 24h após emissão |
| DANFE NFC-e | Substituição do comprovante simples |
| Configurações fiscais (CNPJ, IE, regime) | Nova tela `/settings/fiscal` |
| Campos NCM, CFOP, CST por produto | Já existem no schema, ocultos no MVP |

> A ativação é feita remotamente via chave de licença — sem atualização do app.
> Todos os campos fiscais já estão no schema do banco (v1 do schema).

### Cloud (Pro Plan)

| Funcionalidade | Observação |
|---|---|
| Sync cloud via Supabase | `featureFlags.cloudSync` |
| Painel administrativo web | `featureFlags.adminDashboard` |
| Backup automático na nuvem | Parte do cloud sync |

### Cadastro de Clientes

| Funcionalidade | Observação |
|---|---|
| Formulário completo de criação/edição | CRUD completo na UI |
| Importação de clientes (CSV) | |
| Histórico de compras por cliente | |
| Crédito/conta corrente | `featureFlags.clientCredit` |

### Relatórios Avançados

| Funcionalidade | Observação |
|---|---|
| Exportação PDF | |
| Exportação CSV | |
| Gráficos de tendência | |
| Ranking de produtos | |

---

## Decisões Arquiteturais para Futuro

| Decisão | Impacto |
|---|---|
| Schema fiscal já presente (v1) | Upgrade sem re-entrada de dados pelo cliente |
| Feature flags via licença | Ativação remota sem update de app |
| `fiscal_queue` no schema | Fila de emissão pronta para ser processada |
| `sale_items.ncm/cfop/cst_csosn` | Snapshot fiscal armazenado em cada item de venda |
| Separação `packages/fiscal_module` | Código fiscal isolado, criado apenas no Sprint 11 |

---

## Não está no Roadmap

- Versão mobile (iOS/Android) — o keroo-pdv é exclusivamente desktop
- Versão web — desktop offline-first é a proposta de valor
- Integração com marketplaces (iFood, Rappi) — fora do escopo do PDV físico
