# Geração de Código

O keroo-pdv usa geração de código em três frameworks. Todos os arquivos gerados têm a extensão `.g.dart` ou `.freezed.dart`.

## Quando Re-gerar

| Mudança | Re-gerar? |
|---|---|
| Adicionar coluna/tabela Drift | Sim |
| Adicionar `@riverpod` provider | Sim |
| Adicionar/modificar modelo `@freezed` | Sim |
| Alterar queries de DAO (Drift) | Sim |
| Alterar lógica de negócio (sem annotation) | Não |
| Alterar widgets Flutter | Não |

## Comando

```bash
# Todos os pacotes
melos run build_runner

# Pacote específico (mais rápido durante desenvolvimento)
cd packages/database
dart run build_runner build --delete-conflicting-outputs
```

A flag `--delete-conflicting-outputs` é necessária quando um arquivo `.g.dart` já existe e vai ser regenerado com conteúdo diferente.

## Drift (Banco de Dados)

O Drift gera a partir das classes `Table`:

```dart
// Definição (users.dart)
@DataClassName('User')
class Users extends Table {
  TextColumn get id => text().withDefault(...)();
  TextColumn get name => text()();
  // ...
}

// Gerado (database.g.dart):
// - Classe de dados: User
// - Companion: UsersCompanion (para INSERT/UPDATE)
// - Mixins de DAO
// - Queries type-safe
```

### Nota sobre `@DataClassName`

Usado em todas as tabelas para evitar conflito de nomes com as classes de domínio:

```dart
@DataClassName('LicenseRow')  // evita conflito com domain.License
class Licenses extends Table { ... }

@DataClassName('StoreConfigRow')  // evita conflito com domain.StoreConfig
class StoreConfigs extends Table { ... }
```

### Naming Conflict: `Table.tableName`

A coluna `table_name` em `sync_log` usa `.named()` para evitar conflito com o getter `Table.tableName` do Drift:

```dart
TextColumn get affectedTable => text().named('table_name')();
```

## Riverpod (Gerenciamento de Estado)

O Riverpod 3.x usa `@riverpod` annotation para geração automática de providers:

```dart
// Provider simples (função)
@riverpod
Future<List<Product>> products(Ref ref) async {
  final repo = ref.watch(productRepositoryProvider);
  return repo.getAll();
}
// Gera: productsProvider

// Notifier (classe)
@riverpod
class Cart extends _$Cart {
  @override
  CartState build() => CartState.empty();

  void addItem(Product product) {
    state = state.copyWith(/* ... */);
  }
}
// Gera: cartProvider (NÃO cartNotifierProvider)
```

> O generator **remove o sufixo "Notifier"** do nome da classe para gerar o nome do provider.
> `CartNotifier` → `cartProvider` (não `cartNotifierProvider`)

### Usar `.value` não `.valueOrNull`

Riverpod 3.x não tem `.valueOrNull`. Use `.value` diretamente (retorna null se não há dados):

```dart
// ✓ Correto
final products = ref.watch(productsProvider).value;

// ✗ Errado (não existe em Riverpod 3.x)
final products = ref.watch(productsProvider).valueOrNull;
```

## Freezed (Modelos Imutáveis)

Usado no pacote `domain` para modelos:

```dart
@freezed
class Product with _$Product {
  const factory Product({
    required String id,
    required String name,
    required int salePrice,
  }) = _Product;
}

// Gera:
// - copyWith()
// - == e hashCode
// - toString()
// - _Product implementação
```

## Arquivos Gerados no Repositório

Os arquivos `.g.dart` e `.freezed.dart` **são incluídos no repositório** (não estão no `.gitignore`). Isso garante que:
- CI/CD não precisa rodar `build_runner`
- Desenvolvedores podem buildar sem re-gerar
- O diff dos arquivos gerados fica visível em code review

Ao modificar anotações, sempre commitar os arquivos gerados junto com as mudanças.
