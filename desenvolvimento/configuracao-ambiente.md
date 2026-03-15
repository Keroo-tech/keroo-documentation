# Configuração do Ambiente de Desenvolvimento

## Pré-requisitos

Ver [inicio-rapido/pre-requisitos.md](../inicio-rapido/pre-requisitos.md) para a lista completa.

Resumo:
- Flutter 3.24+ / Dart 3.5+
- Melos 7+
- Windows 10+ ou Linux (Ubuntu 22.04+)

## Setup Inicial

```bash
# 1. Clone
git clone <url> keroo-pdv && cd keroo-pdv

# 2. Dependências
melos bootstrap

# 3. Geração de código
melos run build_runner

# 4. Verificar
melos run analyze
melos run test
```

## IDE Recomendada

**VS Code** com extensões:
- Dart
- Flutter
- Melos (extensão não oficial — opcional)

**Android Studio / IntelliJ IDEA** com plugins:
- Flutter
- Dart

## Configuração do Banco em Desenvolvimento

Em desenvolvimento, o banco é criado automaticamente no primeiro run. Para resetar:

```bash
# Windows — apagar o banco SQLite
del "%USERPROFILE%\Documents\keroo_pdv.db"

# Linux
rm ~/Documents/keroo_pdv.db
```

Após deletar, o app passa pelo setup inicial novamente.

## Modo de Hardware em Desenvolvimento

Sem hardware físico, o app usa automaticamente:
- `NullPrinter`: não imprime, não falha
- `NullScale`: retorna `null` (campo de peso fica manual)

Configure em **Configurações → Hardware → Tipo: Nulo** se necessário.

## Variáveis de Ambiente / Build Flavors

Não há sistema de flavors ou variáveis de ambiente no MVP. A configuração (hardware, modo de sync) é feita na UI do app.

## Dart Workspace

O projeto usa `resolution: workspace` (Dart 3.5+) em todos os `pubspec.yaml`. Isso significa que a resolução de dependências é unificada — não há lockfiles individuais por pacote.

Se houver conflito de versão entre pacotes:
```bash
dart pub deps  # visualizar árvore de dependências
```

## Executar um Pacote Específico

```bash
# Rodar testes de um pacote específico
cd packages/sync_engine
dart test

# Analyze de um pacote específico
cd packages/database
dart analyze
```

## Hot Reload / Hot Restart

O Flutter Desktop suporta hot reload (`r`) e hot restart (`R`) durante o desenvolvimento. Note que alterações no schema Drift requerem hot restart (ou re-run completo) pois envolvem re-geração de código.
