---
name: data-quality-guardrails
description: Use ao implementar, adicionar, criar, revisar ou validar guardrails em pipelines de dados em notebooks de qualquer camada (bronze, silver, gold). Cobre validacoes de qualidade, integridade estrutural, reconciliacao quantitativa e resiliencia operacional. NAO use para decisoes arquiteturais gerais ou revisao de codigo sem foco em guardrails.

---

# Guardrails em Pipelines de Dados

**Versão:** 1.0.1 | **Data:** 2026-08-18 | **Domínio:** data-quality | **Autor:** Pedro O. Silva

## Princípio Fundador

Guardrails não são "validações adicionadas no final" — são **contratos estruturais** que definem o comportamento esperado do pipeline em condições normais e de falha. Um pipeline production-grade deve falhar de forma **previsível, rastreável e recuperável**.

---

## 1. Taxonomia: 4 Tipos de Guardrails

### 1.1 Validações de Qualidade
**O quê**: Verificar que os dados atendem padrões esperados  
**Quando**: Silver → Gold (dados limpos → regras de negócio)  
**Exemplos**:
* Schema validation (tipos, colunas obrigatórias)
* Completude (% de nulos por coluna)
* Freshness (última atualização dentro da janela esperada)
* Valores válidos (categorias, ranges numéricos)
* Detecção de anomalias (outliers estatísticos)

### 1.2 Integridade Estrutural
**O quê**: Garantir que relações entre dados são consistentes  
**Quando**: Todas as camadas (especialmente joins em Silver)  
**Exemplos**:
* Unicidade de chaves primárias
* Cardinalidade de joins (1:1, 1:N esperado)
* Chaves estrangeiras existem (completude referencial)
* Duplicatas detectadas e tratadas
* Nulos em colunas críticas

### 1.3 Integridade Quantitativa / Reconciliação ⭐
**O quê**: Garantir que transformações preservam quantidades esperadas  
**Quando**: CRÍTICO em todas as transições (Bronze → Silver → Gold)  
**Por que é especial**: Em domínios financeiros, contábeis e transacionais, perder ou duplicar valores monetários/contagens é inaceitável  
**Exemplos**:
* Soma de valores monetários entrada = saída
* Contagem de registros antes/depois de transformação
* Proporções mantidas em agregações
* Registros descartados com justificativa

### 1.4 Resiliência Operacional
**O quê**: Garantir que falhas parciais não destroem todo o trabalho  
**Quando**: Todas as camadas (especialmente Bronze com APIs externas ou fontes instáveis)  
**Exemplos**:
* Retry logic com exponential backoff
* Tratamento granular de erros (falha em 1 ano não destrói 4)
* Checkpointing (tabela de controle de estado)
* Logging estruturado
* Validação de pré-requisitos

**IMPORTANTE**: A necessidade de cada guardrail varia pelo tipo de fonte em Bronze:
* **APIs externas/fontes instáveis**: Retry logic crítico
* **Arquivos governamentais/fontes confiáveis**: Retry logic não aplicável (arquivo já está local)

---

## 2. Primitivas de Reconciliação (3 Universais)

Não importa o domínio — toda reconciliação se reduz a 3 primitivas:

### 2.1 Igualdade de Soma (valores contínuos)
**Aplicável a**: Valores monetários, métricas físicas, durações

```python
# Exemplo: Validar que transformação não alterou totais financeiros
total_entrada = df_in.agg(sum("valor")).collect()[0][0]
total_saida = df_out.agg(sum("valor")).collect()[0][0]

TOLERANCIA = 0.01  # Tolerância para erros de arredondamento
assert abs(total_entrada - total_saida) < TOLERANCIA, \
    f"Diferença detectada: entrada={total_entrada}, saída={total_saida}, diff={total_entrada - total_saida}"

logger.info(f"✅ Reconciliação de soma: entrada={total_entrada}, saída={total_saida}")
```

