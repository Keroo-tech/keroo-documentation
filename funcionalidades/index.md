# Funcionalidades

Guia de uso das telas e funcionalidades do keroo-pdv.

## Mapa de Navegação

```mermaid
flowchart TD
    SETUP["/setup\nSetup Inicial"]
    LOGIN["/login\nLogin PIN"]
    LIC["/settings/license\nLicença"]

    SETUP --> LOGIN
    LOGIN --> LIC
    LIC --> PDV

    subgraph "App Principal (ShellRoute + Sidebar)"
        PDV["/pos\nPDV - Venda"]
        CAIXA["/cashier\nCaixa"]
        CAIXA_OPEN["/cashier/open\nAbrir Caixa"]
        CAIXA_CLOSE["/cashier/close\nFechar Caixa"]
        PROD["/products\nProdutos"]
        PROD_NEW["/products/new\nNovo Produto"]
        PROD_CAT["/products/categories\nCategorias"]
        INV["/inventory\nEstoque"]
        INV_ADJ["/inventory/adjustment/:id\nAjuste"]
        INV_HIST["/inventory/history/:id\nHistórico"]
        CLI["/customers\nClientes"]
        REL["/reports\nRelatórios"]
        SET["/settings\nConfigurações"]
        SET_STORE["/settings/store-config\nLoja"]
        SET_HW["/settings/hardware\nHardware"]
        SET_USR["/settings/users\nUsuários"]
        SYNC["/sync/status\nSincronização"]
    end

    PDV --> CAIXA_OPEN
    CAIXA --> CAIXA_OPEN
    CAIXA --> CAIXA_CLOSE
    PROD --> PROD_NEW
    PROD --> PROD_CAT
    INV --> INV_ADJ
    INV --> INV_HIST
    SET --> SET_STORE
    SET --> SET_HW
    SET --> SET_USR
```

## Guards de Acesso

| Rota | Guard | Comportamento |
|---|---|---|
| Todas (exceto `/setup`, `/login`) | Autenticação | Redireciona para `/login` se não autenticado |
| Todas (exceto `/settings/license`) | Licença | Redireciona para `/settings/license` se licença inválida/ausente |
| `/settings/fiscal` | Feature flag | Redireciona para `/settings` se `fiscalNfce == false` |
| `/settings/users*` | Role admin | Redireciona para `/settings` se não for admin |

## Seções das Funcionalidades

| Funcionalidade | Link |
|---|---|
| Autenticação e usuários | [autenticacao.md](autenticacao.md) |
| PDV — tela de venda | [pdv.md](pdv.md) |
| Caixa | [caixa.md](caixa.md) |
| Estoque | [estoque.md](estoque.md) |
| Produtos e categorias | [produtos.md](produtos.md) |
| Clientes | [clientes.md](clientes.md) |
| Relatórios | [relatorios.md](relatorios.md) |
| Configurações | [configuracoes.md](configuracoes.md) |
| Sincronização | [sincronizacao.md](sincronizacao.md) |
