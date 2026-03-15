# Clientes

O módulo de clientes mantém um diretório de clientes para vinculação com vendas.

## Status no MVP

No MVP, o cadastro de clientes é **read-only** na maioria das telas. A criação e edição são feitas diretamente no banco ou via import.

A vinculação de cliente a uma venda está disponível no PDV (campo "Identificar Cliente").

## Listagem de Clientes

**Rota**: `/customers`

Exibe todos os clientes ativos com:
- Nome
- CPF (formatado: xxx.xxx.xxx-xx)
- Telefone
- Limite de crédito (se `featureFlags.clientCredit == true`)

## Dados do Cliente

| Campo | Tipo | Descrição |
|---|---|---|
| `name` | TEXT | Nome completo ou razão social |
| `cpf` | TEXT (único) | CPF sem formatação (11 dígitos) |
| `phone` | TEXT | Telefone de contato |
| `email` | TEXT | Email (dado sensível — nunca logar) |
| `address` | TEXT | Endereço completo |
| `credit_limit` | INTEGER (centavos) | Limite de crédito na loja |
| `notes` | TEXT | Observações internas |

## Crédito na Loja (Feature Flag)

Disponível apenas quando `featureFlags.clientCredit == true`.

Permite que o cliente compre a prazo (conta corrente), descontando do limite de crédito. O pagamento aparece como `store_credit` nas formas de pagamento do PDV.

## Privacidade

- O `email` dos clientes é dado sensível — **nunca incluir em logs, analytics ou mensagens de erro**
- O `cpf` é armazenado sem formatação — formatado apenas na exibição

## Tabela

```
customers: id, name, cpf (único), phone, email, address,
           credit_limit (cents), notes, active,
           created_at, updated_at, deleted_at, sync_version
```

## Sincronização

Clientes usam **Last-Write-Wins** na sincronização entre terminais.

## Roadmap

Na v1.0, o cadastro completo de clientes será disponibilizado, incluindo:
- Formulário de criação/edição na UI
- Importação de lista de clientes
- Histórico de compras por cliente
- Extrato de crédito
