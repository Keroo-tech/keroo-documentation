# API de Sincronização

O `MasterServer` expõe uma API HTTP REST na porta **7890** para comunicação entre terminais na rede local.

## Características

- **Protocolo**: HTTP/1.1 (sem HTTPS — somente LAN)
- **Porta**: `7890` (TCP)
- **Bind**: `0.0.0.0` (todas as interfaces locais)
- **Formato**: JSON (`Content-Type: application/json`)
- **Autenticação**: Nenhuma por credencial — `store_id` no mDNS como identificador de loja
- **Descoberta**: mDNS (`_keroopdv._tcp`, UDP 224.0.0.251:5353)

> **Segurança**: esta API é projetada apenas para rede local (LAN). Nunca expor à internet.

## Endpoints

| Método | Path | Descrição |
|---|---|---|
| `GET` | `/api/v1/info` | Identidade e status do master |
| `GET` | `/api/v1/sync/status` | Versão máxima de sync disponível |
| `GET` | `/api/v1/sync/delta` | Delta de registros desde `from_version` |
| `POST` | `/api/v1/sync/push` | Envio de registros pendentes do cliente |
| `POST` | `/api/v1/stock/reserve` | Reserva de estoque em tempo real |

→ [Documentação completa dos endpoints](endpoints-sync.md)

## Descoberta via mDNS

O master anuncia o serviço a cada 30 segundos:

```
Tipo de serviço: _keroopdv._tcp
Nome da instância: keroo-pdv-master
TXT records:
  role=master
  store_id=<uuid-da-loja>
Porta: 7890
```

Os clientes usam `MDnsClient` do pacote `multicast_dns` para descobrir o master automaticamente.

## Fluxo de Uso Típico

```
Cliente inicializa
  → mDNS scan 5 segundos
  → GET /api/v1/info (verificar store_id)
  → GET /api/v1/sync/status (obter versão atual)
  → GET /api/v1/sync/delta?from_version=X (baixar mudanças)
  → POST /api/v1/sync/push (enviar pendentes locais)
  → Estado: Online

Durante venda (fire-and-forget):
  → POST /api/v1/stock/reserve

Retry se master offline:
  → mDNS scan a cada 30 segundos
```