**Quando usar**: Bronze → Silver (limpeza preserva valores), Silver → Gold (agregações corretas)

### 2.2 Igualdade de Contagem (cardinalidade)
**Aplicável a**: Número de registros, entidades únicas, contadores

```python
# Exemplo: Validar que nenhum registro foi perdido
count_entrada = df_in.count()
count_saida = df_out.count()

assert count_entrada == count_saida, \
    f"Perda/duplicação detectada: entrada={count_entrada}, saída={count_saida}"

logger.info(f"✅ Reconciliação de contagem: {count_entrada} registros preservados")
```

**Variação**: Validar cardinalidade de joins
```python
# Join 1:1 esperado
count_antes = df_left.count()
count_depois = df_joined.count()
assert count_antes == count_depois, \
    f"Join alterou cardinalidade: {count_antes} → {count_depois} (esperado 1:1)"
```

### 2.3 Completude Referencial (integridade relacional)
**Aplicável a**: Chaves estrangeiras, dimensões, proporções

```python
# Exemplo: Validar que todas as chaves foram preservadas
ids_entrada = set(df_in.select("id").rdd.flatMap(lambda x: x).collect())
ids_saida = set(df_out.select("id").rdd.flatMap(lambda x: x).collect())

ids_perdidos = ids_entrada - ids_saida
ids_novos = ids_saida - ids_entrada

assert len(ids_perdidos) == 0, \
    f"IDs perdidos durante transformação: {ids_perdidos}"

if ids_novos:
    logger.warning(f"⚠️ IDs novos detectados (verificar se esperado): {ids_novos}")

logger.info(f"✅ Completude referencial: {len(ids_entrada)} IDs preservados")
```

**Quando usar**: Joins, agregações com groupBy, filtros

---

## 3. Resiliência Operacional

### 3.1 Retry Logic (Falhas Transitórias)

**Quando aplicar retry**:
* ✅ Chamadas HTTP/APIs (timeout, 5xx, rate limit)
* ✅ Leitura de arquivos remotos (rede instável)
* ✅ Escrita em storage (contenção temporária)
* ✅ Operações Delta (conflitos de transação)

**NÃO aplicar retry em**:
* ❌ Erros de schema (dado mal-formado não melhora com retry)
* ❌ Erros de lógica (bug no código)
* ❌ 4xx HTTP (Bad Request, Unauthorized — problema permanente)

#### Padrão: Exponential Backoff com Jitter

```python
import time
import random

def retry_com_backoff(funcao, max_tentativas=3, backoff_inicial=1):
    """
    Retry com exponential backoff + jitter.
    
    Args:
        funcao: Função a executar
        max_tentativas: Máximo de tentativas
        backoff_inicial: Tempo inicial de espera (segundos)
    """
    for tentativa in range(1, max_tentativas + 1):
        try:
            return funcao()
        except Exception as e:
            if tentativa == max_tentativas:
                raise  # Última tentativa, propaga erro
            
            # Exponential backoff: 1s, 2s, 4s, 8s...
            espera = backoff_inicial * (2 ** (tentativa - 1))
            # Jitter: randomiza ±20% para evitar thundering herd
            espera = espera * (0.8 + 0.4 * random.random())
            
            print(f"[RETRY] Tentativa {tentativa}/{max_tentativas} falhou: {e}")
            print(f"[RETRY] Aguardando {espera:.1f}s antes de tentar novamente")
            time.sleep(espera)
```

**Por que jitter?** Se 100 jobs falharem simultaneamente e todos esperarem exatamente 2s, todos vão bater na API ao mesmo tempo de novo — criando um *thundering herd*. Jitter espalha as tentativas.

#### Rate Limit Específico

Quando a API retorna `429 Too Many Requests` ou `Retry-After` header:

