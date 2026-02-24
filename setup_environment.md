# Configuração do Ambiente de Dados (Databricks)
Este documento descreve a configuração técnica do ambiente de dados utilizado no projeto, com foco em organização por camadas, governança, rastreabilidade e suporte à execução do pipeline analítico.


> Notebook: [Configuração do Ambiente](https://github.com/nadinne94/dabricks_data_engineer_learning_plan/blob/main/etl_adventureworks/00_Setup_Environment.ipynb)

## Importação de Bibliotecas

Bibliotecas utilizadas para manipulação de dados com PySpark, transformação de schemas, controle de janelas analíticas e suporte a funções auxiliares do pipeline.

```
%python    

from pyspark.sql.functions import *
from pyspark.sql.types import *
from pyspark.sql import Window
import re
import time
import uuid
from datetime import datetime
```

## Criar Catálogo e Schema
O ambiente é organizado a partir de um catálogo único (`adventureworks`), com schemas separados por camada do pipeline (Bronze, Silver e Gold).
Cada schema possui um volume dedicado para armazenamento de tabelas Delta.
A governança do pipeline é centralizada no schema `metadata`, que contém a tabela de controle de execução (`etl_control`), responsável por registrar status, métricas e erros das cargas.

![Organização do Ambiente](https://github.com/user-attachments/assets/7874d8a4-17fc-4fa2-bafb-859a0e0c7242)

A estrutura abaixo segue o padrão Medallion Architecture, garantindo separação clara de responsabilidades, governança e escalabilidade do pipeline de dados.

```
%sql
-- Criação do catálogo principal do projeto
-- Responsável por organizar e isolar todos os objetos relacionados ao pipeline AdventureWorks
CREATE CATALOG IF NOT EXISTS adventureworks;
USE CATALOG adventureworks;

-- Camada Bronze: dados crus, históricos e imutáveis
CREATE SCHEMA IF NOT EXISTS bronze;
CREATE VOLUME IF NOT EXISTS bronze.delta_tables;

-- Camada Silver: dados tratados, padronizados e conformados
CREATE SCHEMA IF NOT EXISTS silver;
CREATE VOLUME IF NOT EXISTS silver.delta_tables;

-- Camada Gold: dados analíticos, agregados e orientados ao negócio
CREATE SCHEMA IF NOT EXISTS gold;
CREATE VOLUME IF NOT EXISTS gold.delta_tables;

-- Schema de metadados para governança, monitoramento e controle de execução do pipeline
CREATE SCHEMA IF NOT EXISTS metadata;
```

## Configuração de Caminhos no Workspace
Esta etapa define os caminhos de armazenamento utilizados pelo pipeline dentro do workspace, organizando os dados de forma padronizada por camada (Bronze, Silver e Gold).

A utilização de variáveis para os caminhos garante:
- Centralização das configurações de storage
- Facilidade de manutenção e evolução do pipeline
- Separação clara entre dados crus, tratados e analíticos
- Redução de erros por hardcoding de caminhos

Essa abordagem facilita a escalabilidade do ambiente e a padronização do armazenamento ao longo do ciclo de vida do projeto.

```
%python
# Armazenar caminhos dentro de variáveis
base_path = '/Volumes/adventureworks'
bronze_path = f"{base_path}/bronze/delta_tables"
silver_path = f"{base_path}/silver/delta_tables"
gold_path = f"{base_path}/gold/delta_tables"

# Criar diretórios
dbutils.fs.mkdirs(bronze_path)
dbutils.fs.mkdirs(silver_path)
dbutils.fs.mkdirs(gold_path)
```
## Governança e Controle do Pipeline
Esta seção implementa mecanismos básicos de governança e observabilidade do pipeline, permitindo o acompanhamento da execução dos processos de ETL ao longo das diferentes camadas.

A tabela de controle de execução armazena informações como:
- Nome da tabela processada
- Camada do pipeline (Bronze, Silver ou Gold)
- Data e horário da última execução
- Volume de registros processados
- Status da execução (sucesso ou erro)
- Mensagens de erro e tempo de processamento

Esses dados possibilitam auditoria, troubleshooting e monitoramento operacional, além de servirem como base para métricas de qualidade e alertas futuros.

```
%sql
CREATE TABLE IF NOT EXISTS metadata.etl_control (
  table_name STRING,
  layer STRING,
  last_processed_date TIMESTAMP,
  records_count BIGINT,
  status STRING,
  error_message STRING,
  processing_time DOUBLE,
  created_at TIMESTAMP
)
USING DELTA;
```
## Funções Auxiliares de Monitoramento
As funções auxiliares implementadas nesta seção são responsáveis por registrar informações operacionais sobre a execução do pipeline, promovendo maior observabilidade e controle dos processos de ETL.

A função de logging centraliza o registro de eventos de execução, permitindo:
- Padronização dos logs entre diferentes etapas do pipeline
- Registro consistente de sucesso e falha
- Coleta de métricas operacionais (tempo de execução, volume de dados)
- Suporte à análise de falhas e performance

```
%python
def log_etl(table_name, layer, status, records=0, error=None, duration=0):
    """Registra execução do ETL"""
    spark.sql(f"""
        INSERT INTO metadata.etl_control 
        VALUES (
            '{table_name}',
            '{layer}',
            CURRENT_TIMESTAMP(),
            {records},
            '{status}',
            {f"'{error}'" if error else 'NULL'},
            {duration},
            CURRENT_TIMESTAMP()
        )
    """)
    print(f"{'SUCCESS' if status=='SUCCESS' else 'ERROR'} {layer}.{table_name}: {status} - {records} registros - {duration}s")
```
Além disso, funções utilitárias complementares auxiliam na padronização de nomenclaturas e no reaproveitamento de lógica comum ao longo do projeto.
```
%python
#Função utilitária para padronização de nomes de colunas
def to_snake_case(name: str) -> str:
    name = re.sub('(.)([A-Z][a-z]+)', r'\1_\2', name)
    name = re.sub('([a-z0-9])([A-Z])', r'\1_\2', name)
    return name.lower()
```
---
<div align='left'> 

  [Introdução](README.md)
</div><div align='right'>
  
  [Camada Bronze](bronze_ingestion.md)
</div>

---
## Referências
1.Configurar e ajustar um ambiente do Azure Databricks - [Microsoft Learn](https://learn.microsoft.com/pt-br/training/paths/azure-databricks-data-engineer-set-up-configure-environment/)
