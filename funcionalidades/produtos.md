# Produtos e Categorias

O módulo de produtos gerencia o catálogo de itens disponíveis para venda.

## Listagem de Produtos

**Rota**: `/products`

Exibe todos os produtos ativos com:
- Código de barras / código interno
- Nome
- Preço de venda
- Categoria
- Status (ativo/inativo)

## Cadastrar Produto

**Rota**: `/products/new`

Campos obrigatórios:
- **Nome**: nome do produto (máx. recomendado: 40 chars para impressão)
- **Preço de venda**: em reais (armazenado em centavos)
- **Unidade**: UN, KG, L, M, BOX, PKG

Campos opcionais:
- **Código de barras**: EAN-13 ou qualquer código único
- **Código interno**: código próprio da loja
- **Preço de custo**: para cálculo de margem
- **Categoria**: vincula a uma categoria
- **Pesável**: ativa leitura automática da balança no PDV

Campos fiscais (ocultos no MVP, ativados na v1.0):
- NCM, CFOP, CST/CSOSN, Alíquota ICMS

## Editar Produto

**Rota**: `/products/:id`

Todos os campos do cadastro podem ser editados. A edição gera um `SyncLogEntry` para propagação via sync.

## Desativar Produto

Produtos não são deletados — são desativados (`active = false`). Produtos inativos:
- Não aparecem no PDV para busca
- Não aparecem na listagem de estoque
- Mantêm o histórico de vendas e movimentos

## Categorias

**Rota**: `/products/categories`

Categorias organizam os produtos para facilitar a busca no PDV.

### Criar Categoria

**Rota**: `/products/categories/new`

- **Nome**: obrigatório
- **Descrição**: opcional
- **Cor**: código hexadecimal (opcional, para exibição visual)

### Editar / Desativar Categoria

**Rota**: `/products/categories/:categoryId`

Categorias desativadas não aparecem no cadastro de novos produtos, mas os produtos vinculados mantêm a referência.

## Tabelas Relacionadas

### `products`

```
id, barcode, internal_code, name, unit, cost_price (cents),
sale_price (cents), category_id, ncm [fiscal], cfop [fiscal],
cst_csosn [fiscal], icms_rate [fiscal], active, weighable,
created_at, updated_at, deleted_at, sync_version
```

### `categories`

```
id, name, description, color, active,
created_at, updated_at, deleted_at, sync_version
```

## Sincronização

Produtos e categorias usam **Last-Write-Wins** na sincronização: a versão com maior `sync_version` prevalece.
