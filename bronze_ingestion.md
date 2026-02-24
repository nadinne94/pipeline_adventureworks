# Bronze Layer — Ingestão de Dados
Este notebook é responsável pela ingestão inicial dos dados na Bronze Layer, seguindo os princípios da arquitetura Medallion.
O objetivo é carregar os dados exatamente como chegam da origem, preservando histórico, rastreabilidade e metadados de ingestão, sem aplicar regras de negócio ou transformações analíticas.

![Ingestão de Dados](https://github.com/user-attachments/assets/c71695be-1b09-40d4-be2b-a83af87dc6f4)

## Inicialização do Ambiente
Antes da execução da ingestão, o notebook reutiliza um módulo de configuração centralizado.
```
%run ./00_Setup_Environment
```
Essa abordagem garante:
- Reuso de configurações e funções comuns
- Padronização do ambiente entre notebooks
- Redução de código duplicado
- Facilidade de manutenção do pipeline

## Configuração de Credenciais e Contexto
Nesta etapa são definidos os principais parâmetros utilizados durante a ingestão.
```
source_path = '/Volumes/adventureworks/dataset/sales'
schema = 'adventureworks.bronze'
```
Essas variáveis determinam:
- O local de origem dos arquivos CSV
- O schema de destino correspondente à Bronze Layer
- A centralização dessas configurações facilita ajustes futuros e evita hardcoding distribuído ao longo do código.

## Identificação Dinâmica dos Arquivos de Origem

O pipeline identifica automaticamente os arquivos CSV disponíveis no diretório de origem.
```
for f in dbutils.fs.ls(source_path):
  if f.name.endswith(".csv")
```
Esse processo permite:
- Ingestão automática de múltiplas tabelas
- Flexibilidade para adicionar novos arquivos sem alterar o código
- Execução orientada a metadados, não a nomes fixos

## Padronização dos Nomes das Tabelas
Os nomes dos arquivos são convertidos para o padrão snake_case, alinhado às boas práticas de modelagem e nomenclatura em ambientes analíticos.
```
for table in files_name:
  table_name = to_snake_case(table)
  tables.append((table, table_name))
```
Essa etapa garante:
- Consistência entre tabelas
- Facilidade de uso em SQL
- Padronização entre camadas do pipeline

## Função de Ingestão para a Bronze Layer
A ingestão é encapsulada em uma função reutilizável, responsável por todo o fluxo de carregamento.

**Responsabilidades da função:**
- Leitura dos arquivos CSV
- Inclusão de metadados de ingestão
- Persistência dos dados em formato Delta
- Criação das tabelas no catálogo
- Registro de métricas operacionais

```
def ingest_table(source_table, target_table):
  """Ingere arquivo csv do Volume para Bronze"""
  start = time.time()
    
  try:  ...       
     # Ler dados
     df = spark.read...
        
     # Adicionar metadados
     df = df \
           .withColumn("_ingestion_timestamp", current_timestamp()) \
           .withColumn("_source_table", lit(source_table)) \
           .withColumn("_batch_id", lit(str(uuid.uuid4())[:8]))
        
     # Salvar como Delta e tabela no catálogo
     df.write...
 
     # Registrar metadados
     duration = round(time.time() - start, 2)
     count = df.count()
     log_etl(target_table, "bronze", "SUCCESS", count, None, duration)
     return count
        
   except Exception as e:
       duration = round(time.time() - start, 2)
       log_etl(target_table, "bronze", "FAILED", 0, str(e), duration)
       print(f"Erro: {str(e)[:200]}...")
       raise e
```

Metadados adicionados:
- `_ingestion_timestamp`: momento da ingestão
- `_source_table`: tabela de origem
- `_batch_id`: identificador único da carga

Esses campos garantem rastreabilidade, auditoria e suporte a reprocessamentos futuros.

## Persistência dos Dados
Os dados são gravados:
- Em formato Delta Lake
- Com schema sobrescrito quando necessário
- Seguindo organização por camada no workspace
Essa estratégia assegura:
- Confiabilidade
- Evolução de schema controlada
- Compatibilidade com cargas incrementais futuras

## Governança e Logging da Execução
Cada execução é registrada na tabela de controle de ETL, armazenando informações como:
- Tabela processada
- Camada do pipeline
- Quantidade de registros
- Status da execução
- Tempo de processamento
- Mensagens de erro (quando aplicável)
```
log_etl(target_table, "bronze", "SUCCESS", ...)
```
Esse mecanismo fornece observabilidade básica, essencial para pipelines de produção.

## Execução em Lote da Ingestão
A ingestão é executada de forma iterativa para todas as tabelas identificadas.
```
for source, target in tables:
  try:
        count = ingest_table(source, target)
        results[target] = count
    except Exception as e:
        results[target] = f"ERRO: {str(e)[:50]}"
```
Benefícios dessa abordagem:
- Escalabilidade do pipeline
- Tratamento individual de falhas
- Continuidade da execução mesmo com erros pontuais

## Validação Final
Ao final do processo, é realizada uma verificação das tabelas criadas na Bronze Layer.
```
SHOW TABLES IN bronze;
```
Essa validação confirma:
- Criação correta das tabelas
- Registro no catálogo
- Conclusão bem-sucedida da ingestão

## Resultado da Bronze Layer
Ao final deste notebook, a Bronze Layer contém:
- Dados crus e imutáveis
- Histórico preservado
- Metadados de ingestão
- Base confiável para a Silver Layer

>Notebook: [Bronze_Ingestion](https://github.com/nadinne94/dabricks_data_engineer_learning_plan/blob/main/etl_adventureworks/01_Bronze_Ingestion.ipynb)

---
<div align='left'> 

  [Configuração do Ambiente](setup_environment.md)
</div><div align='right'>
  
  [Camada Silver](silver_layer.md)
</div>

---
