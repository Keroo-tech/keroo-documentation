# Stack Tecnológico

Tabela completa de tecnologias utilizadas no keroo-pdv com versões e justificativas.

## Stack Principal

| Camada | Tecnologia | Versão | Justificativa |
|---|---|---|---|
| UI / Desktop | Flutter | 3.24+ | Cross-platform Windows + Linux com uma única codebase |
| Linguagem | Dart | 3.5+ | Tipagem forte, null safety, geração de código |
| Gerenciador de monorepo | Melos | 7+ | Scripts compartilhados, bootstrap unificado, workspace Dart |
| Banco de dados local | Drift (SQLite) | latest | ORM type-safe para Dart; WAL mode para performance |
| Gerenciamento de estado | Riverpod | 3.x | Reatividade compile-safe com code generation (`@riverpod`) |
| Navegação | go_router | ^14.2.7 | Roteamento declarativo com guards de autenticação e feature flags |
| Servidor HTTP (sync) | shelf + shelf_router | latest | Servidor HTTP leve em Dart puro; sem dependência de framework externo |
| Descoberta de rede | multicast_dns | latest | Descoberta automática do terminal master via mDNS (DNS-SD) |
| Cliente HTTP | dio | ^5.4.3 | Chamadas HTTP para a API de sync do master |
| Criptografia | pointycastle | ^3.9.1 | RSA-2048 para validação offline de licença |
| Armazenamento seguro | flutter_secure_storage | ^9.2.2 | DPAPI (Windows) / SecretService (Linux) para chave de licença |
| Cloud (Pro, Sprint 15+) | supabase_flutter | ^2.5.0 | PostgreSQL + Realtime para sync cloud (plano Pro) |
| Impressora ESC/POS | esc_pos_utils (dart) | — | Geração de comandos ESC/POS para impressoras térmicas |
| Balança serial | libserialport | — | Comunicação serial com balanças Toledo/Filizola |
| Persistência de preferências | shared_preferences | ^2.3.0 | Cache de configurações de hardware e modo de sync |
| Logs | logger | ^2.3.0 | Logging estruturado em desenvolvimento |
| Internacionalização | intl | ^0.19.0 | Formatação de moeda (R$), datas no padrão brasileiro |

## Geração de Código

| Framework | Gerador | Artefatos gerados |
|---|---|---|
| Drift | `drift_dev` | `*.g.dart` (queries, companions, DAOs) |
| Riverpod | `riverpod_generator` | `*.g.dart` (providers) |
| Freezed | `freezed` | `*.freezed.dart` (modelos imutáveis, copyWith, ==) |
| build_runner | `build_runner` | Orquestra os geradores acima |

## Dependências Internas

```
pdv (app) → database, domain, document_module,
             hardware_module, sync_engine, license_module
database  → domain
sync_engine → domain
document_module → hardware_module
```

## Versões de SDK

```yaml
# Mínimo suportado
environment:
  sdk: ">=3.5.0 <4.0.0"
```

## Notas de Compatibilidade

- **Riverpod 3.x**: usa `.value` (não `.valueOrNull`) para acessar dados de `AsyncValue`
- **go_router 14+**: `GoRouterState.extra` tipado; use `state.extra as Type`
- **Drift + Workspace Dart**: `resolution: workspace` em cada `pubspec.yaml` resolve conflitos de versão automaticamente
- **pointycastle 3.x**: API de RSA via `RSASigner` com digest `SHA-256/RSA`
