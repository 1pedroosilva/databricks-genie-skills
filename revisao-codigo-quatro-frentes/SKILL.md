---
name: revisao-codigo-quatro-frentes
description: "Use APENAS ao REVISAR/AUDITAR corretude de código existente em 4 frentes -- (1) CORRECAO SEMANTICA: bugs de lógica, alucinação de API, divergência comentário/código; (2) PREMISSAS OCULTAS: unicidade não validada, cardinalidade de join assumida, nulos não tratados, não-determinismo, atomicidade não garantida; (3) CODIGO MORTO: passos redundantes, abstractions prematuras; (4) CUSTO EVITAVEL: quebra de lazy evaluation, UDFs evitáveis, shuffles desnecessários. NÃO use para definir nomenclatura, estrutura de células, implementar padrões ou decisões arquiteturais -- use outras skills para isso."

---

# Revisão de Código — Quatro Frentes

**Versão:** 1.0.0 | **Data:** 2026-08-14 | **Autor:** Pedro O. Silva

## Propósito

Esta skill é acionada sempre que a Genie Code for chamada para **revisar ou validar
notebook/código** — antes de um commit, antes de promover um notebook para produção,
ou quando explicitamente pedido "revisa esse notebook". Ela não é sobre estrutura de
célula, documentação, nomenclatura nem guardrails de schema — se o projeto tiver
instruções próprias sobre esses assuntos, elas têm prioridade e esta skill não as
sobrepõe. Esta skill é sobre se o código **faz o que deveria, com as premissas que ele
realmente pode sustentar, sem peso morto e sem custo evitável**.

## Modo de operação — sugerir, nunca aplicar sozinho

Esta skill produz um **relatório de revisão e, quando fizer sentido, um plano de
mudanças proposto** — ela nunca edita o notebook/arquivo por conta própria como parte da
revisão. Revisão de código sobre pipelines de dados é sensível por natureza: um achado de
PREMISSAS OCULTAS quase sempre tem efeito cascata em camadas downstream, e aplicar a
correção sem o usuário decidir pode alterar dado já consumido por outra coisa.

- Para cada achado, apresente: arquivo/célula, trecho atual, mudança proposta (trecho
 corrigido), frente/severidade, e o impacto de aplicar aquilo (o que muda a jusante, se
 algum histórico precisa ser reprocessado, se algum consumidor downstream depende do
 comportamento atual mesmo que ele esteja errado).
- Ordene o plano pela hierarquia de prioridade das quatro frentes (abaixo) — não pela
 ordem em que os achados foram encontrados no código.
- Só edite arquivos se o usuário aprovar explicitamente **quais** achados do plano quer
 aplicar — nunca aplique "todos os achados" implicitamente, e nunca aplique um achado
 que não estava no plano apresentado.
- Se o usuário pedir a revisão mas não disser nada sobre aplicar mudanças, o output
 esperado é só o relatório/plano — aplicar é uma ação separada, subsequente, que precisa
 de pedido explícito.

## Ordem de prioridade — e por que ela importa

1. **CORREÇÃO** — o código faz o que deveria?
2. **PREMISSAS OCULTAS** — sobre o que o código apoiou isso sem verificar?
3. **NECESSIDADE** — isso precisa existir?
4. **CUSTO** — quanto isso consome sem necessidade?

As duas primeiras **invalidam o notebook inteiro** — não importa quão eficiente ou
enxuto seja o código, se ele produz o resultado errado ou se apoia numa premissa falsa
sem avisar, o resultado é lixo silencioso. As duas últimas só **pioram** um código que
já está correto. Nunca reporte um achado de CUSTO como se tivesse a mesma gravidade de
um achado de CORREÇÃO ou PREMISSA — a ordem da lista de achados de uma revisão deve
seguir esta hierarquia.

## Metodologia — antes de reportar um achado

Uma suspeita não é um achado. Antes de listar algo como problema:

- **Se há dado real disponível (tabela, workspace, execução anterior), verifique contra
 ele.** Uma hipótese de bug baseada só em leitura de código pode estar errada — o
 jeito de descobrir é consultar o estado real (`SELECT` na tabela, `DESCRIBE`,
 histórico de execuções), não presumir. Uma hipótese descartada por evidência real vale
 tanto quanto um achado confirmado: registre as duas coisas separadamente ("o que
 verifiquei e estava correto" vs. "o que confirmei que está quebrado").
- **Prefira o workspace/tabela ao vivo a qualquer cópia local ou memória de sessões
 anteriores.** Notebooks Databricks mudam rápido; código exportado ontem pode já estar
 desatualizado.
- **Para cada achado, aponte o trecho exato, a frente, o que está errado e por quê** —
 não basta dizer "isso pode ser um problema", diga qual é a premissa, o dado ou a
 chamada que está errada e como isso se manifestaria.
- **Ao revisar múltiplos notebooks do mesmo projeto**, separe explicitamente os
 problemas que se repetem em ≥2 notebooks (viram padrão) dos que aparecem uma vez só
 (viram nota, não regra geral do projeto).
- **Se o notebook tem um "irmão"** (mesma estrutura, aplicada a uma fonte/tabela
 diferente — o caso clássico é um notebook novo criado copiando um existente), faça
 diff contra o irmão antes de aceitar o novo como correto. É assim que nasce o bug mais
 comum de 1.3: uma constante ou nome que deveria ter sido criado/renomeado durante a
 cópia e não foi — revisar o notebook novo isolado, sem comparar contra o original,
 deixa esse tipo de erro passar despercebido.

---

## FRENTE 1 — CORREÇÃO

### 1.1 Semântica
A transformação entrega o resultado pretendido, não só um resultado plausível que passa
despercebido. Pergunte: se eu calculasse isso manualmente com uma calculadora e uma
amostra pequena, bateria com o que o código produz?

### 1.2 Lógica
- Filtros, joins e agregações na ordem certa (filtrar antes de agregar quando possível;
 um filtro aplicado depois de uma agregação pode filtrar o resultado errado).
- Condição de janela (`Window.partitionBy`/`orderBy`) e granularidade da agregação
 coerentes com o grain que o resultado final promete ter.
- `distinct()` ou `dropDuplicates()` aplicado achando que remove duplicatas de negócio,
 quando na verdade só remove linhas byte-a-byte idênticas.

### 1.3 Alucinação de API
Função, parâmetro ou assinatura que não existe na versão real da biblioteca em uso —
verifique contra a API real (documentação da versão específica, ou teste direto), nunca
contra o que "parece razoável" ou contra a API de uma biblioteca vizinha.

**O caso mais comum: parâmetros de pandas vazando para chamadas PySpark (ou
vice-versa).**

```python
# ERRADO — PySpark dropDuplicates() não tem parâmetro `keep`.
# Isso é assinatura de pandas.DataFrame.drop_duplicates(), não de PySpark.
df_dedup = df.dropDuplicates(subset=["cnpj", "data"], keep="last")
```

```python
# CORRETO — em PySpark, "manter o mais recente" se resolve com Window,
# não com um parâmetro que não existe.
window_spec = Window.partitionBy("cnpj", "data").orderBy(col("versao").desc())
df_dedup = (
 df.withColumn("_rn", row_number().over(window_spec))
 .filter(col("_rn") == 1)
 .drop("_rn")
)
```

Outros pontos comuns de alucinação a checar: `spark.read.csv(..., dtype=...)` (parâmetro
de pandas, não existe em `spark.read.csv`), uso de `sc.parallelize()`/`.rdd.*` em
compute serverless/Spark Connect (API RDD não suportada nesse modo — checar o tipo de
compute antes de aceitar código que usa RDD), assinatura de `.persist()` com argumentos
inventados, nomes de função que mudaram entre major versions do Spark sem o código ter
sido atualizado.

**Padrão já confirmado em auditoria real, não é hipotético**: uma função de
transformação chamava uma constante de "colunas essenciais" (usada para validar/projetar
schema) que nunca tinha sido definida em lugar nenhum do projeto — nem no arquivo de
configuração compartilhado, nem no próprio notebook. `NameError` puro. O agravante: esse
`NameError` era capturado pelo `try/except` por-unidade-de-trabalho do próprio notebook
(a resiliência operacional dele), então o notebook terminava "com sucesso" — sem exceção
não tratada — enquanto, na verdade, gravava zero linhas em todas as execuções, desde a
criação do notebook. Só ficou visível consultando a tabela de controle de execuções ao
vivo, que mostrava 100% de falha silenciosa para aquela fonte específica.

```python
# ERRADO — nome nunca definido em lugar nenhum do projeto
df_validado = validar_e_projetar_schema(df_raw, COLUNAS_ESSENCIAIS_X, f"X {periodo}")
```

```python
# CORRETO — a lista precisa existir antes de ser referenciada
COLUNAS_ESSENCIAIS_X = ["chave_negocio", "data_referencia", "codigo_item", "valor"]
df_validado = validar_e_projetar_schema(df_raw, COLUNAS_ESSENCIAIS_X, f"X {periodo}")
```

**Lição geral desse caso**: um `try/except` bem construído para isolar falhas por
unidade de trabalho (ano, arquivo, partição) é correto como *mecanismo* de resiliência —
mas ele também é capaz de mascarar um erro incondicional (que sempre vai acontecer, em
toda execução, para toda unidade) transformando-o em "falha silenciosa e repetida" em
vez de "erro óbvio na primeira execução". Ao revisar código com resiliência
operacional, sempre pergunte: **esse except está isolando uma falha ocasional
legítima, ou está mascarando uma falha determinística que deveria ter quebrado o
notebook na cara?**

### 1.4 Escrita SQL via f-string sem tratar `NULL`/aspas
Interpolar uma variável Python direto num literal SQL sem checar `None` nem escapar
aspas é ao mesmo tempo um risco de correção (o valor `None` vira a string literal
`'None'`, não `NULL`) e de injeção (se o valor puder conter aspas simples).

```python
# ERRADO — se data_evento for None, o INSERT grava a string 'None'
# numa coluna TIMESTAMP em vez de NULL; se algum valor tiver aspas, quebra a query.
spark.sql(f"""
 INSERT INTO controle (periodo, data_evento) VALUES ({periodo}, '{data_evento}')
""")
```

```python
# CORRETO — parametrizar via DataFrame + spark.sql com parâmetros nomeados
# (Databricks SQL suporta `:param` desde DBR recente), ou construir o valor
# explicitamente como NULL quando None.
valor_sql = "NULL" if data_evento is None else f"'{data_evento}'"
spark.sql(f"INSERT INTO controle (periodo, data_evento) VALUES ({periodo}, {valor_sql})")

# Melhor ainda: usar spark.sql com parameter markers, que trata None/aspas sozinho.
spark.sql(
 "INSERT INTO controle (periodo, data_evento) VALUES (:periodo, :data)",
 args={"periodo": periodo, "data": data_evento},
)
```

### 1.5 Divergência comentário↔código
Comentário ou docstring afirma um comportamento que o código não implementa de fato —
comum em código gerado por IA, onde o comentário é gerado junto com uma primeira versão
da lógica e sobra desatualizado depois de um ajuste, ou descreve a intenção em vez do
que o código realmente faz.

```python
# ERRADO — o comentário promete um tratamento que o código não tem
# Remove duplicados preservando nulos com segurança
df_limpo = df.dropDuplicates(["chave"])
# dropDuplicates não trata nulo de forma especial; se "chave" for nula,
# a linha nula só participa do dedup normal, sem tratamento à parte.
```

```python
# CORRETO — comentário e código dizem a mesma coisa
df_limpo = df.filter(col("chave").isNotNull()).dropDuplicates(["chave"])
# Remove linhas com chave nula antes de deduplicar; nulos tratados à parte.
```

Trate cada comentário como uma afirmação testável: o código realmente faz o que ele diz?
Se não, o comentário é a mentira, não a fonte de verdade — corrija o código ou o
comentário, mas nunca deixe os dois divergirem silenciosamente.

### 1.6 Segurança e dados sensíveis
Credencial, token, string de conexão ou chave de API hardcoded no código-fonte; dado
pessoal (CPF, CNPJ, e-mail, nome completo) impresso em log/print/display sem
necessidade; SQL montado por concatenação de string a partir de entrada que veio de fora
do controle do próprio pipeline (não só o caso de `None`/aspas de 1.4, mas o caso geral
de injeção).

```python
# ERRADO — credencial hardcoded, visível a qualquer um com acesso de
# leitura ao workspace/repositório; PII exposta em log de execução.
token = "dapi1234567890abcdef"
print(f"Processando CPF {linha['cpf']} - Nome: {linha['nome_completo']}")
```

```python
# CORRETO — credencial via Secrets, nunca hardcoded; log sem PII.
token = dbutils.secrets.get(scope="producao", key="api_token")
print(f"Processando registro {linha['id_hash']}")
```

---

## FRENTE 2 — PREMISSAS OCULTAS

A frente mais importante: o código roda mesmo quando a premissa é falsa, e nada avisa.
Para cada premissa encontrada, diga qual é a premissa e como ela poderia ser verificada
— não basta apontar, tem que dar o caminho de checagem.

### 2.1 Unicidade de chave
`dropDuplicates()` sem `subset` explícito remove duplicatas byte-a-byte, não duplicatas
de negócio. Um `join` que assume 1:1 sem checar (`df.groupBy(chave).count().filter(count
> 1)` do lado direito) pode estar silenciosamente multiplicando linhas.

### 2.2 Cardinalidade do join
Se o join é 1:N e o código foi escrito assumindo 1:1, a contagem de linhas muda e nada
avisa — a query roda, retorna um número de linhas plausível, e ninguém percebe até uma
métrica de negócio vier inflada. Verificação: comparar `df.count()` antes e depois do
join contra o esperado pela cardinalidade declarada.

### 2.3 Granularidade da saída — grain declarado vs. grain real
O grain que a tabela/DataFrame promete ter (via nome, comentário, ou chave de partição
de uma window function) precisa bater com o grain que ela realmente entrega.

**Padrão já confirmado em auditoria real, não é hipotético**: o `Window.partitionBy`
usado para "pegar a versão mais recente de cada registro" incluía uma coluna de
dimensão de período (ex.: período atual vs. período comparativo de uma mesma entidade)
como parte da chave natural:

```python
# ERRADO — a coluna de dimensão de período na partição faz cada valor dela
# sobreviver como linha independente para a mesma chave_negocio+data_referencia+item.
window_spec = Window.partitionBy(
 "chave_negocio", "data_referencia", "codigo_item", "tipo_periodo"
).orderBy(col("_versao_ingestao").desc())
```

Confirmado ao vivo: para a mesma entidade e data de referência, o mesmo número exato de
linhas em cada valor de `tipo_periodo` (um espelhamento 1:1 inteiramente esperado dado o
`partitionBy`, mas não documentado como comportamento pretendido em lugar nenhum).
Qualquer soma de `valor` por chave_negocio+data_referencia+item sem filtrar por um valor
específico de `tipo_periodo` conta cada item em dobro.

```python
# CORRETO, se a intenção é "uma linha por item, sem duplicar o período
# comparativo" — decidir explicitamente se a dimensão de período pertence
# à chave de dedup ou se deve ser filtrada antes dela.
df_periodo_atual = df_bronze.filter(col("tipo_periodo") == "ATUAL")
window_spec = Window.partitionBy(
 "chave_negocio", "data_referencia", "codigo_item"
).orderBy(col("_versao_ingestao").desc())
```

Ao revisar qualquer `Window.partitionBy`, liste as colunas da partição e pergunte: "uma
linha por combinação dessas colunas — é isso mesmo que a tabela promete ser?" Se a
resposta não for óbvia a partir do nome da tabela/comentário, é uma premissa oculta.

### 2.4 Status de sucesso desacoplado de volume real
Um passo que grava `status = 'SUCCESS'` (ou equivalente) sem checar se algo de fato foi
processado (`count() > 0`, arquivo não vazio, linhas afetadas) está confundindo "o
código rodou sem lançar exceção" com "o trabalho foi feito".

**Padrão já confirmado em auditoria real, não é hipotético**: uma fonte de uma camada
anterior nunca recebeu dados por causa de um erro incondicional mascarado (ver 1.3), e a
etapa seguinte do pipeline lia um DataFrame vazio, fazia `DELETE` (deletava nada) e
`APPEND` de 0 linhas — e mesmo assim registrava `SUCCESS` na tabela de controle, para
múltiplos períodos seguidos. Nada na cadeia de camadas jamais checou se o volume
processado era compatível com o esperado.

```python
# ERRADO — grava SUCCESS incondicionalmente
df_silver.write.format("delta").mode("append").saveAsTable(tabela_destino)
registrar_controle_sucesso(ano, ...) # sem checar quantas linhas foram gravadas
```

```python
# CORRETO — valida volume antes de declarar sucesso
count_registros = df_silver.count()
if count_registros == 0:
 raise ValueError(
 f"Ano {ano}: 0 registros produzidos para {tabela_destino} — "
 f"não gravar SUCCESS sobre um resultado vazio inesperado."
 )
df_silver.write.format("delta").mode("append").saveAsTable(tabela_destino)
registrar_controle_sucesso(ano, count_registros, ...)
```

Isso não significa que todo resultado vazio é erro (uma fonte pode legitimamente não ter
dados num período) — significa que a decisão precisa ser **explícita**, não um
subproduto acidental de nada ter quebrado.

### 2.5 Nulos
Coluna usada em `join`, `filter` ou agregação sem tratamento de nulo. `NULL` em SQL não
casa com `NULL` em join (`a.chave = b.chave` descarta silenciosamente as duas linhas
quando `chave` é nula dos dois lados) — se isso for esperado, tudo bem, mas precisa ser
uma decisão, não um acidente.

### 2.6 Tipo e domínio
Cast implícito, comparação entre tipos diferentes, valor esperado dentro de faixa sem
checagem.

**Padrão já confirmado em auditoria real, não é hipotético**: um notebook Bronze com
princípio documentado de "manter a estrutura original da fonte, sem transformação" lia o
CSV de origem via `pd.read_csv(...)` sem `dtype=str`, deixando o pandas inferir tipo
automaticamente — confirmado ao vivo: uma coluna de valor numérico terminou gravada como
string com um sufixo decimal artificial (`.0`) que não existia no arquivo original,
porque o pandas inferiu `float64` e a escrita Delta converteu de volta para string na
hora do append. A premissa ("Bronze preserva o dado bruto") foi violada silenciosamente
por uma inferência de tipo que ninguém pediu.

```python
# ERRADO — deixa o pandas inferir tipo (arrisca alterar a representação
# original de colunas numéricas/textuais antes mesmo de chegar na Bronze)
df_pandas = pd.read_csv(csv_file, sep=";", encoding="ISO-8859-1")
```

```python
# CORRETO — se o contrato da camada é "preservar o bruto", force tudo como
# string na leitura e deixe o cast de tipo explícito acontecer só na Silver.
df_pandas = pd.read_csv(csv_file, sep=";", encoding="ISO-8859-1", dtype=str)
```

### 2.7 Acoplamento implícito via estado global (`%run`, variável de módulo)
Código que depende de um efeito colateral (uma função que faz `global X` dentro de um
arquivo carregado via `%run`) em vez de capturar o valor de retorno explicitamente é uma
premissa oculta sobre a forma como os notebooks são encadeados — funciona hoje porque
`%run` injeta tudo no mesmo namespace, mas quebra silenciosamente se esse arquivo virar
um `import` de módulo algum dia.

```python
# ERRADO — depende do efeito colateral de `global PERIODOS_PROCESSAR` dentro
# da função, chamada via %run no mesmo namespace.
if PERIODOS_PROCESSAR is None:
 inicializar_periodos_processar()
```

```python
# CORRETO — captura o retorno explicitamente; funciona independente de
# %run vs import.
if PERIODOS_PROCESSAR is None:
 PERIODOS_PROCESSAR = inicializar_periodos_processar()
```

### 2.8 Atomicidade assumida em operações multi-passo
`DELETE` seguido de `APPEND` (ou qualquer sequência de duas operações Delta separadas
que juntas formam "uma substituição") não é atômico — se o processo cair entre as duas,
a partição fica parcialmente vazia até o próximo run, sem sinalização de que algo ficou
pela metade.

```python
# ERRADO — duas transações Delta separadas; se cair no meio, a partição
# do ano fica sem dado até o próximo run, sem nenhum alerta.
spark.sql(f"DELETE FROM tabela WHERE ANO = {ano}")
df_silver.write.format("delta").mode("append").saveAsTable("tabela")
```

```python
# CORRETO — replaceWhere faz a substituição como uma única transação
# Delta atômica, para tabelas particionadas pela mesma coluna do filtro.
df_silver.write.format("delta") \
 .option("replaceWhere", f"ANO = {ano}") \
 .mode("overwrite") \
 .saveAsTable("tabela")
```

### 2.9 Classificação de erro por exceção genérica
Um `except Exception:` amplo que trata qualquer falha como um único caso de negócio
(ex.: "arquivo não existe", "primeira execução") conflate causas completamente
diferentes — um timeout de rede, um erro de parsing e um 404 real viram a mesma
resposta, e o motivo verdadeiro nunca aparece no log.

```python
# ERRADO — timeout de rede, erro de parsing e 404 real são todos tratados
# como "arquivo não existe".
try:
 req = urllib.request.Request(url, method="HEAD")
 with urllib.request.urlopen(req, timeout=10) as response:
 ...
except Exception:
 return (False, None, None)
```

```python
# CORRETO — diferencia o que é esperado (404) do que é uma falha real
# que merece aparecer no log.
try:
 req = urllib.request.Request(url, method="HEAD")
 with urllib.request.urlopen(req, timeout=10) as response:
 ...
except urllib.error.HTTPError as e:
 if e.code == 404:
 return (False, None, None)
 raise
except urllib.error.URLError as e:
 logger.warning(f"Falha de rede ao verificar {url}: {e}")
 raise
```

### 2.10 Não-determinismo em ordenação sem critério de desempate
`orderBy`/`sort` usado numa window function ou numa seleção de "o registro mais
recente" sem uma coluna de desempate que garanta unicidade — se houver empate, a escolha
de `row_number() == 1` (ou `first()`/`limit(1)`) entre as linhas empatadas não é
garantidamente estável entre execuções. Não dá erro nenhum: só produz um resultado
diferente num reprocessamento do que produziu da primeira vez.

```python
# ERRADO — se duas linhas empatarem em "versao", a escolha entre elas
# não é garantidamente estável entre execuções.
window_spec = Window.partitionBy("chave").orderBy(col("versao").desc())
```

```python
# CORRETO — acrescenta uma coluna de desempate que garanta unicidade
# (timestamp de ingestão com precisão suficiente, ou um id monotônico)
window_spec = Window.partitionBy("chave").orderBy(
 col("versao").desc(), col("_ingest_ts").desc(), col("_id").desc()
)
```

### 2.11 Entrada externa não validada em fronteira de confiança
Parâmetro vindo de widget, variável de ambiente, arquivo de configuração externo ou
resposta de API é usado direto (`int(...)`, indexação, `.split(...)`) sem tratar o caso
de vir vazio, malformado ou ausente — comum em código júnior, e também em código gerado
por IA quando só o "caminho feliz" foi exercitado.

```python
# ERRADO — widget vazio, com espaço extra, ou com valor não numérico
# quebra com uma exceção genérica e pouco informativa.
anos = [int(a.strip()) for a in dbutils.widgets.get("anos_override").split(",")]
```

```python
# CORRETO — valida a entrada explicitamente antes de usar
valor_bruto = dbutils.widgets.get("anos_override").strip()
if not valor_bruto:
 anos = []
else:
 try:
 anos = [int(a.strip()) for a in valor_bruto.split(",") if a.strip()]
 except ValueError as e:
 raise ValueError(f"Widget 'anos_override' malformado: '{valor_bruto}'") from e
```

### 2.12 Timezone e determinismo temporal
Comparação entre timestamps de origens diferentes (horário de sessão Spark,
`current_timestamp()`, header HTTP de uma fonte externa, timestamp de arquivo) sem
garantir que estão no mesmo fuso horário. `current_timestamp()` usa o timezone da sessão
Spark (`spark.sql.session.timeZone`), que pode não ser UTC nem o mesmo fuso da fonte
externa.

```python
# ERRADO — compara um datetime vindo de uma fonte externa (geralmente
# UTC) com um timestamp local sem garantir o mesmo fuso.
if timestamp_fonte > timestamp_local:
 ...
```

```python
# CORRETO — normaliza os dois lados para UTC explicitamente antes de comparar
from datetime import timezone
timestamp_fonte_utc = timestamp_fonte.astimezone(timezone.utc)
timestamp_local_utc = timestamp_local.astimezone(timezone.utc)
if timestamp_fonte_utc > timestamp_local_utc:
 ...
```

---

## FRENTE 3 — NECESSIDADE

Frente de subtração: aponte o que deve **sair**, não o que falta.

- **Passo redundante ou resultado recalculado**: a mesma transformação computada mais de
 uma vez no fluxo (sem cache) quando bastava computar uma vez e reaproveitar.
- **Abstração prematura**: função, classe ou bloco de configuração criado para algo
 usado uma única vez, num único lugar, sem indício de que vai ganhar um segundo
 consumidor.
- **Parâmetro morto**: argumento de função que existe na assinatura mas nunca é
 referenciado no corpo — sinal de que a função foi generalizada para um caso que nunca
 chegou a existir. Já apareceu em auditoria real como achado isolado (uma função
 recebia um segundo parâmetro de "tipo" que nunca era lido no corpo) — trate como nota
 a verificar, não como padrão automático de todo projeto.
- **Reimplementação do que já existe nativo ou no próprio projeto**: função escrita do
 zero para algo que uma função Spark/Databricks nativa já faz (ex.: reimplementar
 dedup manual em vez de `dropDuplicates`, ou reimplementar merge incremental em vez de
 `MERGE INTO`/`replaceWhere`).
- **Persistência ou cache sem reuso que justifique**: `.cache()`/`.persist()` num
 DataFrame usado uma única vez depois — custo de memória sem benefício, porque não há
 segunda leitura para amortizar.

---

## FRENTE 4 — CUSTO

- **Quebra de lazy evaluation por vaidade**: `count()`, `collect()`, `toPandas()` ou
 `display()` no meio do fluxo só para conferir/logar, sem `.cache()`/`.persist()` antes
 — isso força o Spark a computar a DAG inteira, e a ação seguinte (geralmente um
 `.write()`) recomputa tudo de novo do zero.

 **Padrão já confirmado em auditoria real, não é hipotético** — apareceu de forma
 quase idêntica em múltiplos notebooks do mesmo projeto:
 ```python
 # ERRADO — computa a DAG inteira para imprimir uma contagem, e o
 # .write() logo depois recomputa a mesma DAG do zero.
 print(f"DataFrame Bronze criado: {df_bronze.count():,} registros")
 df_bronze.write.format("delta").mode("append").saveAsTable(tabela)
 ```
 ```python
 # CORRETO — cacheia antes de disparar a primeira ação, se o count()
 # realmente precisa existir; ou melhor, tira o count() do caminho quente
 # e usa métricas do próprio commit Delta (df_bronze.write... e depois
 # ler numOutputRows do log de operação) se só serve para log.
 df_bronze = df_bronze.cache()
 print(f"DataFrame Bronze criado: {df_bronze.count():,} registros")
 df_bronze.write.format("delta").mode("append").saveAsTable(tabela)
 df_bronze.unpersist()
 ```

- **UDF Python onde existe função nativa do Spark**: UDFs quebram a otimização do
 Catalyst e forçam serialização linha a linha entre JVM e Python. Antes de aceitar uma
 UDF, pergunte se `pyspark.sql.functions` já cobre o caso.

- **`withColumn` em loop**: cada chamada de `withColumn` adiciona um nó ao plano lógico;
 um loop Python chamando `withColumn` repetidamente para N colunas infla o plano de
 forma desnecessária.

 ```python
 # ERRADO — N chamadas de withColumn em loop, N nós no plano lógico
 for c in colunas_numericas:
 df = df.withColumn(c, F.col(c).cast("double"))
 ```
 ```python
 # CORRETO — um único select reescreve todas as colunas de uma vez
 df = df.select(
 *[F.col(c).cast("double").alias(c) if c in colunas_numericas else F.col(c)
 for c in df.columns]
 )
 ```

- **Shuffle evitável**: `repartition()` sem necessidade antes de um `write` numa tabela
 já particionada pela mesma coluna; `join` sem broadcast quando um dos lados é
 claramente pequeno (tabela de dimensão, lookup, referência).

 ```python
 # ERRADO — join grande x pequeno sem broadcast, shuffle nos dois lados
 df_fatos.join(df_dimensao_pequena, "chave")
 ```
 ```python
 # CORRETO
 from pyspark.sql.functions import broadcast
 df_fatos.join(broadcast(df_dimensao_pequena), "chave")
 ```

- **Leitura sem projeção ou sem filtro empurrado para a fonte**: materializar
 (`collect()`/`toPandas()`) uma tabela inteira para filtrar em Python quando o filtro
 podia ser expresso em Spark e teria pushdown para a fonte (partition pruning numa
 tabela particionada, predicate pushdown em Parquet/Delta).

---

## Fora de escopo desta skill

- Estrutura de célula e separação de responsabilidades por célula.
- Nomenclatura de notebooks, tabelas e variáveis.
- Retry/backoff, logging estruturado, checkpointing como padrão operacional em si (esta
 skill entra nesse assunto só quando a resiliência mascara um erro — ver 1.3).
- Documentação e protocolo de atualização de docs.
- Guardrails de schema (validação de colunas obrigatórias) em si — esta skill assume que
 o guardrail existe e revisa o que acontece *depois* dele.

Se o projeto tiver documentação própria cobrindo esses pontos, ela tem prioridade sobre
qualquer suposição desta skill.