```python
import requests
import time

def request_com_rate_limit(url, max_tentativas=5):
    for tentativa in range(1, max_tentativas + 1):
        response = requests.get(url)
        
        if response.status_code == 200:
            return response
        
        if response.status_code == 429:
            # Respeitar Retry-After se disponível
            retry_after = int(response.headers.get('Retry-After', 60))
            print(f"[RATE LIMIT] Aguardando {retry_after}s")
            time.sleep(retry_after)
            continue
        
        # Outros erros: backoff padrão
        response.raise_for_status()
```

### 3.2 Tratamento Granular de Erros

**Princípio**: Falha Parcial ≠ Falha Total

**Antipadrão** (❌):
```python
# Processar 5 anos em um único bloco try/except
try:
    for ano in [2021, 2022, 2023, 2024, 2025]:
        processar_ano(ano)  # Se 2022 falhar, TUDO falha
except Exception as e:
    print(f"Erro: {e}")
    # Perdeu todo o trabalho de 2021
```

**Padrão robusto** (✅):
```python
# Try/except POR ano — falha isolada
anos_sucesso = []
anos_falha = []

for ano in [2021, 2022, 2023, 2024, 2025]:
    try:
        processar_ano(ano)
        anos_sucesso.append(ano)
        print(f"✅ Ano {ano} processado com sucesso")
    except Exception as e:
        anos_falha.append((ano, str(e)))
        print(f"❌ Ano {ano} falhou: {e}")
        # Continua processando próximos anos

# Relatório final
print(f"\n📊 Resumo: {len(anos_sucesso)} sucessos, {len(anos_falha)} falhas")
if anos_falha:
    print("\n⚠️ Anos que falharam:")
    for ano, erro in anos_falha:
        print(f"  - {ano}: {erro}")
```

**Ganho**: Se 2022 falha (arquivo corrompido), 2021/2023/2024/2025 são processados. Reprocessamento só precisa corrigir 2022.

### 3.3 Logging Estruturado

Cada etapa deve logar:
* Timestamp
* Ação (o que está tentando fazer)
* Contexto (ano, arquivo, tabela)
* Status (início, sucesso, falha)
* Duração

```python
import logging
import time
from datetime import datetime

# Configurar logger estruturado
logging.basicConfig(
    level=logging.INFO,
    format='%(asctime)s | %(levelname)s | %(message)s',
    datefmt='%Y-%m-%d %H:%M:%S'
)
logger = logging.getLogger(__name__)

def processar_ano_com_log(ano):
    inicio = time.time()
    logger.info(f"[INÍCIO] Processando ano={ano}")
    
    try:
        # Lógica de processamento
        resultado = processar_ano(ano)
        
        duracao = time.time() - inicio
        logger.info(f"[SUCESSO] ano={ano} | duração={duracao:.2f}s | registros={resultado['count']}")
        return resultado
        
    except Exception as e:
        duracao = time.time() - inicio
        logger.error(f"[FALHA] ano={ano} | duração={duracao:.2f}s | erro={str(e)}")
        raise
```

### 3.4 Checkpointing e Retomada

#### Tabela de Controle (State Management)

Para pipelines que processam múltiplos períodos/arquivos, manter tabela de controle:

```sql
CREATE TABLE IF NOT EXISTS bronze.controle_ingestao (
    fonte          STRING,
    ano            INT,
    status         STRING,  -- 'pendente', 'processando', 'sucesso', 'falha'
    data_inicio    TIMESTAMP,
    data_fim       TIMESTAMP,
    tentativas     INT,
    erro           STRING,
    registros      BIGINT,
    CONSTRAINT pk PRIMARY KEY (fonte, ano)
)
```

#### Padrão: Checkpoint antes de iniciar, commit após sucesso

