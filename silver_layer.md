# Silver Layer — Tratamento e Padronização de Dados
A camada _**Silver**_ é responsável por transformar os dados crus ingeridos na Bronze Layer em **entidades de negócio confiáveis**, padronizadas e enriquecidas, prontas para suportar análises e modelagem dimensional na camada Gold.

Nesta camada, os dados passam por processos de:
* Limpeza
* Padronização de formatos
* Enriquecimento entre entidades
* Aplicação de regras de qualidade
* Registro de métricas e status do pipeline

## Estrutura de Dados da Silver Layer
A Silver Layer é organizada por **entidades de negócio**, e não por fatos ou dimensões analíticas.
```text
silver.address
silver.customer
silver.customer_address
silver.product
silver.sales_orders
silver.sales_detail
```

## Padrões Técnicos Aplicados
Todos os pipelines da Silver Layer seguem um padrão comum:
* Leitura de dados da Bronze ou Silver
* Padronização de nomes de colunas (snake_case)
* Normalização de tipos de dados (int, decimal, date, timestamp)
* Inclusão de metadados técnicos
* Escrita em formato **Delta Lake**
* Registro de execução via função de logging

Campos técnicos comuns:
* `source_modified_date`
* `processed_timestamp`
* Flags de qualidade (`is_valid_*`, `valid_*`)

## Transformação de Endereços (`silver.address`)
* Padronização de texto (trim, initcap)
* Construção de campo `full_address`
* Validação básica de endereço e CEP
* Remoção de duplicidades
* Preparação do dado para reutilização por clientes e vendas

Notebook: [Address](https://github.com/nadinne94/dabricks_data_engineer_learning_plan/blob/main/etl_adventureworks/02_Silver_Address.ipynb)

## Transformação de Clientes (`silver.customer`)
* Padronização de nomes e dados pessoais
* Construção do campo `full_name`
* Validação de clientes válidos (nome e sobrenome)
* Inclusão de dados comerciais (empresa, vendedor)
* Preparação do cliente como entidade central de negócio

Notebook: [Customer](https://github.com/nadinne94/dabricks_data_engineer_learning_plan/blob/main/etl_adventureworks/03_Silver_Customer.ipynb)

## Relacionamento Cliente–Endereço (`silver.customer_address`)
* Agrupamento de múltiplos endereços por cliente
* Identificação de:
  * Endereço principal
  * Endereço de entrega
* Criação de uma visão consolidada do relacionamento cliente ↔ endereço
Essa tabela atua como **entidade conformada**, simplificando joins futuros na Gold Layer.

Notebook: [CustomerAddress](https://github.com/nadinne94/dabricks_data_engineer_learning_plan/blob/main/etl_adventureworks/04_Silver_CustomerAddress.ipynb)

## Transformação de Produtos (`silver.product`)
* Enriquecimento com categoria, modelo e descrição
* Padronização de atributos físicos e financeiros
* Conversão segura de preços e custos
* Classificação do status do produto (Ativo, Inativo, Descontinuado)
* Consolidação de todas as informações relevantes do produto em uma única entidade.

Notebook: [Products](https://github.com/nadinne94/dabricks_data_engineer_learning_plan/blob/main/etl_adventureworks/05_Silver_Products.ipynb)

## Transformação de Vendas — Orders (`silver.sales_orders`)
* Consolidação do cabeçalho do pedido
* Enriquecimento com dados de cliente e endereço
* Normalização de datas e valores monetários
* Particionamento por ano e mês para performance
Um pedido pode conter múltiplos produtos; por isso, o modelo separa o cabeçalho do pedido do detalhamento dos itens, mantendo a granularidade no nível de pedido.

Notebook: [Sales Order Header](https://github.com/nadinne94/dabricks_data_engineer_learning_plan/blob/main/etl_adventureworks/06_Silver_Sales.ipynb)

## Transformação de Vendas — Detail (`silver.sales_detail`)
* Granularidade por item vendido
* Enriquecimento com dados do produto
* Cálculos financeiros:
  * Subtotal
  * Descontos
  * Valor final da linha
* Flags de qualidade para quantidade e preço
  
Notebook: [Sales Order Detail](https://github.com/nadinne94/dabricks_data_engineer_learning_plan/blob/main/etl_adventureworks/07_SilverDetails.ipynb)

## Funções Auxiliares de Qualidade e Governança
Foram implementadas funções reutilizáveis para controle da qualidade dos dados:
* `cast_columns`: padroniza tipos de colunas `_id`
* `nulls_report`: identifica colunas com valores nulos
* `duplicates_report`: detecta registros duplicados
* `run_data_quality`: gera métricas de nulos por tabela
  
Notebook: [Funções Auxiliares](https://github.com/nadinne94/dabricks_data_engineer_learning_plan/blob/main/etl_adventureworks/08_Def_Cleaning.ipynb)

## Métricas e Monitoramento
A Silver Layer registra métricas em tabelas de metadados:
```text
metadata.data_quality_metrics
metadata.pipeline_status
```
Essas tabelas permitem:
* Acompanhamento da qualidade ao longo do tempo
* Identificação de falhas automáticas por threshold
* Auditoria de execuções do pipeline
  
Notebook: [Métricas e Monitoramento](https://github.com/nadinne94/dabricks_data_engineer_learning_plan/blob/main/etl_adventureworks/09_Metrics.ipynb)

## Papel da Silver Layer na Arquitetura
A Silver Layer atua como **camada confiável e reutilizável**, desacoplando:
* A complexidade do dado cru (Bronze)
* Da lógica analítica e dimensional (Gold)
Essa separação garante escalabilidade, governança e evolução segura do pipeline.

