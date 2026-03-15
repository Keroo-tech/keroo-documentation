# Módulo: document_module

O pacote `document_module` é responsável pela geração de documentos impressos: **comprovante de venda** e **relatório de fechamento de caixa**.

> Estes documentos **não têm valor fiscal**. São comprovantes de controle interno.

## Responsabilidades

- `ComprovanteBuilder`: gera os bytes ESC/POS do comprovante de venda
- `ClosingReportBuilder`: gera o relatório de fechamento de caixa
- Formatação adapta-se à largura do papel (`mm58` ou `mm80`)

## ComprovanteBuilder

Gera o comprovante de venda com:

- Cabeçalho: nome da loja, CNPJ, endereço, telefone
- Data e hora da venda
- Número do pedido
- Lista de itens: produto, quantidade, valor unitário, subtotal
- Subtotal, desconto (se houver), total
- Formas de pagamento e valores
- Troco (se pagamento em dinheiro)
- Rodapé: mensagem de agradecimento

```dart
final bytes = await ComprovanteBuilder(
  sale: sale,
  storeConfig: config,
  paperWidth: PaperWidth.mm80,
).build();

await printer.printReceipt(bytes);
```

## ClosingReportBuilder

Gera o relatório de fechamento de caixa com:

- Período (abertura até fechamento)
- Operador responsável
- Saldo de abertura
- Resumo por forma de pagamento
- Total de vendas
- Sangrias e suprimentos
- Saldo calculado vs declarado
- Diferença

## PaperWidth e Formatação

A extensão `PaperWidthEscPos` converte o enum de largura para o número de caracteres por linha:

| `PaperWidth` | Chars por linha |
|---|---|
| `mm58` | 32 |
| `mm80` | 48 |

O builder ajusta automaticamente o alinhamento, quebras de linha e colunas de acordo com a largura configurada.

## Dependências

```
hardware_module → PrinterInterface (para envio dos bytes)
domain         → Sale, CashRegister, StoreConfig, PaperWidth
```

## Campos Fiscais

O comprovante **não** imprime campos fiscais (NCM, CFOP, CST) no MVP. Estes campos só aparecem no v1.0 quando `featureFlags.fiscalNfce == true` e o documento gerado será um DANFE NFC-e pelo módulo fiscal.