```python
from pyspark.sql.functions import current_timestamp, lit

def processar_com_checkpoint(fonte, ano):
    # 1. Marcar como 'processando'
    spark.sql(f"""
        MERGE INTO bronze.controle_ingestao t
        USING (SELECT '{fonte}' as fonte, {ano} as ano) s
        ON t.fonte = s.fonte AND t.ano = s.ano
        WHEN MATCHED THEN UPDATE SET
            status = 'processando',
            data_inicio = current_timestamp(),
            tentativas = t.tentativas + 1
        WHEN NOT MATCHED THEN INSERT
            (fonte, ano, status, data_inicio, tentativas)
            VALUES ('{fonte}', {ano}, 'processando', current_timestamp(), 1)
    """)
    
    try:
        # 2. Processar
        resultado = processar_ano(ano)
        
        # 3. Marcar como 'sucesso'
        spark.sql(f"""
            UPDATE bronze.controle_ingestao
            SET status = 'sucesso',
                data_fim = current_timestamp(),
                registros = {resultado['count']},
                erro = NULL
            WHERE fonte = '{fonte}' AND ano = {ano}
        """)
        
    except Exception as e:
        # 4. Marcar como 'falha'
        spark.sql(f"""
            UPDATE bronze.controle_ingestao
            SET status = 'falha',
                data_fim = current_timestamp(),
                erro = '{str(e).replace("'", "''")}'  -- Escape SQL
            WHERE fonte = '{fonte}' AND ano = {ano}
        """)
        raise
```

#### Detecção Inteligente de Períodos Pendentes

```python
def get_anos_pendentes(fonte, janela_anos=5):
    """
    Retorna anos pendentes ou com falha, dentro da janela temporal.
    """
    ano_atual = datetime.now().year
    ano_inicio = ano_atual - janela_anos + 1
    
    df_controle = spark.sql(f"""
        SELECT ano, status, tentativas
        FROM bronze.controle_ingestao
        WHERE fonte = '{fonte}'
          AND ano BETWEEN {ano_inicio} AND {ano_atual}
    """)
    
    # Anos dentro da janela
    anos_janela = set(range(ano_inicio, ano_atual + 1))
    
    # Anos já processados com sucesso
    anos_sucesso = set(
        row.ano for row in df_controle.filter("status = 'sucesso'").collect()
    )
    
    # Anos pendentes = janela - sucesso
    anos_pendentes = sorted(anos_janela - anos_sucesso)
    
    return anos_pendentes
```

### 3.5 Validação de Pré-requisitos

**Antipadrão**: Começar processamento e falhar no meio.

**Padrão robusto**: Validar tudo ANTES de iniciar.

```python
def validar_prerequisitos(fonte, ano):
    """
    Valida que todos os pré-requisitos existem antes de processar.
    Retorna (bool, mensagem_erro).
    """
    # 1. Arquivo na Landing Zone existe?
    arquivo_path = f"/Volumes/cvm/landing/{fonte}/{ano}/arquivo_{ano}.zip"
    try:
        dbutils.fs.ls(arquivo_path)
    except:
        return False, f"Arquivo não encontrado: {arquivo_path}"
    
    # 2. Tabela de destino existe?
    if not spark.catalog.tableExists(f"bronze.{fonte}"):
        return False, f"Tabela bronze.{fonte} não existe"
    
    # 3. Tabela de controle existe?
    if not spark.catalog.tableExists("bronze.controle_ingestao"):
        return False, "Tabela de controle não existe"
    
    # 4. Não está sendo processado por outro job?
    em_processamento = spark.sql(f"""
        SELECT COUNT(*) as cnt
        FROM bronze.controle_ingestao
        WHERE fonte = '{fonte}' AND ano = {ano}
          AND status = 'processando'
          AND data_inicio > current_timestamp() - INTERVAL 2 HOURS
    """).first().cnt
    
    if em_processamento > 0:
        return False, f"Ano {ano} já está sendo processado por outro job"
    
    return True, None

# Uso
valido, erro = validar_prerequisitos('dfp_dre', 2023)
if not valido:
    raise RuntimeError(f"Pré-requisito não atendido: {erro}")
```

