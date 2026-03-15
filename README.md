# keroo-documentation

Documentação técnica do **keroo-pdv** — sistema de Ponto de Venda (PDV) desktop offline-first para pequenos negócios brasileiros.

Construída com [Docsify](https://docsify.js.org/), renderizada diretamente no navegador a partir de arquivos Markdown.

## Sobre o keroo-pdv

O keroo-pdv roda em **Windows** e **Linux**, construído com Flutter. A filosofia do produto é **"Gestão Primeiro, Fiscal Depois"**: o MVP valida velocidade de vendas e controle de estoque/caixa sem integrações fiscais — o módulo NF-e/SAT é ativado na v1.0 via flag de licença.

## Como visualizar a documentação

**Opção 1 — servidor local com `docsify-cli`:**

```bash
npm i -g docsify-cli
docsify serve .
```

Acesse `http://localhost:3000`.

**Opção 2 — qualquer servidor HTTP estático:**

```bash
# Python
python -m http.server 3000

# Node
npx serve .
```

> Não abra `index.html` diretamente via `file://` — o Docsify requer um servidor HTTP para carregar os arquivos Markdown.

## Estrutura

```
.
├── index.html              # Entrada do Docsify
├── index.md                # Página inicial da documentação
├── _sidebar.md             # Navegação lateral
├── roadmap.md              # MVP vs v1.0
├── inicio-rapido/          # Instalação e primeiro uso
├── arquitetura/            # Visão técnica do sistema
├── modulos/                # Documentação de cada pacote Flutter
├── funcionalidades/        # Guia de uso das telas
├── banco-de-dados/         # Schema SQLite completo
├── api/                    # Endpoints HTTP de sincronização
└── desenvolvimento/        # Ambiente, testes, convenções
```

## Seções

| Seção | Público | Descrição |
|---|---|---|
| Início Rápido | Todos | Instalação e primeiro uso |
| Arquitetura | Devs | Visão geral técnica do monorepo Flutter |
| Módulos | Devs | Documentação de cada pacote |
| Funcionalidades | Todos | Guia de uso das telas |
| Banco de Dados | Devs | Schema SQLite (Drift) |
| API de Sync | Devs | Endpoints HTTP internos (porta 7890) |
| Desenvolvimento | Devs | Ambiente, testes, convenções de código |
| Roadmap | Todos | MVP (sprints 1–10) vs v1.0 (sprints 11–16) |
