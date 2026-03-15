# Módulo: domain

O pacote `domain` é a **camada mais pura** do sistema. Contém os modelos de negócio e as interfaces de repositório — sem nenhuma dependência de Flutter, IO ou banco de dados.

## Responsabilidades

- **Modelos Freezed**: representações imutáveis das entidades de negócio
- **Interfaces de Repositório**: contratos que o `database` implementa
- **Enums**: tipos de dados enumerados usados em todo o sistema
- **Serviços de Domínio**: lógica de negócio pura (sem I/O)

## Dependências

```
freezed_annotation  → geração de código para modelos imutáveis
(nenhuma dependência de pacotes internos)
```

## Por que Dart Puro?

A separação estrita permite:
- **Testabilidade**: testes de domínio sem setup de banco ou Flutter
- **Reuso**: os modelos podem ser usados em qualquer contexto (CLI, testes, futura API web)
- **Clareza**: o contrato de negócio é legível sem overhead de infraestrutura

## Conflito de Nomes

Tanto `domain` quanto `database` têm classes `Product`, `Sale`, `Customer`, etc. Os repositórios em `database` sempre usam alias:

```dart
import 'package:database/src/database.dart';        // Drift: Product, Sale
import 'package:domain/domain.dart' as domain;      // domain.Product, domain.Sale

// Mapper: Drift row → domain model (direção correta)
domain.Product _toDomain(Product row) { ... }
```

## Seções Relacionadas

- [Modelos Freezed](modelos.md)
- [Serviços de Domínio](servicos.md)
