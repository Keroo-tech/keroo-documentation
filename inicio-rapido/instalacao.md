# Instalação

## 1. Clonar o Repositório

```bash
git clone <url-do-repositorio> keroo-pdv
cd keroo-pdv
```

## 2. Instalar Dependências

O Melos gerencia todas as dependências do monorepo de uma vez:

```bash
melos bootstrap
```

Isso executa `flutter pub get` em todos os pacotes e no app, resolvendo as dependências do workspace. Aguarde a conclusão — pode levar alguns minutos na primeira execução.

## 3. Gerar Código

O projeto usa geração de código para três frameworks:

- **Drift**: gera as classes de dados e queries do banco SQLite
- **Riverpod**: gera os providers com `@riverpod`
- **Freezed**: gera os modelos imutáveis (no pacote `domain`)

```bash
melos run build_runner
```

Este comando executa `dart run build_runner build --delete-conflicting-outputs` em todos os pacotes que o necessitam. Os arquivos `.g.dart` e `.freezed.dart` são gerados automaticamente.

> **Atenção:** Os arquivos gerados (`.g.dart`, `.freezed.dart`) são incluídos no repositório. Só é necessário re-gerar ao modificar tabelas Drift, adicionar providers `@riverpod` ou alterar modelos `@freezed`.

## 4. Executar o App

```bash
# Windows
cd apps/pdv
flutter run -d windows

# Linux
cd apps/pdv
flutter run -d linux
```

Para listar os dispositivos disponíveis:
```bash
flutter devices
```

## 5. Verificar a Instalação

```bash
# Rodar todos os testes
melos run test

# Lint e análise estática
melos run analyze
```

Todos os testes devem passar e não deve haver erros de análise.

## Troubleshooting

### `melos bootstrap` falha com erro de resolução
Verifique se o `pubspec.yaml` raiz tem `resolution: workspace` e que a versão do Dart é ≥ 3.5:
```bash
dart --version
```

### Erros de geração de código (build_runner)
Delete os arquivos gerados e re-execute:
```bash
find . -name "*.g.dart" -delete
find . -name "*.freezed.dart" -delete
melos run build_runner
```

### Flutter não encontra o target de desktop
```bash
flutter config --enable-windows-desktop  # Windows
flutter config --enable-linux-desktop    # Linux
flutter doctor
```
