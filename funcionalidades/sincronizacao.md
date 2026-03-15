# Sincronização

O sistema de sincronização permite que múltiplos terminais operem na mesma rede local (LAN), compartilhando produtos, clientes, estoque e configurações em tempo real.

## Conceitos Fundamentais

- **Offline-first**: toda venda é confirmada localmente antes de qualquer sync
- **Manual master**: o operador define qual terminal é o master
- **Sem internet**: funciona apenas em rede local (LAN)
- **Não-bloqueante**: sync acontece em background, nunca bloqueia o caixa

## Tela de Sincronização

**Rota**: `/sync/status`

Exibe:
- **Status**: Online (conectado ao master) ou Offline
- **Master**: IP:porta do terminal master (quando online)
- **Pendentes**: número de registros aguardando sync
- **Última sincronização**: timestamp do último sync bem-sucedido

Ações:
- **Sincronizar Agora**: força uma rodada de sync imediata
- **Promover a Master**: torna este terminal o master da loja
- **Rebaixar de Master**: para o servidor HTTP deste terminal

## Configurar Multi-terminal

### Terminal Master

1. Vá em **Configurações → Sincronização**
2. Clique em "Promover a Master"
3. O sistema inicia o servidor HTTP na porta 7890
4. O terminal começa a anunciar via mDNS

### Terminais Client

1. Nos outros terminais, vá em **Configurações → Sincronização**
2. O terminal client descobre o master automaticamente via mDNS
3. O status muda para "Online" quando a conexão é estabelecida

### Descoberta Manual

Se o mDNS não funcionar na rede, é possível configurar o IP do master manualmente (campo de IP direto na tela de sync).

## Dados Sincronizados

| Tabela | Estratégia | Direção |
|---|---|---|
| `products` | Last-Write-Wins | Bidirecional |
| `categories` | Last-Write-Wins | Bidirecional |
| `customers` | Last-Write-Wins | Bidirecional |
| `inventory` | Delta aditivo | Bidirecional |
| `sales` | Imutável após completed | Client → Master |
| `store_configs` | Last-Write-Wins | Master → Client |
| `users` | Last-Write-Wins | Master → Client |

## Soft Delete no Sync

A propagação de deleção (campo `deleted_at` preenchido) **sempre prevalece** sobre um update. Se o master deletou um produto e o client atualizou, o delete ganha.

## Indicador de Status na Interface

Um ícone na barra lateral indica o status de sincronização em tempo real:
- 🟢 Online — conectado ao master
- 🔴 Offline — sem master disponível
- 🟡 Sincronizando — sync em andamento
- Número badge com registros pendentes

## Resolução de Conflitos

Veja a documentação completa em [modulos/sync-engine/resolucao-conflitos.md](../modulos/sync-engine/resolucao-conflitos.md).
