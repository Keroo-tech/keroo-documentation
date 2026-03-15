# Referência das Tabelas

Documentação completa das **15 tabelas** do banco de dados keroo-pdv (schema v5).

## 1. `users` — Usuários / Operadores

| Coluna | Tipo | Constraint | Descrição |
|---|---|---|---|
| `id` | TEXT | PK, DEFAULT uuid | UUID gerado no cliente |
| `name` | TEXT | NOT NULL | Nome do operador |
| `email` | TEXT | UNIQUE, NULL | Email (opcional, sensível) |
| `pin_hash` | TEXT | NOT NULL | Hash bcrypt do PIN de 3 dígitos |
| `operator_code` | TEXT | DEFAULT '000' | Código sequencial de login (ex: "001") |
| `role` | TEXT | NOT NULL, CHECK | `'admin'`, `'operator'`, `'manager'` |
| `active` | INTEGER | DEFAULT 1 | 0 = desativado |
| `created_at` | TEXT | NOT NULL, DEFAULT now | ISO-8601 UTC |
| `updated_at` | TEXT | NOT NULL, DEFAULT now | ISO-8601 UTC |
| `deleted_at` | TEXT | NULL | Soft delete |

---

## 2. `categories` — Categorias de Produto

| Coluna | Tipo | Constraint | Descrição |
|---|---|---|---|
| `id` | TEXT | PK, DEFAULT uuid | UUID |
| `name` | TEXT | NOT NULL | Nome da categoria |
| `description` | TEXT | NULL | Descrição |
| `color` | TEXT | NULL | Hex color (ex: `#FF5733`) |
| `active` | INTEGER | DEFAULT 1 | |
| `created_at` | TEXT | NOT NULL | ISO-8601 UTC |
| `updated_at` | TEXT | NOT NULL | ISO-8601 UTC |
| `deleted_at` | TEXT | NULL | Soft delete |
| `sync_version` | INTEGER | DEFAULT 0 | Controle de conflito LWW |

---

## 3. `customers` — Clientes

| Coluna | Tipo | Constraint | Descrição |
|---|---|---|---|
| `id` | TEXT | PK, DEFAULT uuid | UUID |
| `name` | TEXT | NOT NULL | Nome do cliente |
| `cpf` | TEXT | UNIQUE, NULL | CPF (11 dígitos, sem formatação) |
| `phone` | TEXT | NULL | Telefone |
| `email` | TEXT | NULL | Email (sensível — não logar) |
| `address` | TEXT | NULL | Endereço |
| `credit_limit` | INTEGER | DEFAULT 0 | Limite de crédito em centavos |
| `notes` | TEXT | NULL | Observações |
| `active` | INTEGER | DEFAULT 1 | |
| `created_at` | TEXT | NOT NULL | ISO-8601 UTC |
| `updated_at` | TEXT | NOT NULL | ISO-8601 UTC |
| `deleted_at` | TEXT | NULL | Soft delete |
| `sync_version` | INTEGER | DEFAULT 0 | |

---

## 4. `products` — Produtos

| Coluna | Tipo | Constraint | Descrição |
|---|---|---|---|
| `id` | TEXT | PK, DEFAULT uuid | UUID |
| `barcode` | TEXT | UNIQUE, NULL | Código de barras EAN ou personalizado |
| `internal_code` | TEXT | UNIQUE, NULL | Código interno da loja |
| `name` | TEXT | NOT NULL | Nome do produto |
| `unit` | TEXT | DEFAULT 'UN', CHECK | `'UN'`, `'KG'`, `'L'`, `'M'`, `'BOX'`, `'PKG'` |
| `cost_price` | INTEGER | DEFAULT 0 | Preço de custo em centavos |
| `sale_price` | INTEGER | NOT NULL | Preço de venda em centavos |
| `category_id` | TEXT | FK categories.id, NULL | |
| `ncm` | TEXT | NULL | **[FISCAL]** NCM para NF-e v1.0 |
| `cfop` | TEXT | NULL | **[FISCAL]** Código de Operação Fiscal |
| `cst_csosn` | TEXT | NULL | **[FISCAL]** Código de Situação Tributária |
| `icms_rate` | INTEGER | NULL | **[FISCAL]** Alíquota ICMS (centésimos de %) |
| `active` | INTEGER | DEFAULT 1 | |
| `weighable` | INTEGER | DEFAULT 0 | 1 = ativa leitura de balança no PDV |
| `created_at` | TEXT | NOT NULL | ISO-8601 UTC |
| `updated_at` | TEXT | NOT NULL | ISO-8601 UTC |
| `deleted_at` | TEXT | NULL | Soft delete |
| `sync_version` | INTEGER | DEFAULT 0 | |

