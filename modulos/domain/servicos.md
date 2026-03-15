# Serviços de Domínio

Serviços de domínio são classes em `packages/domain` que encapsulam **lógica de negócio pura** — sem acesso a banco de dados, rede ou UI.

## CashRegisterService

Centraliza as regras de negócio relacionadas ao caixa.

### Responsabilidades

- Calcular o saldo do caixa a partir dos movimentos
- Validar se o caixa pode ser fechado
- Calcular a diferença entre valor declarado e calculado no fechamento

### Regra Fundamental

> O saldo do caixa **nunca é um campo armazenado**. É sempre calculado:
> ```sql
> SELECT SUM(amount) FROM cash_movements WHERE cash_register_id = :id
> ```

Movimentos negativos (saídas, sangrias) têm `amount` negativo. Movimentos positivos (vendas, suprimentos) têm `amount` positivo.

### Tipos de Movimentos

| Tipo (`type`) | Descrição | Sinal |
|---|---|---|
| `opening` | Abertura do caixa (fundo de troco) | + |
| `sale` | Pagamento em dinheiro de uma venda | + |
| `return` | Devolução de venda em dinheiro | - |
| `withdrawal` | Sangria (retirada) | - |
| `replenishment` | Suprimento (adição de dinheiro) | + |
| `discount` | Desconto manual aplicado | - |
| `closing` | Registro do fechamento | 0 |

## SaleService

Centraliza as regras de criação e validação de vendas.

### Responsabilidades

- Calcular totais do carrinho (subtotal, descontos, total)
- Validar formas de pagamento (soma deve cobrir o total)
- Calcular troco

### Cálculo de Troco

```dart
int calculateChange(int total, int amountPaid) {
  assert(amountPaid >= total, 'Pagamento insuficiente');
  return amountPaid - total;
}
```

### Validações

- Total da venda deve ser > 0
- Soma dos pagamentos deve ser ≥ total da venda
- Pelo menos um item no carrinho
- Produto deve ter `active == true`
- Caixa deve estar com `status == 'open'`

## Lógica de Conflito de Inventário

Implementada em `sync_engine/ConflictResolver` mas baseada em regras de domínio:

**Fórmula Aditiva** (para `inventory.quantity`):
```
novo_master = atual_master + (cliente_final - cliente_inicial)
```

Exemplo:
- Master tem 10 unidades
- Cliente A inicia venda com 10 (client_initial = 10)
- Cliente B inicia venda com 10 ao mesmo tempo
- Cliente A finaliza com 8 (vendeu 2): push `client_final = 8`
- Master aplica: `10 + (8 - 10) = 8` ✓
- Cliente B finaliza com 9 (vendeu 1): push `client_final = 9`
- Master aplica: `8 + (9 - 10) = 7` ✓ (saldo correto: 10 - 2 - 1 = 7)

Esta fórmula garante que **nunca** haja phantom inventory em vendas simultâneas.