**Ganho**: Falha RÁPIDA e CLARA antes de gastar tempo/recursos.

### 3.6 Aplicação em Notebooks (Resiliência sem Quebrar Estrutura)

**Por que "try numa célula, except em outra" não funciona**:

Cada célula de um notebook Databricks é compilada e executada **isoladamente**. Um bloco `try:` aberto em uma célula sem o `except`/`finally` correspondente na MESMA célula é `SyntaxError`.

**Padrão correto**: Funções por responsabilidade + 1 célula de orquestração resiliente

```python
# Célula 6 — EXTRAÇÃO (função dedicada)
def extrair(ano):
    return ler_dados(ano)
```

```python
# Célula 7 — TRANSFORMAÇÃO (função dedicada)
def transformar_dados(df_raw):
    return transformar(df_raw)
```

```python
# Célula 8 — GRAVAÇÃO (função dedicada)
def gravar_dados(df):
    gravar(df)
```

```python
# Célula 9 — ORQUESTRAÇÃO RESILIENTE (única célula com o loop e o try/except)
anos_sucesso, anos_falha = [], []
for ano in ANOS_PROCESSAR:
    try:
        logger.info(f"[EXTRAÇÃO] Ano {ano}")
        df_raw = extrair(ano)

        logger.info(f"[TRANSFORMAÇÃO] Ano {ano}")
        df_bronze = transformar_dados(df_raw)

        logger.info(f"[GRAVAÇÃO] Ano {ano}")
        gravar_dados(df_bronze)
        anos_sucesso.append(ano)
    except Exception as e:
        anos_falha.append((ano, str(e)))
        logger.error(f"[FALHA] {ano}: {e}")
```

**Regra**: Try/except e loop SEMPRE numa única célula (a de orquestração). As funções chamadas por ele ficam separadas por responsabilidade.

---

## 4. Validações de Qualidade

### 4.1 Schema Validation

```python
def validar_schema(df, schema_esperado):
    """
    Valida que DataFrame tem o schema esperado.
    
    Args:
        df: DataFrame a validar
        schema_esperado: Dict {"coluna": "tipo"}
    """
    schema_atual = {field.name: str(field.dataType) for field in df.schema.fields}
    
    colunas_faltantes = set(schema_esperado.keys()) - set(schema_atual.keys())
    colunas_extras = set(schema_atual.keys()) - set(schema_esperado.keys())
    tipos_incorretos = {
        col: (schema_esperado[col], schema_atual[col])
        for col in schema_esperado.keys() & schema_atual.keys()
        if schema_esperado[col] != schema_atual[col]
    }
    
    erros = []
    if colunas_faltantes:
        erros.append(f"Colunas faltantes: {colunas_faltantes}")
    if colunas_extras:
        erros.append(f"Colunas extras: {colunas_extras}")
    if tipos_incorretos:
        erros.append(f"Tipos incorretos: {tipos_incorretos}")
    
    if erros:
        raise ValueError(f"Schema inválido: {'; '.join(erros)}")
    
    logger.info("✅ Schema validado com sucesso")
```

### 4.2 Completude (Nulos)

```python
from pyspark.sql.functions import col, count, when

def validar_completude(df, colunas_obrigatorias, max_percent_nulos=0.05):
    """
    Valida que colunas obrigatórias têm <= max_percent_nulos.
    """
    total_registros = df.count()
    
    for coluna in colunas_obrigatorias:
        nulos = df.filter(col(coluna).isNull()).count()
        percent_nulos = nulos / total_registros if total_registros > 0 else 0
        
        if percent_nulos > max_percent_nulos:
            raise ValueError(
                f"Coluna '{coluna}' tem {percent_nulos:.1%} nulos "
                f"(máximo permitido: {max_percent_nulos:.1%})"
            )
    
    logger.info(f"✅ Completude validada para {len(colunas_obrigatorias)} colunas")
```

