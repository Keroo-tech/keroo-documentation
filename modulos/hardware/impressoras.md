# Impressoras Térmicas

O keroo-pdv suporta impressoras térmicas compatíveis com o protocolo **ESC/POS**.

## Protocolo ESC/POS

ESC/POS é o padrão da Epson para impressoras térmicas de recibo, amplamente adotado por fabricantes como Elgin, Bematech, Daruma, Epson e outros.

Os comandos ESC/POS são sequências de bytes enviados diretamente à impressora. O `document_module` gera estes bytes, e o `hardware_module` os envia ao dispositivo.

## Implementações Disponíveis

### NullPrinter (Desenvolvimento / Testes)

```dart
class NullPrinter implements PrinterInterface {
  @override
  Future<void> printReceipt(List<int> escPosBytes) async {
    // Não faz nada — apenas descarta os bytes
  }

  @override
  Future<void> openCashDrawer() async {}

  @override
  Future<bool> isAvailable() async => true;
}
```

Usar sempre em testes. **Nunca assuma que hardware está presente em ambiente de teste.**

### WindowsPrinter (Windows)

Usa a API `winspool` do Windows para enviar dados à impressora instalada no sistema. A impressora deve estar instalada como impressora de sistema (Painel de Controle → Impressoras).

Configuração: nome da impressora do sistema (ex: `"TM-T20III"`)

### SerialPrinter (Linux / Serial)

Comunicação direta via porta serial (`/dev/ttyUSB0`, `/dev/ttyS0`, etc.).

Configuração: caminho da porta e baud rate (geralmente 9600 ou 115200 bps).

## Gaveta de Dinheiro

A gaveta é acionada via impressora (comando ESC/POS `ESC p`). Não é um periférico independente no software — é aberta sempre após a impressão do comprovante de venda em dinheiro.

## Largura do Papel

Configurável em **Configurações → Loja → Largura do Papel**:

| Opção | Valor | Chars por linha |
|---|---|---|
| 58 mm | `mm58` | 32 caracteres |
| 80 mm | `mm80` | 48 caracteres |

A largura é armazenada em `store_configs.paper_width` e usada pelo `document_module` para formatar o comprovante.

## Configurar Impressora no App

1. Vá em **Configurações → Hardware**
2. Selecione o tipo de impressora
3. Configure o nome/porta
4. Teste a impressão com o botão "Imprimir Teste"
