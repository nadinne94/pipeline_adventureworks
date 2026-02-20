# AdventureWorks Data Pipeline

<img src="https://img.shields.io/badge/language-SQL-blue" alt="Logo SQL" height="20"> ![Databricks](https://img.shields.io/badge/Databricks-FF3621?logo=databricks&logoColor=white)
<img src="https://img.shields.io/badge/Python-yellow?style=for-the-badge&logo=python&logoColor=blue" alt="Logo do Python" height="20"> ![Delta Lake](https://img.shields.io/badge/Delta_Lake-003366?logo=apachespark&logoColor=white) <img src="https://img.shields.io/badge/Apache_Spark-A689E1?style=for-the-badge&logo=apachespark&logoColor=#E35A16" height="20"> ![Medallion](https://img.shields.io/badge/Medallion%20Architecture-orange)

Pipeline de dados analítico desenvolvido com base no AdventureWorks, aplicando boas práticas de Engenharia de Dados, modelagem dimensional e qualidade de dados, com foco em escalabilidade, governança e consumo analítico.

Este repositório apresenta a visão geral de um pipeline de dados end-to-end. A documentação detalhada de cada camada está disponível nos links abaixo.

## Objetivo do Projeto
Construir um pipeline de dados end-to-end, desde a ingestão de dados operacionais até a disponibilização de dados business-ready, simulando um cenário real de ambiente corporativo analítico.

## Arquitetura de Dados
Arquitetura baseada no padrão Medallion (Bronze / Silver / Gold), amplamente utilizada em plataformas modernas de dados.

![Diagrama Raw → Bronze → Silver → Gold](https://github.com/user-attachments/assets/9c0797c5-b911-447a-96a7-905b07b59b8a)

Cada camada possui responsabilidades bem definidas, desde ingestão de dados brutos até disponibilização de dados analíticos prontos para consumo.

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
- Carga full load inicial
- Preservação do histórico completo
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
A Silver Layer é responsável por transformar os dados crus da Bronze Layer em **entidades de negócio confiáveis**, aplicando regras de qualidade, padronização e conformidade, **sem aplicação de modelagem dimensional**.

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
- Criar catálago de dados[^1]
- Configurar linha de dados[^2]

> Detalhamento da camada Silver: [Silver Layer](https://github.com/nadinne94/pipeline_adventureworks/blob/main/silver_layer.md)

> Os processos de governança são mais complexos e ficaram de fora do escopo inicial do projeto mas que serão aplicados posteriormente[^3]

## Camada Analítica (Gold Layer)
Disponibilizar dados prontos para análise por meio de **modelagem dimensional (Star Schema)**, com foco em performance, simplicidade de uso e consumo por ferramentas analíticas.

### Modelagem
#### Modelo Dimensional (Star Schema)
- Dimensões
  - dim_consumer
  - dim_address
  - dim_product
- Fatos
  - fact_sales_header
  - fact_sales_detail
### Benefícios
- Consultas analíticas otimizadas
- Facilidade de uso por times de BI e Analytics
- Integração direta com ferramentas de visualização (ex.: Power BI)

>Detalhamento da camada Gold: [Gold Layer](https://github.com/nadinne94/pipeline_adventureworks/blob/main/gold_layer.md)

## Tecnologias Utilizadas
- SQL
- Python / PySpark
- Spark
- Delta Lake
- SQL Server

## Próximos Passos
- Incremental load com CDC
- Streaming de dados
- Mascaramento de PII
- Testes automatizados de qualidade
- Dashboards analíticos
---
## Notas
[^1]:Catálogo de dados: organiza e descreve os dados disponíveis
[^2]:Linha de dados mostra o caminho que eles percorrem da origem até o consumo
[^3]:Processos implementação de pipeline por níveis: [Checklist de Implementação](https://www.linkedin.com/in/nadinne-cavalcante/)
