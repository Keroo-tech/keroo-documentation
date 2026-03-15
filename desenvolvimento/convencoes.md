# Convenções de Código

Padrões obrigatórios para o desenvolvimento no keroo-pdv.

## Idioma do Código

**Todo código deve ser em inglês**: nomes de classes, funções, variáveis, comentários técnicos.

```dart
// ✓ Correto
class SaleService {
  Future<void> finalizeSale(Sale sale, PaymentMethod payment) { ... }
}

// ✗ Errado
class ServicoVenda {
  Future<void> finalizarVenda(Venda venda, FormaPagamento pagamento) { ... }
}
```

**Exceção**: strings exibidas na UI são em português brasileiro.

## Proibições

| Proibição | Alternativa |
|---|---|
| `double` para valores monetários | `int` em centavos |
| `INTEGER AUTO_INCREMENT` como PK | `TEXT` UUID |
| `DELETE` em `cash_movements` / `inventory_movements` | Append-only; registre correção como novo movimento |
| `UPDATE` em `cash_movements` / `inventory_movements` | Idem |
| `DELETE` de qualquer registro de domínio | Soft delete com `deleted_at` |
| `ALTER TABLE` sem criar migração | Criar migração versionada |
| `packages/fiscal_module/` (não existe ainda) | Módulo fiscal criado apenas no Sprint 11 (v1.0) |
| "Nota Fiscal", "NF-e", "NFC-e", "SAT", "DANFE" na UI | Termos neutros (recibo, comprovante) |
| `Navigator.push()` / `Navigator.pushNamed()` | `context.go()` / `context.push()` (go_router) |
| `valueOrNull` no Riverpod | `.value` (Riverpod 3.x) |

## Padrão Riverpod

**Sempre use code generation** (`@riverpod`), nunca constructors manuais:

```dart
// ✓ Correto
@riverpod
Future<List<Product>> products(Ref ref) async {
  return ref.watch(productRepositoryProvider).watchAll().first;
}

@riverpod
class Cart extends _$Cart {
  @override
  CartState build() => CartState.empty();
}
// Gera: cartProvider (remove "Notifier" do nome)

// ✗ Errado
final productsProvider = FutureProvider<List<Product>>((ref) async { ... });
```

## Padrão de Repositório

Interfaces em `domain`, implementações em `database`:

```dart
// domain — interface
abstract class ProductRepository {
  Stream<List<Product>> watchAll();
  Future<void> upsert(Product product);
}

// database — implementação
class ProductRepositoryImpl implements ProductRepository {
  final ProductsDao _dao;
  ProductRepositoryImpl(this._dao);

  @override
  Stream<List<Product>> watchAll() =>
      _dao.watchActiveProducts().map((rows) => rows.map(_toDomain).toList());

  domain.Product _toDomain(Product row) => domain.Product(
    id: row.id,
    name: row.name,
    salePrice: row.salePrice,
    // ...
  );
}
```

## Nomes de Providers (Riverpod)

O generator remove o sufixo "Notifier":

| Classe | Provider gerado |
|---|---|
| `Cart extends _$Cart` | `cartProvider` |
| `ActiveLicenseNotifier` | `activeLicenseProvider` |
| `SessionNotifier` | `sessionProvider` |

## Guard de Feature Flag

Antes de qualquer lógica fiscal:

```dart
// ✓ Sempre verificar antes de renderizar ou processar campos fiscais
final flags = ref.watch(featureFlagsProvider);
if (flags.fiscalNfce) {
  // lógica NF-e
}
```

## Convenções de Rota

```dart
// ✓ Sempre usar constantes de Routes
context.go(Routes.pos);
context.push(Routes.productDetail(product.id));

// ✗ Nunca usar strings literais
context.go('/pos');
context.push('/products/${product.id}');
```

## Formatação e Lint

```bash
# Formatar código
dart format .

# Analisar (sem erros permitidos)
melos run analyze
```

Todo PR deve ter `melos run analyze` sem erros antes do merge.

## Segurança no Código

- PINs: sempre usar bcrypt, nunca plain text
- CNPJ e email do cliente: nunca logar
- Queries SQL: sempre usar parâmetros do Drift (sem interpolação de string)
- Comunicação Supabase: somente HTTPS
- API interna (`:7890`): apenas LAN, bind em `anyIPv4`

## Documentação no Código

- Não adicionar docstrings desnecessárias a código óbvio
- `CLAUDE.md` de cada diretório deve ser atualizado na mesma resposta que a mudança de código
- Novo pacote = `README.md` + `CLAUDE.md` criados imediatamente
