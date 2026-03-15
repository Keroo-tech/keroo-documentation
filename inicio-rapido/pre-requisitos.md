# Pré-requisitos

## Sistema Operacional

O keroo-pdv suporta:

| SO | Versão Mínima | Observação |
|---|---|---|
| Windows | 10 (64-bit) | Driver de impressora via `winspool` |
| Linux | Ubuntu 22.04+ / Debian 12+ | Impressora via `libusb` ou serial |

## Flutter e Dart

| Ferramenta | Versão | Instalação |
|---|---|---|
| Flutter SDK | 3.24+ | [flutter.dev/docs/get-started](https://flutter.dev/docs/get-started/install) |
| Dart SDK | 3.5+ | Incluído no Flutter SDK |

Verifique a instalação:
```bash
flutter --version
dart --version
flutter doctor
```

O `flutter doctor` deve mostrar o target de desktop habilitado:
```
[✓] Flutter (Channel stable, 3.x.x)
[✓] Windows Version (...)   ← ou Linux
[✓] Visual Studio - develop Windows apps
```

## Melos

Melos é o gerenciador de monorepo. Instale globalmente via Dart:

```bash
dart pub global activate melos
```

Verifique:
```bash
melos --version
```

## Git

```bash
git --version  # qualquer versão recente
```

## Hardware (Opcional)

Para uso de impressora térmica ou balança:

| Hardware | Requisito |
|---|---|
| Impressora térmica ESC/POS | Driver instalado no SO (Windows: `winspool`) |
| Balança serial | `libserialport` — dependência nativa incluída |
| Gaveta de dinheiro | Conectada via impressora (comando ESC/POS) |

Em ambiente de desenvolvimento, todas as interações de hardware podem ser simuladas pelas implementações nulas (`NullPrinter`, `NullScale`) — nenhum hardware físico é necessário para desenvolver.

## Configuração de Rede (Multi-terminal)

Para usar sincronização entre terminais:

- Todos os terminais devem estar na **mesma rede local (LAN)**
- Porta **TCP 7890** deve estar liberada no firewall do terminal master
- Multicast DNS (mDNS) deve funcionar na rede — porta UDP 5353