### 4.3 Freshness (Atualidade)

```python
from datetime import datetime, timedelta

def validar_freshness(tabela, coluna_timestamp, max_idade_horas=24):
    """
    Valida que última atualização da tabela está dentro da janela esperada.
    """
    ultima_atualizacao = spark.sql(f"""
        SELECT MAX({coluna_timestamp}) as max_ts
        FROM {tabela}
    """).first().max_ts
    
    if ultima_atualizacao is None:
        raise ValueError(f"Tabela {tabela} está vazia")
    
    idade = datetime.now() - ultima_atualizacao
    max_idade = timedelta(hours=max_idade_horas)
    
    if idade > max_idade:
        raise ValueError(
            f"Dados desatualizados: última atualização há {idade.total_seconds()/3600:.1f}h "
            f"(máximo permitido: {max_idade_horas}h)"
        )
    
    logger.info(f"✅ Freshness OK: última atualização há {idade.total_seconds()/3600:.1f}h")
```

---

## 5. Integridade Estrutural

### 5.1 Unicidade de Chaves

```python
def validar_unicidade(df, colunas_chave):
    """
    Valida que combinação de colunas forma chave única.
    """
    total_registros = df.count()
    total_unicos = df.select(colunas_chave).distinct().count()
    
    duplicatas = total_registros - total_unicos
    
    if duplicatas > 0:
        # Mostrar exemplos de duplicatas
        df_duplicatas = (
            df.groupBy(colunas_chave)
            .count()
            .filter("count > 1")
            .orderBy(col("count").desc())
            .limit(5)
        )
        
        raise ValueError(
            f"Encontradas {duplicatas} duplicatas em {colunas_chave}. "
            f"Exemplos:\n{df_duplicatas.toPandas().to_string()}"
        )
    
    logger.info(f"✅ Unicidade validada: {total_registros} registros únicos em {colunas_chave}")
```

### 5.2 Cardinalidade de Joins

```python
def validar_join_1_to_1(df_left, df_right, chave_join, nome_join=""):
    """
    Valida que join é 1:1 (não duplica registros).
    """
    count_antes = df_left.count()
    df_joined = df_left.join(df_right, chave_join, "inner")
    count_depois = df_joined.count()
    
    if count_depois != count_antes:
        raise ValueError(
            f"Join {nome_join} alterou cardinalidade: {count_antes} → {count_depois} "
            f"(esperado 1:1). Verifique duplicatas na tabela direita."
        )
    
    logger.info(f"✅ Join {nome_join} preservou cardinalidade: {count_antes} registros")
    return df_joined
```

---

## 6. Matriz: Guardrail × Camada

Qual guardrail é **crítico** em qual camada:

| Guardrail | Bronze | Silver | Gold |
|-----------|--------|--------|------|
| **Reconciliação - Igualdade de Soma** | ⭐ CRÍTICO | ⭐ CRÍTICO | ⭐ CRÍTICO |
| **Reconciliação - Igualdade de Contagem** | ⭐ CRÍTICO | ⭐ CRÍTICO | Opcional |
| **Reconciliação - Completude Referencial** | Opcional | ⭐ CRÍTICO | Opcional |
| **Retry Logic** | ⭐ CRÍTICO (APIs)* | Raro | Não aplicável |
| **Tratamento Granular de Erros** | ⭐ CRÍTICO | ⭐ CRÍTICO | ⭐ CRÍTICO |
| **Checkpointing** | ⭐ CRÍTICO | Opcional | Não aplicável |
| **Logging Estruturado** | ⭐ CRÍTICO | ⭐ CRÍTICO | ⭐ CRÍTICO |
| **Validação de Pré-requisitos** | ⭐ CRÍTICO | Opcional | Não aplicável |
| **Schema Validation** | Opcional | ⭐ CRÍTICO | ⭐ CRÍTICO |
| **Validação de Completude** | Raro | ⭐ CRÍTICO | ⭐ CRÍTICO |
| **Validação de Freshness** | Opcional | Opcional | ⭐ CRÍTICO |
| **Unicidade de Chaves** | Raro | ⭐ CRÍTICO | ⭐ CRÍTICO |
| **Cardinalidade de Joins** | Não aplicável | ⭐ CRÍTICO | Opcional |

