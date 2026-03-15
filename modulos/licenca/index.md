# Módulo: license_module

O pacote `license_module` gerencia a validação offline de licenças e as feature flags derivadas.

## Validação RSA-2048 Offline

A licença é validada **sem conexão com a internet** usando criptografia RSA-2048:

```
GERAÇÃO (servidor de licenciamento — nunca no cliente):
  1. Montar payload JSON: { id, cnpj, plan, expiresAt, maxTerminals, features }
  2. Assinar com RSA-2048 PKCS#1 v1.5 SHA-256 (chave privada no servidor)
  3. license_key = base64url(payload) + "." + base64url(signature)

VALIDAÇÃO (cliente, sem internet):
  1. Separar no último "." → payloadBytes + sigBytes
  2. Verificar assinatura com chave pública embarcada no binário
  3. Se inválida → LicenseInvalid
  4. Deserializar JSON → LicensePayload
  5. Se expiresAt < now → LicenseExpired
  6. Extrair FeatureFlags do campo "features" → LicenseValid
```

A chave pública é **embarcada no binário compilado**. A chave privada fica **exclusivamente no servidor de licenciamento** — nunca no repositório.

## Formato da Chave de Licença

```
<base64url(json_payload)>.<base64url(rsa_sha256_signature)>
```

Payload JSON:
```json
{
  "id": "uuid",
  "cnpj": "00000000000100",
  "plan": "basic",
  "expiresAt": "2027-01-01T00:00:00.000Z",
  "maxTerminals": 1,
  "features": {
    "fiscalNfce": false,
    "fiscalSat": false,
    "cloudSync": false,
    "adminDashboard": false,
    "hardwarePrinter": true,
    "clientCredit": false
  }
}
```

## LicenseStatus (Sealed Class)

```dart
sealed class LicenseStatus {}

final class LicenseValid extends LicenseStatus {
  final LicensePayload payload;
  final FeatureFlags features;
}

final class LicenseExpired extends LicenseStatus {
  final LicensePayload payload;
}

final class LicenseInvalid extends LicenseStatus {
  final String reason;
}

final class LicenseNotSet extends LicenseStatus {}
```

## FeatureFlags (MVP vs v1.0)

| Flag | MVP | Pro | v1.0 |
|---|---|---|---|
| `fiscalNfce` | `false` | `false` | `true` |
| `fiscalSat` | `false` | `false` | `true` |
| `cloudSync` | `false` | `true` | `true` |
| `adminDashboard` | `false` | `true` | `true` |
| `hardwarePrinter` | `true` | `true` | `true` |
| `clientCredit` | `false` | `true` | `true` |
| `maxTerminals` | `1` | `1–N` | `1–N` |

## Armazenamento da Chave

A chave de licença é armazenada via `flutter_secure_storage`:
- **Windows**: DPAPI (criptografia baseada nas credenciais do usuário Windows)
- **Linux**: SecretService (GNOME Keyring / KWallet)

Nunca armazenada em plain text ou em arquivos acessíveis.

## Planos Disponíveis

| Plano | Descrição |
|---|---|
| `basic` | 1 terminal, hardware_printer habilitado, sem fiscal |
| `pro` | Multi-terminal, cloud sync, admin dashboard |
| `enterprise` | Pro + suporte dedicado + SLA |

## Componentes do Pacote

| Arquivo | Classe | Descrição |
|---|---|---|
| `feature_flags.dart` | `FeatureFlags` | Flags derivadas + `fromJson()` + `FeatureFlags.mvp()` |
| `license_payload.dart` | `LicensePayload` | Payload JSON deserializado |
| `license_status.dart` | `LicenseStatus` | Sealed class de estado |
| `license_validator.dart` | `LicenseValidator` | Validação RSA-SHA256 |
| `license_storage.dart` | `LicenseStorage` | Wrapper do flutter_secure_storage |

## Segurança

- ❌ Nunca expor a chave privada RSA no repositório
- ❌ Nunca alterar feature flags em runtime sem revalidar a assinatura
- ❌ Nunca incluir o CNPJ em logs ou mensagens de erro visíveis ao usuário
- ❌ Usar `LicenseValidator()` (chave embarcada) em testes — usar `LicenseValidator.withKey()`
