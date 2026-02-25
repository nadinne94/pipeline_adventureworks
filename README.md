# AdventureWorks Data Pipeline

<img src="https://img.shields.io/badge/language-SQL-blue" alt="Logo SQL" height="20"> ![Databricks](https://img.shields.io/badge/Databricks-FF3621?logo=databricks&logoColor=white)
<img src="https://img.shields.io/badge/Python-yellow?style=for-the-badge&logo=python&logoColor=blue" alt="Logo do Python" height="20"> ![Delta Lake](https://img.shields.io/badge/Delta_Lake-003366?logo=apachespark&logoColor=white) <img src="https://img.shields.io/badge/Apache_Spark-A689E1?style=for-the-badge&logo=apachespark&logoColor=#E35A16" height="20"> ![Medallion](https://img.shields.io/badge/Medallion%20Architecture-orange)

Este projeto simula a construção de um pipeline de dados corporativo utilizando o dataset AdventureWorks. O objetivo é estruturar um fluxo completo de ingestão, transformação e modelagem analítica seguindo boas práticas de Engenharia de Dados.

O pipeline foi desenvolvido com foco em arquitetura Lakehouse e modelagem dimensional para consumo analítico.

O projeto foi desenvolvido em ambiente Databricks utilizando processamento distribuído com Apache Spark.

## Arquitetura do Pipeline
O projeto segue a arquitetura Lakehouse, organizando os dados em três camadas:
![Diagrama Raw → Bronze → Silver → Gold](https://github.com/user-attachments/assets/9c0797c5-b911-447a-96a7-905b07b59b8a)

### Bronze
* Ingestão de dados brutos
* Preservação da estrutura original
* Base para rastreabilidade

### Silver
* Limpeza e padronização
* Tratamento de inconsistências
* Enriquecimento de dados

### Gold
* Modelagem dimensional (Star Schema)
* Criação de tabelas fato e dimensão
* Estrutura otimizada para consultas analíticas

## Stack Tecnológica
* Python
* PySpark
* SQL
* Delta Lake
* Git

## Configuração do Ambiente
O ambiente de dados foi configurado em plataforma distribuída, seguindo boas práticas de organização, governança e separação por camadas, garantindo escalabilidade e rastreabilidade desde a ingestão dos dados.

As principais configurações incluem:
- Criação de catálogo e schemas segregados por camada (bronze, silver e gold)
- Organização do armazenamento utilizando tabelas Delta
- Preparação de estrutura para governança e monitoramento do pipeline
- Padronização de nomenclaturas e ambientes

> Detalhamento da configuração do ambiente: [Configuração do Ambiente](setup_environment.md)

## Ingestão de Dados (Bronze Layer)
### Fontes de Dados
- Banco de dados relacional (SQL Server – AdventureWorks)
- Arquivos CSV (exportação de backup .bak)
- Estrutura preparada para ingestão via APIs e streaming

### Características
- Carga inicial full load com estrutura preparada para futuras cargas incrementais.
- Dados armazenados sem transformações
- Base para rastreabilidade e auditoria

### Tabelas Bronze:
```
- Sales.SalesOrderHeader
- Sales.SalesOrderDetail
- Production.Product
- Production.ProductCategory
- Production.ProductModel
- Production.ProductDescription
- Person.Customer
- Person.Address
- Person.CustomerAddress
```
>Detalhamento da camada Bronze: [Bronze Layer](https://github.com/nadinne94/pipeline_adventureworks/blob/main/bronze_ingestion.md)

## Tratamento e Qualidade de Dados (Silver Layer)
A Silver Layer é responsável por transformar os dados crus da Bronze Layer em **entidades de negócio confiáveis**, aplicando regras de qualidade, padronização e conformidade, mantendo estrutura normalizada orientada a entidades de negócio, sem aplicação de modelagem dimensional nesta etapa.

### Principais Atividades
- Limpeza de dados
- Padronização de formatos
- Aplicação de regras de negócio
- Enriquecimento de dados
### Estrutura da Silver Layer — Entidades de negócios
```
-- Entidades de Negócio (dados conformados)
silver.customers
silver.products
silver.addresses
silver.sales_orders
silver.sales_order_details
```
> A Silver Layer atua como base confiável para a construção da camada analítica (Gold), garantindo separação clara entre tratamento de dados e modelagem orientada ao negócio.

### Regras de Qualidade Aplicadas
- Validação de campos obrigatórios
- Deduplicação de registros
- Padronização de datas, moedas e unidades

### Governança de Dados
- Implementação planejada de catálogo de dados[^1]
- Planejamento de rastreabilidade (data lineage)[^2]

> Detalhamento da camada Silver: [Silver Layer](https://github.com/nadinne94/pipeline_adventureworks/blob/main/silver_layer.md)

> Os processos de governança são mais complexos e ficaram de fora do escopo inicial do projeto mas que serão aplicados posteriormente[^3]

## Camada Analítica — Gold Layer

A **Gold Layer** disponibiliza dados **prontos para análise**, aplicando **modelagem dimensional (Star Schema)** e **Data Marts analíticos**, com foco em performance e simplicidade de consumo.

### Modelagem Dimensional
A camada Gold foi modelada utilizando Star Schema, com:
* Tabelas fato contendo métricas de negócio
* Tabelas dimensão contendo atributos descritivos
* Relacionamentos estruturados para consultas analíticas eficientes

Essa abordagem permite:
* Melhor performance em consultas
* Facilidade na geração de KPIs
* Estrutura clara para ferramentas de BI

#### Dimensões
* `dim_date`
* `dim_customer`
* `dim_product`
* `dim_address`

#### Tabelas Fato
* `fact_sales` — grão definido no nível de pedido
* `fact_detail` - grão definido no nível de item do pedidonível de item do pedido

### Data Marts Analíticos
Além do modelo dimensional, a Gold Layer disponibiliza **Data Marts com métricas pré-agregadas**, prontos para consumo por ferramentas de BI e Analytics:
* `mart_sales_by_category`
* `mart_top_customers`
* `mart_monthly_performance`

>Detalhamento da camada Gold: [Gold Layer](https://github.com/nadinne94/pipeline_adventureworks/blob/main/gold_layer.md)

## Desafios Técnicos Enfrentados
Durante o desenvolvimento do pipeline, alguns desafios técnicos incluíram:
* Garantir integridade entre tabelas fato e dimensão
* Padronização de dados provenientes de múltiplas fontes
* Definição do grão das tabelas fato
* Separação adequada entre camada de tratamento (Silver) e modelagem (Gold)
Essas decisões foram fundamentais para garantir consistência analítica e escalabilidade.

## Decisões Arquiteturais
* Separação clara entre tratamento (Silver) e modelagem analítica (Gold)
* Utilização de Star Schema para otimizar consultas e reduzir complexidade de JOINs
* Uso de Delta Lake para garantir consistência transacional (ACID)
* Estrutura preparada para evolução para cargas incrementais

## Conceitos Aplicados
* ETL
* Arquitetura Lakehouse
* Modelagem Dimensional
* Star Schema
* JOINs complexos
* CTE
* Window Functions
* Versionamento com Delta Lake

## Escalabilidade e Evolução
Em ambiente produtivo, este pipeline poderia evoluir com:
* Particionamento por data
* Processamento incremental
* Orquestração com ferramenta dedicada (ex: Apache Airflow)
* Monitoramento e logging
* Validação de qualidade de dados

---
## Notas
[^1]:Catálogo de dados: organiza e descreve os dados disponíveis
[^2]:Data Lineage representa o fluxo dos dados desde a origem até o consumo final.
[^3]:Processos implementação de pipeline por níveis: [Checklist de Implementação](https://www.linkedin.com/in/nadinne-cavalcante/)