**Legenda**:
* ⭐ **CRÍTICO**: Guardrail essencial nesta camada
* **Opcional**: Aplicar se contexto exigir
* **Raro**: Situações específicas
* **Não aplicável**: Não faz sentido nesta camada

*Nota: Retry Logic em Bronze é crítico para APIs externas/fontes instáveis, mas N/A para arquivos de órgãos reguladores já baixados localmente. Ver seção 6 para diferenciação por tipo de fonte.

---

## 7. Padrões de Implementação

### 7.1 Onde Colocar Guardrails no Código

**Estrutura de célula em notebooks**:

```python
# Célula N — EXTRAÇÃO
def extrair_dados(ano):
    # 1. Validação de pré-requisitos
    validar_prerequisitos('fonte', ano)
    
    # 2. Retry para chamadas externas
    df = retry_com_backoff(lambda: ler_api(ano))
    
    # 3. Logging
    logger.info(f"Extraídos {df.count()} registros para ano {ano}")
    
    return df
```

```python
# Célula N+1 — TRANSFORMAÇÃO
def transformar_dados(df):
    # 1. Schema validation
    validar_schema(df, SCHEMA_ESPERADO)
    
    # 2. Transformação
    df_transformado = df.filter(...).select(...)
    
    # 3. Reconciliação (igualdade de contagem)
    assert df.count() == df_transformado.count()
    
    return df_transformado
```

```python
# Célula N+2 — GRAVAÇÃO
def gravar_dados(df, tabela):
    # 1. Reconciliação antes de gravar
    total_gravar = df.agg(sum("valor")).collect()[0][0]
    logger.info(f"Gravando total={total_gravar}")
    
    # 2. Gravação
    df.write.mode("append").saveAsTable(tabela)
    
    # 3. Validação pós-gravação
    total_tabela = spark.table(tabela).agg(sum("valor")).collect()[0][0]
    assert abs(total_tabela - total_gravar) < 0.01
    
    logger.info(f"✅ Gravação concluída: {df.count()} registros")
```

```python
# Célula N+3 — ORQUESTRAÇÃO RESILIENTE
anos_sucesso, anos_falha = [], []
for ano in ANOS_PROCESSAR:
    try:
        df_raw = extrair_dados(ano)
        df_transformado = transformar_dados(df_raw)
        gravar_dados(df_transformado, "bronze.tabela")
        anos_sucesso.append(ano)
    except Exception as e:
        anos_falha.append((ano, str(e)))
        logger.error(f"❌ Ano {ano} falhou: {e}")

# Relatório final
print(f"📊 {len(anos_sucesso)} sucessos, {len(anos_falha)} falhas")
```

### 7.2 Trade-offs (Performance vs Segurança)

| Guardrail | Custo | Quando Pular |
|-----------|-------|-------------|
| Reconciliação de soma/contagem | Baixo (1 scan) | **NUNCA** em dados financeiros |
| Completude referencial (set comparison) | Alto (collect) | Datasets gigantes (usar amostra) |
| Schema validation | Baixíssimo | **NUNCA** |
| Unicidade (distinct + count) | Médio (shuffle) | Se já há constraint PK na tabela |
| Cardinalidade de joins | Médio (2 counts) | Se join é sabidamente 1:1 por design |

**Regra de Ouro**: Em dados financeiros/transacionais, reconciliação quantitativa é **não-negociável**. O custo de 1 scan adicional é irrelevante comparado ao custo de errar valores.

---