---

## 5. `inventory` — Estoque Atual

| Coluna | Tipo | Constraint | Descrição |
|---|---|---|---|
| `id` | TEXT | PK, DEFAULT uuid | UUID |
| `product_id` | TEXT | NOT NULL, FK products.id | Um registro por produto |
| `quantity` | REAL | DEFAULT 0 | Quantidade atual (suporta KG) |
| `min_quantity` | REAL | DEFAULT 0 | Quantidade mínima para alerta |
| `updated_at` | TEXT | NOT NULL | ISO-8601 UTC |
| `sync_version` | INTEGER | DEFAULT 0 | Usado no delta aditivo |

---

## 6. `inventory_movements` — Histórico de Estoque (**Append-Only**)

| Coluna | Tipo | Constraint | Descrição |
|---|---|---|---|
| `id` | TEXT | PK, DEFAULT uuid | UUID |
| `product_id` | TEXT | NOT NULL, FK products.id | |
| `type` | TEXT | NOT NULL, CHECK | `'in'`, `'out'`, `'adjustment'`, `'return'` |
| `quantity` | REAL | NOT NULL | Quantidade movida |
| `previous_quantity` | REAL | NOT NULL | Quantidade antes do movimento |
| `new_quantity` | REAL | NOT NULL | Quantidade após o movimento |
| `reference_id` | TEXT | NULL | UUID da venda ou ajuste de origem |
| `reference_type` | TEXT | NULL | `'sale'`, `'adjustment'`, `'return'` |
| `user_id` | TEXT | NOT NULL, FK users.id | Operador responsável |
| `terminal_id` | TEXT | NOT NULL | Identificador do terminal |
| `created_at` | TEXT | NOT NULL | ISO-8601 UTC |
| `notes` | TEXT | NULL | Observação do ajuste |

> **Append-only**: triggers SQLite bloqueiam UPDATE e DELETE.

---

## 7. `cash_registers` — Caixas

| Coluna | Tipo | Constraint | Descrição |
|---|---|---|---|
| `id` | TEXT | PK, DEFAULT uuid | UUID |
| `terminal_id` | TEXT | NOT NULL | Terminal onde o caixa foi aberto |
| `opened_by_user_id` | TEXT | NOT NULL, FK users.id | Operador que abriu |
| `closed_by_user_id` | TEXT | NULL, FK users.id | Operador que fechou |
| `status` | TEXT | DEFAULT 'open', CHECK | `'open'`, `'closed'` |
| `opening_amount` | INTEGER | DEFAULT 0 | Fundo de abertura em centavos |
| `closing_amount_declared` | INTEGER | NULL | Valor contado pelo operador |
| `closing_amount_calculated` | INTEGER | NULL | Valor calculado pelo sistema |
| `difference` | INTEGER | NULL | Diferença (declared - calculated) |
| `opened_at` | TEXT | NOT NULL | Timestamp de abertura |
| `closed_at` | TEXT | NULL | Timestamp de fechamento |
| `created_at` | TEXT | NOT NULL | ISO-8601 UTC |
| `updated_at` | TEXT | NOT NULL | ISO-8601 UTC |

---

## 8. `cash_movements` — Movimentos de Caixa (**Append-Only**)

| Coluna | Tipo | Constraint | Descrição |
|---|---|---|---|
| `id` | TEXT | PK, DEFAULT uuid | UUID |
| `cash_register_id` | TEXT | NOT NULL, FK cash_registers.id | |
| `type` | TEXT | NOT NULL, CHECK | `'opening'`, `'sale'`, `'return'`, `'withdrawal'`, `'replenishment'`, `'discount'`, `'closing'` |
| `amount` | INTEGER | NOT NULL | Valor em centavos (negativo = saída) |
| `payment_method` | TEXT | NULL, CHECK | `'cash'`, `'pix'`, `'debit_card'`, `'credit_card'`, `'store_credit'`, `'other'` |
| `reference_id` | TEXT | NULL | UUID da venda relacionada |
| `user_id` | TEXT | NOT NULL, FK users.id | Operador |
| `notes` | TEXT | NULL | Observação |
| `created_at` | TEXT | NOT NULL | ISO-8601 UTC |

