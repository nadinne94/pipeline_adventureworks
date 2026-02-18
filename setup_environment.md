# Configuração do Ambiente
A configuração do ambiente é útil porque ...

## Importar bibliotecas
Bibliotecas básicas pra utilizar pyspark, transformar arquivos formato de data
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
Para armazenar as tabelas de acordo com cada etapa de transformação foram armazenados em schemas separados, e como não estavamos utilizando volume externo, criamos volumes para armazer as tabelas delta. Um schema de metados para armazer dados de governança.

```
%sql
CREATE CATALOG IF NOT EXISTS adventureworks;
USE CATALOG adventureworks;

CREATE SCHEMA IF NOT EXISTS bronze;
CREATE VOLUME IF NOT EXISTS bronze.delta_tables;

CREATE SCHEMA IF NOT EXISTS silver;
CREATE VOLUME IF NOT EXISTS silver.delta_tables;

CREATE SCHEMA IF NOT EXISTS gold;
CREATE VOLUME IF NOT EXISTS gold.delta_tables;

CREATE SCHEMA IF NOT EXISTS metadata;
```
## Configurar caminhos no Workspace
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
## Governança de dados
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
## Funções
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

```
%python
def to_snake_case(name: str) -> str:
    name = re.sub('(.)([A-Z][a-z]+)', r'\1_\2', name)
    name = re.sub('([a-z0-9])([A-Z])', r'\1_\2', name)
    return name.lower()
```
## Referências
1.[Configurar e ajustar um ambiente do Azure Databricks](https://learn.microsoft.com/pt-br/training/paths/azure-databricks-data-engineer-set-up-configure-environment/)
