# Configurações

**Rota**: `/settings`

A tela de configurações centraliza as opções de configuração do sistema.

## Configurações da Loja

**Rota**: `/settings/store-config`

Dados do estabelecimento exibidos no comprovante:

| Campo | Descrição |
|---|---|
| Nome da empresa | Aparece no cabeçalho do comprovante |
| CNPJ | Somente dígitos; formatado na exibição |
| Endereço | Endereço completo do estabelecimento |
| Telefone | Contato do estabelecimento |
| Largura do papel | `mm58` (32 chars) ou `mm80` (48 chars) |

Armazenado em `store_configs` (tabela de registro único — sempre uma linha).

## Configurações de Hardware

**Rota**: `/settings/hardware`

| Configuração | Descrição |
|---|---|
| Tipo de impressora | Null / Windows (`winspool`) / Serial |
| Nome/porta da impressora | Nome do dispositivo no SO ou porta serial |
| Tipo de balança | Null / Serial (Toledo / Filizola) |
| Porta da balança | Porta serial (ex: `COM3`, `/dev/ttyUSB0`) |

Armazenado via `shared_preferences` (não no SQLite — são configurações de hardware físico, específicas por terminal).

## Licença

**Rota**: `/settings/license`

- Exibe a licença ativa (plano, CNPJ, validade, terminais permitidos, feature flags)
- Campo para inserir/trocar a chave de licença
- Botão "Validar Licença"

Esta tela é acessível mesmo sem licença válida (não tem guard de licença) — é usada como tela de bloqueio quando a licença está ausente ou expirada.

## Usuários

**Rota**: `/settings/users`

**Acesso**: somente `admin`

- Listagem de todos os operadores
- Criar novo operador
- Editar dados e PIN de operador existente
- Ativar / desativar operador

## Configurações Fiscais

**Rota**: `/settings/fiscal`

**Disponível somente quando** `featureFlags.fiscalNfce == true` (v1.0+)

No MVP, acessar esta rota redireciona automaticamente para `/settings`.

## Sincronização (Status)

**Rota**: `/sync/status`

Exibe o status atual da sincronização:
- Modo atual: Online / Offline
- IP e porta do master (quando online)
- Número de registros pendentes de sincronização
- Botão "Sincronizar Agora" (forçar sync manual)
- Botão "Promover a Master" / "Rebaixar de Master"
