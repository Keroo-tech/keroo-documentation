# API do Sync Engine

O `MasterServer` expõe 5 endpoints HTTP na porta **7890**. Todos retornam `Content-Type: application/json`.

Para a documentação completa com exemplos de request/response, veja [api/endpoints-sync.md](../../api/endpoints-sync.md).

## Resumo dos Endpoints

| Método | Path | Descrição |
|---|---|---|
| `GET` | `/api/v1/info` | Identidade do master (store_id, server_time) |
| `GET` | `/api/v1/sync/status` | Versão máxima de sync disponível |
| `GET` | `/api/v1/sync/delta` | Entradas novas desde `from_version` |
| `POST` | `/api/v1/sync/push` | Envia registros pendentes do cliente |
| `POST` | `/api/v1/stock/reserve` | Reserva de estoque durante venda |

## Configuração de Rede

- **Porta**: `7890` (TCP)
- **Bind**: `InternetAddress.anyIPv4` — todas as interfaces locais
- **mDNS**: serviço `_keroopdv._tcp`, anunciado via UDP multicast `224.0.0.251:5353`
- **Intervalo de anúncio mDNS**: 30 segundos

## Autenticação

Não há autenticação por credenciais de usuário entre terminais. O `store_id` no TXT record do mDNS e na resposta de `/api/v1/info` serve como identificador de que o master pertence à mesma loja.

> A API é exposta apenas na LAN — nunca na internet.

## Formato das Entradas de Sync

```json
{
  "id": "uuid",
  "terminal_id": "terminal-uuid",
  "table_name": "products",
  "record_id": "product-uuid",
  "operation": "update",
  "payload": "{\"name\":\"Produto X\",\"sale_price\":1050,...}",
  "sync_version": 42,
  "synced": false,
  "created_at": "2026-03-12T10:30:00.000Z"
}
```

O `payload` é um JSON serializado como string (escape duplo) contendo os campos modificados do registro.
