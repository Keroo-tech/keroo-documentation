# Caixa

O módulo de caixa controla a abertura, operação e fechamento do caixa físico. Cada turno de trabalho corresponde a um registro em `cash_registers`.

## Abertura de Caixa

**Rota**: `/cashier/open`

1. Informe o valor de abertura (troco inicial em dinheiro)
2. Confirme

O sistema cria um registro em `cash_registers` com `status = 'open'` e um movimento de `opening` em `cash_movements`.

> Apenas um caixa pode estar aberto por terminal ao mesmo tempo.

## Tela de Caixa

**Rota**: `/cashier`

Exibe:
- Status do caixa (aberto/fechado)
- Saldo atual (calculado em tempo real via `SUM(cash_movements)`)
- Lista de movimentos recentes
- Botões de ação: Sangria, Suprimento, Fechar Caixa

## Movimentos de Caixa

A tabela `cash_movements` é **append-only** — nunca se altera ou deleta registros.

| Tipo | Descrição | Quando ocorre |
|---|---|---|
| `opening` | Abertura do caixa | Ao abrir caixa |
| `sale` | Venda em dinheiro | Ao finalizar venda paga em cash |
| `return` | Devolução em dinheiro | Ao cancelar venda paga em cash |
| `withdrawal` | Sangria | Retirada de dinheiro durante o turno |
| `replenishment` | Suprimento | Adição de dinheiro durante o turno |
| `discount` | Desconto manual | Ajuste operacional |
| `closing` | Fechamento | Ao fechar caixa |

## Sangria (Withdrawal)

Remove dinheiro do caixa para depósito ou guarda:

1. Clique em "Sangria"
2. Informe o valor
3. Opcionalmente, adicione uma observação
4. Confirme

Cria um `CashMovement` com `type = 'withdrawal'` e `amount` negativo.

## Suprimento (Replenishment)

Adiciona dinheiro ao caixa (troco extra, etc.):

1. Clique em "Suprimento"
2. Informe o valor
3. Confirme

Cria um `CashMovement` com `type = 'replenishment'` e `amount` positivo.

## Fechamento de Caixa

**Rota**: `/cashier/close`

1. O sistema calcula o saldo calculado (`SUM(cash_movements)`)
2. O operador informa o valor contado fisicamente (`closing_amount_declared`)
3. O sistema exibe a diferença
4. Confirme

Ao fechar:
- `status` → `'closed'`
- `closed_at` → timestamp atual
- `closed_by_user_id` → usuário logado
- `closing_amount_calculated` → saldo pelo sistema
- `difference` → declarado - calculado

## Relatório de Fechamento

Após o fechamento, um relatório é gerado automaticamente pelo `document_module` e enviado para impressão. Inclui:
- Período do turno
- Totais por forma de pagamento
- Sangrias e suprimentos
- Saldo calculado vs declarado
- Diferença

## Regra Fundamental

> O saldo do caixa **nunca é um campo armazenado**. Sempre calculado:
> ```sql
> SELECT SUM(amount) FROM cash_movements WHERE cash_register_id = :id
> ```
