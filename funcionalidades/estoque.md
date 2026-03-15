# Estoque

O módulo de estoque controla as quantidades de produtos disponíveis para venda.

## Visão Geral

**Rota**: `/inventory`

A tela de estoque exibe todos os produtos com:
- Nome do produto
- Quantidade atual em estoque
- Quantidade mínima configurada
- Indicador visual de estoque baixo (quando `quantity < min_quantity`)

## Ajuste de Estoque

**Rota**: `/inventory/adjustment/:productId`

Permite corrigir a quantidade de um produto (inventário físico, perda, etc.):

1. Selecione o produto
2. Informe a nova quantidade
3. Selecione o motivo (obrigatório)
4. Adicione observação (opcional)
5. Confirme

O ajuste cria dois registros:
- Atualiza `inventory.quantity` para o novo valor
- Insere em `inventory_movements` com `type = 'adjustment'`

## Tipos de Movimentos de Estoque

| Tipo | Origem | Descrição |
|---|---|---|
| `in` | Cadastro de produto / recebimento | Entrada de estoque |
| `out` | Venda finalizada | Saída automática por venda |
| `adjustment` | Ajuste manual | Correção de inventário |
| `return` | Cancelamento de venda | Estorno de saída |

## Histórico de Movimentos

**Rota**: `/inventory/history/:productId`

Exibe os últimos 100 movimentos de um produto, com:
- Data e hora
- Tipo de movimento
- Quantidade movida
- Quantidade anterior e nova
- Operador responsável
- Observação (quando houver)

## Tabelas Relacionadas

### `inventory` (Posição Atual)

```
id, product_id, quantity (REAL), min_quantity (REAL),
updated_at, sync_version
```

Um registro por produto. Atualizado a cada venda ou ajuste manual.

### `inventory_movements` (Histórico — Append-Only)

```
id, product_id, type, quantity, previous_quantity, new_quantity,
reference_id, reference_type, user_id, terminal_id, created_at, notes
```

**Nunca** alterar ou deletar registros desta tabela. Protegida por triggers SQLite.

## Estoque Negativo

Em cenário multi-terminal com venda simultânea, o estoque pode momentaneamente ficar negativo. Isso é **esperado e aceitável** — a venda não é revertida.

O sistema sinaliza o alerta `STOCK_EXHAUSTED` (via sync) para que o gerente tome providências (reposição ou ajuste).

## Sincronização do Estoque

A quantidade de estoque é sincronizada entre terminais usando **delta aditivo** (não Last-Write-Wins):

```
novo_master = atual_master + (cliente_final - cliente_inicial)
```

Veja [resolução de conflitos](../modulos/sync-engine/resolucao-conflitos.md) para detalhes.
