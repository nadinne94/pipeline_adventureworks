# Gold Layer – Camada Analítica (Business-Ready)
A **Gold Layer** representa a camada final do pipeline de dados, projetada para **consumo analítico**, **dashboards** e **análises de negócio**.
Os dados nesta camada seguem o **modelo dimensional (Star Schema)**, garantindo:
* Alta performance para consultas analíticas
* Clareza semântica para usuários de negócio
* Separação clara entre métricas (fatos) e contextos (dimensões)

Nesta camada, os dados já passaram por:
* Ingestão e persistência (Bronze)
* Limpeza, padronização e enriquecimento (Silver)

## Modelo Dimensional (Star Schema)
### Dimensões
As dimensões fornecem **contexto descritivo** para análise das métricas de vendas.
* `dim_date`
* `dim_customer`
* `dim_product`
* `dim_address`

### Tabelas Fato
As tabelas fato armazenam **métricas quantitativas**, com diferentes níveis de granularidade:
* `fact_sales` → nível de pedido
* `fact_detail` → nível de item do pedido

## Dimensões
### `gold.dim_date` — Dimensão Tempo
**Objetivo:**
Padronizar a análise temporal e permitir agregações por diferentes níveis de tempo.

**Granularidade:**
1 linha por dia (2011–2025)

**Principais atributos:**
* `date_key` (YYYYMMDD – surrogate key)
* Dia, mês, ano, trimestre, semana
* Nome do dia e do mês
* Indicador de fim de semana

**Uso em análises:**
* Evolução de vendas no tempo
* Comparações mensais, trimestrais e anuais
* Análises de sazonalidade

### `gold.dim_customer` — Dimensão Cliente
**Objetivo:**
Centralizar informações do cliente e permitir análises por perfil de consumidor.

**Fonte:**
Dados consolidados da Silver (`customer` + `customer_address`)

**Características:**
* Uso de **surrogate key** (`customer_key`)
* Estrutura preparada para **Slowly Changing Dimension (SCD Tipo 2)**

**Principais atributos:**
* Identificadores do cliente
* Nome completo
* Email
* Endereços principais e de entrega
* Controle de vigência (`valid_from`, `valid_to`, `is_current`)

### `gold.dim_product` — Dimensão Produto
**Objetivo:**
Fornecer contexto completo do produto para análises comerciais e financeiras.

**Principais atributos:**
* Categoria e modelo
* Preço de lista e custo padrão
* Margem percentual
* Status do produto (Ativo / Descontinuado)

**Destaques:**
* Preparada para histórico de mudanças
* Controle de produtos descontinuados
* Base para cálculo de margem e lucro

### `gold.dim_address` — Dimensão Endereço
**Objetivo:**
Permitir análises geográficas e logísticas.

**Principais atributos:**
* Cidade, estado, país
* Código postal
* Endereço completo
* Flags de validação de endereço e CEP

**Uso comum:**
* Análise por região
* Comparação entre endereço de cobrança e entrega

>Notebook: [Dimensões](https://github.com/nadinne94/dabricks_data_engineer_learning_plan/blob/main/etl_adventureworks/10_Gold_Dimensions.ipynb)
## Tabelas Fato
### `gold.fact_sales` — Fato Vendas (Nível de Pedido)
**Descrição:**
Representa o **nível de pedido**, sem granularidade de item.

> Como um único pedido pode conter múltiplos produtos, esta tabela armazena **apenas informações do pedido em si**, enquanto o detalhamento dos itens é tratado em uma tabela fato separada.

**Granularidade:**
1 linha por pedido

**Principais métricas:**
* Subtotal
* Impostos
* Frete
* Valor total do pedido

**Chaves estrangeiras:**
* `order_date_key`
* `customer_key`
* `ship_address_key`
* `bill_address_key`

**Casos de uso:**
* Receita total
* Número de pedidos
* Ticket médio
* Análise de pedidos online vs. offline

>Notebook: [Fact Order](https://github.com/nadinne94/dabricks_data_engineer_learning_plan/blob/main/etl_adventureworks/11_Gold_Fact_Sales.ipynb)

### `gold.fact_detail` — Fato Detalhe de Vendas (Nível de Item)
**Descrição:**
Armazena o **nível mais granular da venda**, representando cada produto vendido dentro de um pedido.

**Granularidade:**
1 linha por item do pedido

**Principais métricas:**
* Quantidade
* Preço unitário
* Desconto
* Total da linha
* Lucro bruto
* Margem de lucro (%)

**Cálculos de negócio:**
* **Lucro Bruto:**
  `line_total - (standard_cost × quantity)`
* **Margem de Lucro (%):**
  `(lucro bruto / custo total) × 100`

**Uso em análises:**
* Produtos mais rentáveis
* Impacto de descontos
* Margem por categoria ou modelo

>Notebook: [Fact Detail](https://github.com/nadinne94/dabricks_data_engineer_learning_plan/blob/main/etl_adventureworks/12_Gold_Fact_Detail.ipynb)

## Persistência e Governança
* Todas as tabelas são gravadas em **Delta Lake**
* Persistência dupla:
  * Diretório físico (`gold_path`)
  * Tabela gerenciada (`gold.*`)
* Controle de execução via função `log_etl`
* Estrutura preparada para:
  * Auditoria
  * Monitoramento
  * Evolução incremental
