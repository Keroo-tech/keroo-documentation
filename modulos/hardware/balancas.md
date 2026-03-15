# Balanças

O keroo-pdv suporta balanças seriais para venda de produtos pesáveis (frutas, carnes, frios, etc.).

## Interface

```dart
abstract class ScaleInterface {
  /// Lê o peso atual. Retorna null se a balança estiver indisponível.
  Future<double?> readWeight();

  /// Stream contínuo de leituras (para exibição em tempo real no PDV).
  Stream<double> weightStream();
}
```

## Implementações

### NullScale (Desenvolvimento / Testes)

```dart
class NullScale implements ScaleInterface {
  @override
  Future<double?> readWeight() async => null;

  @override
  Stream<double> weightStream() => const Stream.empty();
}
```

Usar sempre em testes. Retorna `null` para indicar que nenhuma leitura foi obtida.

### SerialScale (Produção)

Comunicação via porta serial usando `libserialport`.

Suporta:
- **Toledo Prix 3** (protocolo Toledo)
- **Filizola MF** (protocolo Filizola)
- Balanças genéricas que emitem peso via serial no formato `\r\n<peso>\r\n`

Configuração: porta serial (ex: `COM3` no Windows, `/dev/ttyUSB0` no Linux) e baud rate.

## Tolerância a Falhas

- Se a balança não responder em 2 segundos: retorna `null`
- O PDV exibe campo de peso manual quando `readWeight()` retorna `null`
- O operador pode digitar o peso manualmente a qualquer momento
- Falha na balança nunca impede uma venda

## Produtos Pesáveis

Produtos com `weighable = true` na tabela `products` ativam a leitura automática da balança no PDV quando o produto é adicionado ao carrinho. A quantidade (em KG) é preenchida automaticamente ou pode ser ajustada manualmente.

## Configurar Balança no App

1. Vá em **Configurações → Hardware**
2. Selecione o tipo de balança e porta serial
3. Teste a leitura com o botão "Testar Balança"

## Implementações Futuras

- Balança USB (HID) — planejada para v1.1
- Balança com display integrado — planejada para v1.1
