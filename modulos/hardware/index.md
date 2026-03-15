# Módulo: hardware_module

O pacote `hardware_module` abstrai a comunicação com periféricos de PDV (impressora térmica, balança, gaveta de dinheiro) usando o padrão **Strategy**.

## Princípio de Design

Toda interação com hardware é feita através de **interfaces**. O app nunca depende de uma implementação específica — apenas do contrato da interface.

Isso permite:
- **Desenvolvimento sem hardware**: usar `NullPrinter` e `NullScale`
- **Troca de hardware**: trocar a implementação sem alterar a UI
- **Testes determinísticos**: `NullPrinter` não falha, não tem timeout

## Interfaces

### PrinterInterface

```dart
abstract class PrinterInterface {
  Future<void> printReceipt(List<int> escPosBytes);
  Future<void> openCashDrawer();
  Future<bool> isAvailable();
}
```

### ScaleInterface

```dart
abstract class ScaleInterface {
  Future<double?> readWeight();  // null se balança indisponível
  Stream<double> weightStream(); // stream contínuo de leituras
}
```

## Implementações

| Implementação | Quando usar |
|---|---|
| `NullPrinter` | Desenvolvimento, testes, quando não há impressora |
| `WindowsPrinter` | Windows com driver `winspool` instalado |
| `SerialPrinter` | Impressora via porta serial (Linux/Windows) |
| `NullScale` | Desenvolvimento, testes, quando não há balança |
| `SerialScale` | Balança Toledo/Filizola via porta serial (`libserialport`) |

## Detecção de Hardware

O módulo tenta detectar o hardware disponível no startup. Se nenhum hardware for encontrado, usa a implementação Null automaticamente.

A configuração pode ser feita manualmente em **Configurações → Hardware**.

## Tolerância de Falhas

- Se a impressora falhar durante a impressão do comprovante, a venda **já está confirmada** — o erro de impressão é reportado ao operador mas não reverte a venda
- Se a balança parar de responder, o campo de peso volta ao modo manual

## Seções Relacionadas

- [Impressoras — ESC/POS e Windows](impressoras.md)
- [Balanças — serial e tolerância](balancas.md)
