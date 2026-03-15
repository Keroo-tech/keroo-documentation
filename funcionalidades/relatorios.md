# Relatórios

O módulo de relatórios fornece visibilidade sobre as operações do dia e períodos anteriores.

## Tela de Relatórios

**Rota**: `/reports`

**Acesso**: operadores com role `manager` ou `admin`

## Relatório Diário (Dashboard)

Exibe o resumo do dia atual:

- **Total de vendas**: soma de `sales.total` com `status = 'completed'`
- **Número de transações**: contagem de vendas completadas
- **Ticket médio**: total / número de transações
- **Formas de pagamento**: breakdown por método (dinheiro, PIX, cartão, etc.)
- **Produtos mais vendidos**: top 10 por quantidade

## Fechamentos de Caixa por Período

Permite selecionar um período (data inicial e final) e visualizar:

- Lista de fechamentos de caixa no período
- Por fechamento: operador, horário, saldo calculado, diferença
- Totais consolidados do período

## Vendas por Período

- Lista de vendas no período selecionado
- Filtros por forma de pagamento, operador
- Total por período

## Como os Dados são Gerados

Os relatórios são calculados via queries no banco local (SQLite). Não há tabelas de agregação pré-computadas — os dados são sempre calculados em tempo real.

Queries utilizadas:
- `SalesDao.watchSalesByDateRange(startUtc, endUtc)` — vendas completadas no período
- `SalesDao.watchPaymentsByDateRange(startUtc, endUtc)` — pagamentos por período
- `CashRegistersDao.watchClosedByDateRange(startUtc, endUtc)` — caixas fechados

## Relatório de Fechamento de Caixa (Impresso)

Gerado automaticamente ao fechar o caixa pelo `document_module`. Inclui:
- Período do turno
- Total de vendas por forma de pagamento
- Sangrias e suprimentos
- Saldo calculado vs declarado
- Diferença

## Filtros Disponíveis

| Filtro | Tipo | Descrição |
|---|---|---|
| Período | Data inicial / final | Filtra por `created_at` UTC |
| Operador | Seleção | Filtra por `user_id` |
| Forma de pagamento | Multi-seleção | Filtra por método |

## Exportação

Exportação de relatórios (PDF, CSV) está planejada para a v1.0.
