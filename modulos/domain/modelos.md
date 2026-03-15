# Modelos de Domínio

Os modelos em `packages/domain` são classes **imutáveis** geradas com `@freezed`. Representam as entidades de negócio do keroo-pdv.

## Principais Modelos

### User (Usuário / Operador)

```dart
@freezed
class User with _$User {
  const factory User({
    required String id,
    required String name,
    String? email,
    required String pinHash,      // bcrypt — nunca plain text
    required String operatorCode, // "001", "002" — código de login
    required UserRole role,
    required bool active,
    required String createdAt,
    required String updatedAt,
    String? deletedAt,
  }) = _User;
}

enum UserRole { admin, operator, manager }
```

### Product (Produto)

```dart
@freezed
class Product with _$Product {
  const factory Product({
    required String id,
    String? barcode,
    String? internalCode,
    required String name,
    required ProductUnit unit,
    required int costPrice,       // centavos
    required int salePrice,       // centavos
    String? categoryId,
    String? ncm,                  // [FISCAL] — oculto quando !featureFlags.fiscalNfce
    String? cfop,                 // [FISCAL]
    String? cstCsosn,             // [FISCAL]
    int? icmsRate,                // [FISCAL] — em centésimos de percentual
    required bool active,
    required bool weighable,      // produto pesável (balança)
    required String createdAt,
    required String updatedAt,
    String? deletedAt,
    required int syncVersion,
  }) = _Product;
}

enum ProductUnit { un, kg, l, m, box, pkg }
```

### Sale (Venda)

```dart
@freezed
class Sale with _$Sale {
  const factory Sale({
    required String id,
    required int number,
    required String cashRegisterId,
    required String terminalId,
    required String userId,
    String? customerId,
    required SaleStatus status,
    required int subtotal,        // centavos
    required int discount,        // centavos
    required int total,           // centavos
    required int amountPaid,      // centavos
    required int changeAmount,    // centavos (troco)
    required FiscalStatus fiscalStatus,
    String? fiscalKey,
    required String createdAt,
    required String updatedAt,
    required int syncVersion,
    List<SaleItem> items,
    List<SalePayment> payments,
  }) = _Sale;
}

enum SaleStatus { open, completed, cancelled, suspended }
enum FiscalStatus { nonFiscal, pending, issued, contingency, cancelled }
```

### CashRegister (Caixa)

```dart
@freezed
class CashRegister with _$CashRegister {
  const factory CashRegister({
    required String id,
    required String terminalId,
    required String openedByUserId,
    String? closedByUserId,
    required CashRegisterStatus status,
    required int openingAmount,           // centavos
    int? closingAmountDeclared,           // centavos
    int? closingAmountCalculated,         // centavos (SUM de cash_movements)
    int? difference,                      // centavos
    required String openedAt,
    String? closedAt,
    required String createdAt,
    required String updatedAt,
  }) = _CashRegister;
}

enum CashRegisterStatus { open, closed }
```

### InventoryItem (Estoque)

```dart
@freezed
class InventoryItem with _$InventoryItem {
  const factory InventoryItem({
    required String id,
    required String productId,
    required double quantity,    // REAL — suporta KG
    required double minQuantity, // alerta de estoque mínimo
    required String updatedAt,
    required int syncVersion,
  }) = _InventoryItem;
}
```

### FeatureFlags (Flags de Licença)

```dart
class FeatureFlags {
  final bool fiscalNfce;       // v1.0: NF-e
  final bool fiscalSat;        // v1.0: SAT
  final bool cloudSync;        // Pro: Supabase
  final bool adminDashboard;   // Pro: painel web
  final bool hardwarePrinter;  // todos (default: true)
  final bool clientCredit;     // todos: crédito/conta corrente
  final int maxTerminals;      // max terminais (default: 1)

  const FeatureFlags.mvp();    // valores padrão MVP
  factory FeatureFlags.fromJson(Map<String, dynamic> json);
}
```

## Enums

| Enum | Valores |
|---|---|
| `UserRole` | `admin`, `operator`, `manager` |
| `ProductUnit` | `un`, `kg`, `l`, `m`, `box`, `pkg` |
| `SaleStatus` | `open`, `completed`, `cancelled`, `suspended` |
| `FiscalStatus` | `nonFiscal`, `pending`, `issued`, `contingency`, `cancelled` |
| `CashRegisterStatus` | `open`, `closed` |
| `PaymentMethod` | `cash`, `pix`, `debitCard`, `creditCard`, `storeCredit`, `other` |
| `PaperWidth` | `mm58`, `mm80` |
| `SyncMode` | `online`, `offline` |

## Interfaces de Repositório

Cada tabela principal tem uma interface de repositório em `domain`:

```
ProductRepository    → CRUD + watch de produtos e categorias
SaleRepository       → Criação, finalização, consulta de vendas
CashRegisterRepository → Abertura, fechamento, movimentos de caixa
InventoryRepository  → Estoque, ajustes, histórico
CustomerRepository   → CRUD de clientes
StoreConfigRepository → Config da loja (single-row)
SyncLogRepository    → Log de sync (usado pelo sync_engine)
UserRepository       → CRUD de usuários
LicenseRepository    → Upsert e leitura de licença ativa
```