## 8. Antipadrões

| Antipadrão | Consequência | Solução |
|------------|--------------|----------|
| Try/except global sem granularidade | Falha em 1 item destrói todo trabalho | Try/except por unidade de trabalho |
| Retry sem backoff | Thundering herd, piora rate limit | Exponential backoff + jitter |
| Processar sem validar pré-requisitos | Falha no meio, desperdício | Validar ANTES de iniciar |
| Hardcoding de listas de períodos | Não detecta pendentes | Tabela de controle + detecção inteligente |
| Sem logging estruturado | Impossível diagnosticar | Logger com timestamp, contexto, status |
| Sem reconciliação em dados financeiros | Valores errados em produção | **SEMPRE** reconciliar soma/contagem |
| Reconciliação só no final | Descobre erro tarde demais | Reconciliar a cada transição de camada |
| Sem checkpointing | Reprocessa tudo sempre | State management em tabela Delta |

---

## 9. Checklist de Guardrails (Validação de Código)

Antes de considerar um pipeline production-grade:

### Bronze (Ingestão)
- [ ] **Retry logic** para chamadas HTTP/APIs (exponential backoff + jitter) — APENAS se fonte externa/API. N/A para arquivos locais/Landing Zone.
- [ ] **Tratamento granular**: Try/except por período/arquivo
- [ ] **Checkpointing**: Tabela de controle registra estado de cada unidade
- [ ] **Validação de pré-requisitos**: Valida arquivos/tabelas ANTES de processar — crítico para APIs, opcional para órgãos reguladores.
- [ ] **Logging estruturado**: Timestamp, contexto, status, duração
- [ ] **Reconciliação**: Contagem de registros extraídos vs gravados — crítico para APIs, observabilidade para fontes confiáveis.
- [ ] **Detecção inteligente**: Identifica automaticamente períodos pendentes
- [ ] **Schema validation**: Detecta mudanças na estrutura da fonte — crítico para órgãos reguladores, opcional para APIs controladas.

### Silver (Transformação)
- [ ] **Schema validation**: Valida schema antes de transformar
- [ ] **Unicidade**: Valida chaves primárias
- [ ] **Cardinalidade de joins**: Valida que joins não duplicam
- [ ] **Reconciliação - Soma**: Valores totais preservados nas transformações
- [ ] **Reconciliação - Contagem**: Nº registros antes/depois explicável
- [ ] **Completude referencial**: Chaves estrangeiras existem
- [ ] **Tratamento de nulos**: Colunas críticas sem nulos
- [ ] **Logging estruturado**: Cada transformação loga contexto

### Gold (Agregação/Negócio)
- [ ] **Reconciliação - Soma**: Agregações preservam totais esperados
- [ ] **Validação de completude**: Dados não têm gaps temporais
- [ ] **Validação de freshness**: Última atualização dentro da janela
- [ ] **Schema validation**: Output atende contrato esperado
- [ ] **Logging estruturado**: Regras de negócio aplicadas logadas

### Todos (Transversal)
- [ ] **Parametrização**: Janela temporal/configurações vêm de config externa
- [ ] **Relatório de falhas**: Lista explicitamente o que falhou ao final
- [ ] **Documentação**: Guardrails aplicados documentados no notebook

**NOTA**: Idempotência NÃO é um guardrail - é uma propriedade arquitetural que depende da estratégia de gravação escolhida. Ver skill `medallion-architecture` para detalhes sobre como implementar idempotência em cada camada.

---

## 10. Referências

Esta skill consolida práticas de:
* Resiliência operacional (retry, checkpointing, erro granular)
* Integridade de dados (reconciliação, unicidade, cardinalidade)
* Qualidade de dados (schema, completude, freshness)
* Observabilidade (logging estruturado, auditoria)

Para contexto de arquitetura (estratégias de gravação, idempotência), consulte outras skills de arquitetura se disponíveis.