> **Append-only**: triggers SQLite bloqueiam UPDATE e DELETE.
> O saldo é sempre calculado: `SELECT SUM(amount) FROM cash_movements WHERE cash_register_id = :id`

---

## 9. `sales` — Vendas

| Coluna | Tipo | Constraint | Descrição |
|---|---|---|---|
| `id` | TEXT | PK, DEFAULT uuid | UUID |
| `number` | INTEGER | NOT NULL | Número sequencial da venda |
| `cash_register_id` | TEXT | NOT NULL, FK cash_registers.id | |
| `terminal_id` | TEXT | NOT NULL | Terminal de origem |
| `user_id` | TEXT | NOT NULL, FK users.id | Operador |
| `customer_id` | TEXT | NULL, FK customers.id | Cliente (opcional) |
| `status` | TEXT | DEFAULT 'open', CHECK | `'open'`, `'completed'`, `'cancelled'`, `'suspended'` |
| `subtotal` | INTEGER | DEFAULT 0 | Em centavos |
| `discount` | INTEGER | DEFAULT 0 | Em centavos |
| `total` | INTEGER | DEFAULT 0 | Em centavos |
| `amount_paid` | INTEGER | DEFAULT 0 | Em centavos |
| `change_amount` | INTEGER | DEFAULT 0 | Troco em centavos |
| `fiscal_status` | TEXT | DEFAULT 'non_fiscal' | `'non_fiscal'`, `'pending'`, `'issued'`, `'contingency'`, `'cancelled'` |
| `fiscal_key` | TEXT | UNIQUE, NULL | Chave de acesso NF-e (v1.0) |
| `created_at` | TEXT | NOT NULL | ISO-8601 UTC |
| `updated_at` | TEXT | NOT NULL | ISO-8601 UTC |
| `sync_version` | INTEGER | DEFAULT 0 | |

---

## 10. `sale_items` — Itens da Venda

| Coluna | Tipo | Constraint | Descrição |
|---|---|---|---|
| `id` | TEXT | PK, DEFAULT uuid | UUID |
| `sale_id` | TEXT | NOT NULL, FK sales.id (CASCADE) | |
| `product_id` | TEXT | NOT NULL, FK products.id | |
| `quantity` | REAL | NOT NULL | Quantidade (suporta KG) |
| `unit_price` | INTEGER | NOT NULL | Preço unitário no momento da venda (centavos) |
| `cost_price` | INTEGER | DEFAULT 0 | Custo no momento da venda (centavos) |
| `discount` | INTEGER | DEFAULT 0 | Desconto por item (centavos) |
| `total` | INTEGER | NOT NULL | Total do item (centavos) |
| `ncm` | TEXT | NULL | **[FISCAL]** Snapshot do NCM |
| `cfop` | TEXT | NULL | **[FISCAL]** Snapshot do CFOP |
| `cst_csosn` | TEXT | NULL | **[FISCAL]** Snapshot do CST |
| `created_at` | TEXT | NOT NULL | ISO-8601 UTC |

---

## 11. `sale_payments` — Pagamentos da Venda

| Coluna | Tipo | Constraint | Descrição |
|---|---|---|---|
| `id` | TEXT | PK, DEFAULT uuid | UUID |
| `sale_id` | TEXT | NOT NULL, FK sales.id (CASCADE) | |
| `method` | TEXT | NOT NULL, CHECK | `'cash'`, `'pix'`, `'debit_card'`, `'credit_card'`, `'store_credit'`, `'other'` |
| `amount` | INTEGER | NOT NULL | Valor pago (centavos) |
| `created_at` | TEXT | NOT NULL | ISO-8601 UTC |

---

## 12. `fiscal_queue` — Fila Fiscal (**v1.0**)

> Não processada no MVP. Schema presente para ativação via feature flag na v1.0.

