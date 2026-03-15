# Resolução de Conflitos

O `ConflictResolver` aplica estratégias diferentes dependendo da tabela, garantindo consistência de dados em cenários de escrita simultânea entre terminais.

## Estratégias por Tabela

| Tabela | Estratégia | Justificativa |
|---|---|---|
| `products` | Last-Write-Wins (LWW) | Produto é editado raramente; o sync_version garante a versão mais recente |
| `categories` | Last-Write-Wins (LWW) | Idem produtos |
| `customers` | Last-Write-Wins (LWW) | Idem produtos |
| `inventory.quantity` | **Delta Aditivo** | Vendas simultâneas devem acumular, não sobrescrever |
| `sales` | Imutável após `completed` | Venda finalizada nunca é sobrescrita |
| `cash_movements` | Append-only | Cada terminal gerencia seu próprio caixa |
| `inventory_movements` | Append-only | Histórico de movimentos é sempre cumulativo |
| Soft delete (`deleted_at`) | Delete vence | Propagação de deleção sempre prevalece sobre update |

## Last-Write-Wins (LWW)

```dart
// Aceitar a entrada se o sync_version recebido é maior que o local
bool shouldAccept(SyncLogEntry incoming, SyncLogEntry? local) {
  if (local == null) return true;
  return incoming.syncVersion > local.syncVersion;
}
```

Aplica-se a `products`, `categories`, `customers` e demais tabelas de dados mestre.

## Delta Aditivo (Inventário)

**Nunca aplicar LWW para `inventory.quantity`.**

```dart
double resolveInventoryQuantity({
  required double masterCurrentQty,
  required double clientInitialQty,
  required double clientFinalQty,
}) {
  final delta = clientFinalQty - clientInitialQty;
  return masterCurrentQty + delta;
}
```

### Exemplo de Conflito Simultâneo

```
Estoque inicial no master: 10 unidades

Terminal A inicia venda:  client_initial = 10
Terminal B inicia venda:  client_initial = 10 (ao mesmo tempo)

Terminal A finaliza (vendeu 3): client_final = 7
  Master aplica: 10 + (7 - 10) = 7

Terminal B finaliza (vendeu 2): client_final = 8
  Master aplica: 7 + (8 - 10) = 5  ✓

Resultado correto: 10 - 3 - 2 = 5
```

### Estoque Negativo (STOCK_EXHAUSTED)

Se o resultado da fórmula for `<= 0`, o master retorna `alert: "STOCK_EXHAUSTED"` na resposta do `/api/v1/stock/reserve`. A venda **já está confirmada** no terminal — o alerta é apenas informativo para o operador revisar o estoque.

## Modo Offline e Reconexão

Quando o master fica indisponível:

1. `SyncClient` muda para `SyncMode.offline`
2. Vendas continuam sendo registradas localmente
3. `sync_log` acumula entradas com `synced = false`
4. A cada 30 segundos, o cliente tenta redescobrir o master via mDNS

Ao reconectar:
1. `POST /api/v1/sync/push` com todos os registros com `synced = false`
2. Master aplica deltas aditivos para inventário
3. Se inventário ficar negativo após push: sinaliza para revisão gerencial (não reverte vendas)
4. `SyncClient` muda para `SyncMode.online`

## O que NÃO Fazer

- ❌ LWW para `inventory.quantity` — resulta em phantom inventory
- ❌ Bloquear a UI ou o commit da venda esperando confirmação de sync
- ❌ Reverter vendas após STOCK_EXHAUSTED — a venda é definitiva
- ❌ Eleição automática de master — é sempre manual pelo operador
