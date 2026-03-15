# Início Rápido

Guia para colocar o keroo-pdv em funcionamento do zero.

## Pré-requisitos

Antes de começar, verifique que seu ambiente tem tudo instalado:

→ [Pré-requisitos detalhados](pre-requisitos.md)

## Instalação

Clone o repositório, instale as dependências e gere o código:

→ [Guia de instalação](instalacao.md)

## Primeiro Uso

Crie o administrador inicial, abra o caixa e registre a primeira venda:

→ [Primeiro uso](primeiro-uso.md)

## Resumo dos Comandos

```bash
# 1. Instalar dependências de todos os pacotes
melos bootstrap

# 2. Gerar código (Drift + Riverpod + Freezed)
melos run build_runner

# 3. Executar o app
cd apps/pdv && flutter run -d windows   # Windows
cd apps/pdv && flutter run -d linux     # Linux

# 4. Rodar todos os testes
melos run test

# 5. Lint
melos run analyze
```