| Coluna | Tipo | Constraint | Descrição |
|---|---|---|---|
| `id` | TEXT | PK, DEFAULT uuid | UUID |
| `sale_id` | TEXT | NOT NULL, FK sales.id | |
| `type` | TEXT | NOT NULL, CHECK | `'nfce'`, `'sat'`, `'nfce_cancellation'`, `'sat_cancellation'` |
| `status` | TEXT | DEFAULT 'pending', CHECK | `'pending'`, `'processing'`, `'issued'`, `'contingency'`, `'permanent_error'`, `'cancelled'` |
| `attempts` | INTEGER | DEFAULT 0 | Tentativas de emissão |
| `max_attempts` | INTEGER | DEFAULT 5 | Máximo de tentativas |
| `next_retry_at` | TEXT | NULL | Próxima tentativa |
| `xml_request` | TEXT | NULL | XML enviado à SEFAZ (sensível) |
| `xml_response` | TEXT | NULL | XML de resposta da SEFAZ (sensível) |
| `access_key` | TEXT | NULL | Chave de acesso (44 dígitos) |
| `protocol` | TEXT | NULL | Número do protocolo SEFAZ |
| `error_code` | TEXT | NULL | Código de erro |
| `error_message` | TEXT | NULL | Mensagem de erro |
| `environment` | TEXT | DEFAULT 'production', CHECK | `'testing'`, `'production'` |
| `created_at` | TEXT | NOT NULL | ISO-8601 UTC |
| `updated_at` | TEXT | NOT NULL | ISO-8601 UTC |

---

## 13. `sync_log` — Log de Sincronização

| Coluna | Tipo | Constraint | Descrição |
|---|---|---|---|
| `id` | TEXT | PK, DEFAULT uuid | UUID |
| `terminal_id` | TEXT | NOT NULL | Terminal de origem |
| `table_name` | TEXT | NOT NULL | Nome da tabela afetada |
| `record_id` | TEXT | NOT NULL | UUID do registro |
| `operation` | TEXT | NOT NULL, CHECK | `'insert'`, `'update'`, `'delete'` |
| `payload` | TEXT | NOT NULL | JSON com os dados modificados |
| `sync_version` | INTEGER | NOT NULL | Versão monotônica |
| `synced` | INTEGER | DEFAULT 0 | 0 = pendente, 1 = sincronizado |
| `created_at` | TEXT | NOT NULL | ISO-8601 UTC |

> Coluna `table_name` usa `.named('table_name')` no Drift para evitar conflito com `Table.tableName`.

---

## 14. `licenses` — Licenças

| Coluna | Tipo | Constraint | Descrição |
|---|---|---|---|
| `id` | TEXT | PK, DEFAULT uuid | UUID |
| `key` | TEXT | UNIQUE, NOT NULL | Chave RSA (base64url.base64url) |
| `owner_cnpj` | TEXT | NOT NULL | CNPJ do titular (sensível — não logar) |
| `company_name` | TEXT | NOT NULL | Razão social |
| `plan` | TEXT | CHECK | `'basic'`, `'pro'`, `'enterprise'` |
| `started_at` | TEXT | NOT NULL | Início da vigência |
| `expires_at` | TEXT | NULL | Expiração (NULL = perpétua) |
| `max_terminals` | INTEGER | DEFAULT 1 | Máximo de terminais |
| `additional_terminals` | INTEGER | DEFAULT 0 | Terminais adicionais comprados |
| `features` | TEXT | DEFAULT '{}' | JSON das feature flags |
| `signature` | TEXT | NOT NULL | Assinatura RSA-SHA256 |
| `last_validated_at` | TEXT | NULL | Última validação local |
| `active` | INTEGER | DEFAULT 1 | |
| `created_at` | TEXT | NOT NULL | ISO-8601 UTC |
| `updated_at` | TEXT | NOT NULL | ISO-8601 UTC |

> `@DataClassName('LicenseRow')` para evitar conflito com `domain.License`

---

## 15. `store_configs` — Configurações da Loja

| Coluna | Tipo | Constraint | Descrição |
|---|---|---|---|
| `id` | TEXT | PK, DEFAULT uuid | UUID |
| `company_name` | TEXT | NOT NULL | Nome do estabelecimento |
| `cnpj` | TEXT | NOT NULL | CNPJ (somente dígitos) |
| `address` | TEXT | NOT NULL | Endereço completo |
| `phone` | TEXT | NULL | Telefone |
| `paper_width` | TEXT | DEFAULT 'mm80' | `'mm58'` ou `'mm80'` |
| `updated_at` | TEXT | NOT NULL | ISO-8601 UTC |

> Tabela single-row: sempre um único registro por banco de dados.
> `@DataClassName('StoreConfigRow')` para evitar conflito com `domain.StoreConfig`
