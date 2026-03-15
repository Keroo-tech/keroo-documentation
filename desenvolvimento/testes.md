# Testes

Estratégias e padrões de teste para o keroo-pdv.

## Executar Testes

```bash
# Todos os pacotes
melos run test

# Pacote específico
cd packages/sync_engine
dart test

# Com cobertura
cd packages/sync_engine
dart test --coverage=coverage/
```

## Princípio Fundamental: Sem Hardware Real

**Nunca assuma que hardware está presente em testes.** Use sempre as implementações nulas:

```dart
// ✓ Correto
final printer = NullPrinter();
final scale = NullScale();

// ✗ Errado (falha em CI e em máquinas sem hardware)
final printer = WindowsPrinter(deviceName: 'Epson TM-T20');
```

## Testando o Domain

Testes de serviços de domínio são Dart puro — sem setup especial:

```dart
test('calcular troco corretamente', () {
  final service = SaleService();
  expect(service.calculateChange(total: 1000, amountPaid: 2000), 1000);
});
```

## Testando o Banco de Dados (Drift)

Use `NativeDatabase.memory()` para criar um banco em memória nos testes:

```dart
late AppDatabase db;

setUp(() {
  db = AppDatabase(NativeDatabase.memory());
});

tearDown(() async {
  await db.close();
});

test('inserir produto', () async {
  final dao = db.productsDao;
  await dao.upsertProduct(ProductsCompanion.insert(
    name: const Value('Café'),
    salePrice: const Value(1050),
  ));
  final products = await dao.getAll();
  expect(products.length, 1);
  expect(products.first.name, 'Café');
});
```

## Testando o Sync Engine

O `sync_engine` tem testes unitários para:

- `ConflictResolver` — resolução de conflitos LWW e delta aditivo
- `SyncClient` — transições de modo (online/offline), parsing de JSON

```dart
// Exemplo: conflict_resolver_test.dart
test('delta aditivo: dois terminais vendem simultaneamente', () {
  final resolver = ConflictResolver();
  final result = resolver.resolveInventoryQuantity(
    masterCurrentQty: 10.0,
    clientInitialQty: 10.0,
    clientFinalQty: 8.0,  // vendeu 2
  );
  expect(result, 8.0);
});
```

## Testando Providers Riverpod

Use `ProviderContainer` com overrides:

```dart
test('cartProvider começa vazio', () {
  final container = ProviderContainer(overrides: [
    saleRepositoryProvider.overrideWith((_) => FakeSaleRepository()),
  ]);
  addTearDown(container.dispose);

  final cart = container.read(cartProvider);
  expect(cart.items, isEmpty);
  expect(cart.total, 0);
});
```

## Cobertura de Testes por Módulo

| Módulo | Foco dos testes |
|---|---|
| `domain` | Serviços, cálculos de domínio |
| `database` | DAOs com banco in-memory, migrações |
| `sync_engine` | `ConflictResolver`, transições de modo, protocolo JSON |
| `license_module` | Validação RSA, parsing de payload, feature flags |
| `hardware_module` | `NullPrinter`/`NullScale` (smoke test de interface) |
| `document_module` | Geração de bytes ESC/POS (verificação de estrutura) |

## Mocks e Fakes

Prefira **Fakes** (implementações simples) a **Mocks** (biblioteças de mocking) quando possível:

```dart
// Fake — mais simples, sem magia
class FakeSaleRepository implements SaleRepository {
  final _sales = <Sale>[];

  @override
  Future<void> finalizeSale(Sale sale, PaymentMethod payment) async {
    _sales.add(sale.copyWith(status: SaleStatus.completed));
  }

  @override
  Stream<List<Sale>> watchAll() => Stream.value(_sales);
}
```

## O que NÃO Fazer em Testes

- ❌ Usar hardware real (`WindowsPrinter`, `SerialScale`)
- ❌ Banco de dados em arquivo (usar `NativeDatabase.memory()`)
- ❌ Chamadas de rede reais no sync_engine (mockar `HttpClient`)
- ❌ Testes que dependem do horário atual sem injetar o clock
