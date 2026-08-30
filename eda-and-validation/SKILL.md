---
name: eda-and-validation
description: Use ao organizar, estruturar, planejar ou definir o ciclo de descoberta e verificacao em projetos de dados -- QUANDO criar EDA vs validacao, ONDE colocar (estrutura de pastas dedicadas para investigacoes vs validacoes), COMO registrar o fluxo completo (ACHADO -> DECISAO -> CODIGO -> VALIDACAO), COMO conectar etapas no arquivo de evolucao do projeto, COMO lidar com investigacoes inconclusivas. Time Travel e mecanismo auxiliar de auditoria, NAO evidencia principal (que e o output do notebook commitado). NAO use para definir nomenclatura de notebooks EDA/validacao (naming-conventions cobre), implementar codigo de guardrails (data-quality-guardrails cobre), definir estrutura interna de notebooks (notebook-structure cobre), ou escrever documentacao (technical-writing cobre).

---

# Ciclo EDA-Validacao: Organizacao da Descoberta e Verificacao

**Versão:** 1.0.1 | **Data:** 2026-08-18 | **Domínio:** data-quality | **Autor:** Pedro O. Silva

## Quando Usar Esta Skill

**USAR** ao organizar o ciclo de descoberta e verificacao em projetos de dados:
- Usuario quer estruturar pastas de analises exploratorias vs validacoes
- Usuario pergunta onde registrar achados de investigacao
- Usuario quer conectar descobertas com decisoes tecnicas implementadas
- Usuario esta desenhando fluxo de trabalho para projetos de portfolio
- Usuario precisa decidir quando criar notebook de EDA vs validacao

**NAO USAR** para:
- Definir nomenclatura de notebooks EDA/validacao (usar `naming-conventions`)
- Implementar codigo de guardrails ou validacoes (usar `data-quality-guardrails`)
- Definir estrutura interna de notebooks (usar `notebook-structure`)
- Escrever documentacao tecnica (usar `technical-writing`)
- Decisoes arquiteturais de pipeline (usar `medallion-architecture`)

## Principio Fundamental

**EDA nao e camada de pipeline — e superficie de investigacao do pipeline.**

Bronze/Silver/Gold sao camadas de **transformacao progressiva**.
EDA e camada de **inteligencia sobre as transformacoes**.

```
MODELO ERRADO (EDA como camada de dados):
Bronze → Silver → Gold → EDA → BI

MODELO CORRETO (EDA como investigacao):
                ┌─────────────┐
                │     EDA     │ ← investiga qualquer ponto
                │ (ortogonal) │
                └─────────────┘
                       │
                   descobre
                       ↓
Bronze → Silver → Gold → BI
   ↑        ↑       ↑
   └────────┴───────┘
     implementa decisoes
```

## Distincao Critica: EDA vs Validacao

### EDA (Exploratory Data Analysis)

**Proposito**: Descoberta

**Caracteristicas**:
- Investigacao que **GERA** decisoes tecnicas
- Snapshot historico de um estado especifico do pipeline
- **Pode quebrar** depois que o pipeline evolui (comportamento normal)
- Registro do contexto que motivou uma mudanca

**Ciclo de vida**:
1. Criada para investigar problema ou hipotese
2. Produz evidencias e achados
3. Gera decisao tecnica
4. Torna-se registro historico (pode ficar desatualizada)

**Exemplo**:
```
EDA_002_duplicidades_dre.py
├─ Dataset analisado: 201_dre_silver v1 (10M linhas)
├─ Data: 2026-08-16
├─ Achado: 12,4% das chaves possuem duplicidade
├─ Hipotese: ORDEM_EXERC mantem registros conflitantes
├─ Evidencia: [consultas SQL, graficos, exemplos]
└─ Decisao: filtrar ORDEM_EXERC = 'ULTIMO' na transformacao Silver
```

### Validacao

**Proposito**: Verificacao

**Caracteristicas**:
- **VERIFICA** que decisao tecnica foi implementada corretamente
- Executavel permanente sobre estado atual do pipeline
- **Deve passar sempre** (se quebrar = regressao ou nova condicao)
- Teste de conformidade com regras de negocio

**Ciclo de vida**:
1. Criada apos implementacao de uma regra
2. Verifica continuamente que regra funciona
3. Integra-se ao processo de qualidade
4. Pode evoluir para guardrail automatico no pipeline

**Exemplo**:
```
VAL_002_deduplicacao_dre.py
├─ Valida: duplicidade CNPJ+DT_REFER+CD_CONTA = 0
├─ Origem: implementacao da decisao do EDA_002
├─ Resultado esperado: 0 duplicatas
├─ Status: PASS
└─ Se falhar: alerta de regressao ou nova condicao a investigar
```

## Ciclo Completo: ACHADO → DECISAO → CODIGO → VALIDACAO

### Etapa 1: ACHADO (EDA)

**Onde**: Pasta dedicada para investigacoes (ex: `investigacoes/`, `eda/`, `analises_exploratorias/`), nomenclatura tipo `EDA_nnn_descricao`

**O que registrar no notebook**:
```markdown
## Contexto
Dataset analisado: [catalog.schema.table]
Data: [YYYY-MM-DD]
Estado: [quantidade de registros, versao]

## Investigacao
[Pergunta ou problema que motivou a analise]

## Resultado
[Metricas, graficos, exemplos concretos]

## Hipotese
[Explicacao proposta para o achado]

## Decisao
[O que sera implementado no pipeline]
```

### Etapa 2: DECISAO (evolucao_projeto.md)

**Onde**: `evolucao_projeto.md` do projeto

**O que registrar**:
```markdown
## YYYY-MM-DD — [Titulo da Decisao]

**Contexto**: EDA_nnn revelou [achado]
**Decisao**: [O que sera implementado]
**Implementado em**: [arquivo/linha onde foi aplicado]
**Impacto**: [volumetria, qualidade, ou outro efeito mensuravel]
```

### Etapa 3: CODIGO (Pipeline)

**Onde**: Notebook de transformacao (Bronze/Silver/Gold)

**O que fazer**:
- Implementar a regra decidida
- Adicionar comentario referenciando a origem da decisao
- Exemplo:
```python
# Decisao: EDA_002 (2026-08-16) - manter apenas ORDEM_EXERC='ULTIMO'
df_filtrado = df.filter(col("ORDEM_EXERC") == "ULTIMO")
```

### Etapa 4: VALIDACAO (Verificacao)

**Onde**: `05_validacoes/VAL_nnn_descricao.py`

**O que registrar no notebook**:
```markdown
## Validacao
Regra validada: [descricao da regra]
Origem: [referencia ao EDA que gerou a decisao]

## Resultado Esperado
[Metrica ou condicao que deve ser atendida]

## Resultado Obtido
[Resultado da execucao atual]

## Status
PASS | FAIL
```

**Estrutura do codigo**:
```python
# Executar validacao
resultado = spark.sql("""
    SELECT COUNT(*) as duplicatas
    FROM catalog.schema.table
    GROUP BY chave_1, chave_2
    HAVING COUNT(*) > 1
""").collect()[0]['duplicatas']

# Avaliar
status = "PASS" if resultado == 0 else "FAIL"
print(f"Status: {status}")
print(f"Duplicatas encontradas: {resultado}")

if status == "FAIL":
    raise ValueError(f"Validacao falhou: {resultado} duplicatas encontradas")
```

## Validacao Hierarquica: Dois Niveis de Rigor

### Contexto

Dados hierarquicos (DRE, BPA, BPP, plano de contas) possuem relacionamentos estruturais entre contas pai e contas filhas. Validacoes hierarquicas devem distinguir dois tipos de relacionamento, cada um exigindo estrategia de validacao diferente.

### Classificacao de Relacionamentos Hierarquicos

#### Tipo 1: Contas Aditivas (Soma Vertical)

**Definicao**: Conta pai = soma das contas filhas

**Exemplo**:
```
3.01 (Receita)              = 100.000
  ├─ 3.01.01 (Receita Produto A) = 60.000
  └─ 3.01.02 (Receita Produto B) = 40.000

Validacao: 3.01 = SUM(3.01.01, 3.01.02) ✓
```

**Validacao**: Soma matematica (vertical) das filhas deve igualar o pai

#### Tipo 2: Contas Derivadas (Formula Horizontal)

**Definicao**: Conta = formula entre contas irmas (mesmo nivel hierarquico)

**Exemplo**:
```
3.01 (Receita)              = 100.000
3.02 (Custo)                = -60.000
3.03 (Resultado Bruto)      =  40.000

Validacao: 3.03 = 3.01 - 3.02 ✓
```

**Validacao**: Formula matematica (horizontal) entre contas do mesmo nivel

### Dois Niveis de Rigor na Validacao

#### Nivel 1: Interpretacao (Qualitativa)

**Proposito**: Compreender a estrutura e propor hipoteses

**Metodo**:
- Observar padroes na estrutura hierarquica
- Consultar documentacao de dominio (ex: estrutura padrao de DRE)
- Propor formulas plausíveis baseadas em conhecimento do dominio

**Limitacao**: Nao prova que os **valores reais** seguem a interpretacao proposta

**Registro no notebook**:
```markdown
## Classificacao de Contas Totalizadoras

| Tipo | Quantidade | Interpretacao |
| --- | --- | --- |
| Aditivas | 6 | Soma das filhas (estrutura observada) |
| Derivadas | 6 | Formulas interpretadas (nao validadas) |

**Observacao**: Contas derivadas baseiam-se em estrutura padrao de DRE.
Validacao numerica pendente.
```

#### Nivel 2: Verificacao Numerica (Quantitativa)

**Proposito**: Provar com dados reais que a hipotese esta correta

**Metodo**:
- Para contas aditivas: SQL testando `pai = SUM(filhas)`, divergencia < tolerancia
- Para contas derivadas: SQL testando formula proposta, divergencia < tolerancia

**Validacao concreta**: Executar sobre amostra representativa e medir divergencias

**Exemplo SQL (contas aditivas)**:
```sql
WITH pai AS (
  SELECT VL_CONTA as vl_pai
  FROM tabela
  WHERE CD_CONTA = '3.01'
),
filhas AS (
  SELECT SUM(VL_CONTA) as vl_soma_filhas
  FROM tabela
  WHERE CD_CONTA LIKE '3.01.%'
    AND LENGTH(CD_CONTA) - LENGTH(REPLACE(CD_CONTA, '.', '')) + 1 = 3
)
SELECT 
  vl_pai,
  vl_soma_filhas,
  ABS(vl_pai - vl_soma_filhas) as divergencia,
  CASE WHEN ABS(vl_pai - vl_soma_filhas) < 0.01 THEN 'PASS' ELSE 'FAIL' END as status
FROM pai, filhas
```

**Exemplo SQL (contas derivadas)**:
```sql
WITH valores AS (
  SELECT 
    MAX(CASE WHEN CD_CONTA = '3.01' THEN VL_CONTA END) as receita,
    MAX(CASE WHEN CD_CONTA = '3.02' THEN VL_CONTA END) as custo,
    MAX(CASE WHEN CD_CONTA = '3.03' THEN VL_CONTA END) as resultado_bruto_real
  FROM tabela
)
SELECT
  receita,
  custo,
  resultado_bruto_real,
  (receita - custo) as resultado_bruto_calculado,
  ABS(resultado_bruto_real - (receita - custo)) as divergencia,
  CASE WHEN ABS(resultado_bruto_real - (receita - custo)) < 0.01 THEN 'PASS' ELSE 'FAIL' END as status
FROM valores
```

**Registro no notebook**:
```markdown
## Validacao Numerica de Contas Derivadas

### Amostra: WLM, Q4/2025, PENULTIMO

| Conta | Formula Testada | Status | Divergencia |
| --- | --- | --- | --- |
| 3.03 | 3.01 - 3.02 | PASS | < 0,01 |
| 3.05 | 3.03 + 3.04 | PASS | < 0,01 |
| 3.07 | 3.05 + 3.06 | PASS | < 0,01 |

**Conclusao**: Formulas derivadas validadas numericamente na amostra.
```

### Transparencia de Escopo nos Achados

**Problema comum**: Numeros contraditorios sem explicacao de escopo

**Exemplo**:
```markdown
❌ CONFUSO:
"A tabela tem 14 contas totalizadoras."
[10 celulas depois]
"As 12 contas totalizadoras foram validadas."
```

**Solucao**: Explicitar escopo sempre que o numero mudar

```markdown
✓ CLARO:
"A tabela tem 14 contas totalizadoras (escopo: dataset completo, todas as empresas)."
[10 celulas depois]
"Das 14 contas totalizadoras globais, 12 aparecem no reporte da empresa WLM (escopo: amostra especifica).
As 12 contas da amostra foram validadas numericamente."
```

**Principio**: Cada metrica deve identificar o escopo que a gerou

### Documentacao de Gaps Pendentes

**Principio**: "Ainda nao verificado" e informacao tao importante quanto "validado"

**Registro no notebook**:
```markdown
## Status da Validacao Hierarquica

### Validado
- [x] 6 contas aditivas: validadas por SQL, divergencia < 0,01
- [x] Escopo: amostra WLM, Q4/2025, PENULTIMO

### Pendente
- [ ] 6 contas derivadas: interpretadas, mas nao validadas numericamente
- [ ] Validacao: criar celula SQL testando formulas propostas
- [ ] Generalizacao: validar em amostra maior (outras empresas/periodos)

**Proxima acao**: Adicionar celula SQL para validacao numerica de contas derivadas.
```

**Por que documentar gaps**:
- Demonstra honestidade intelectual
- Facilita continuidade do trabalho
- Evita interpretar "interpretacao" como "validacao"
- Recrutadores valorizam rigor tecnico e transparencia

### Checklist de Validacao Hierarquica

Ao validar estruturas hierarquicas em EDA:

1. [ ] Classificar relacionamentos (aditivas vs derivadas)
2. [ ] Nivel 1: Interpretar estrutura (documentar como hipotese)
3. [ ] Nivel 2: Validar numericamente (SQL sobre amostra)
4. [ ] Explicitar escopo de cada metrica reportada
5. [ ] Documentar gaps pendentes explicitamente
6. [ ] Registrar tolerancia de divergencia aceitavel (ex: < 0,01)
7. [ ] Amostras pequenas: suficientes para validacao, explicitar limitacao

### Integracao com Ciclo EDA-Validacao

**EDA hierarquico**:
- Investiga estrutura hierarquica
- Classifica relacionamentos (aditivas/derivadas)
- Valida numericamente em amostra
- Documenta achados e gaps pendentes

**Validacao hierarquica**:
- Verifica integridade hierarquica em escopo completo
- Testa continuamente pos-implementacao
- Pode evoluir para guardrail automatico

**Exemplo de progressao**:
```
EDA_001_hierarquia_dre
    ↓ descobre e valida em amostra
Decisao: sem correcao necessaria, estrutura integra
    ↓ validacao continua opcional
VAL_001_integridade_hierarquica_dre
    ↓ testa em todo dataset periodicamente
```

## Estrutura de Pastas

### Estrutura Recomendada

```
projeto/
├── 01_bronze/
├── 02_silver/
├── 03_gold/
├── 04_investigacoes/          ← DESCOBERTA (pode ficar desatualizada)
│   ├── EDA_001_qualidade_dados_bronze.py
│   ├── EDA_002_duplicidades_dre.py
│   └── EDA_003_estrutura_contas_plano.py
├── 05_validacoes/             ← VERIFICACAO (deve passar sempre)
│   ├── VAL_001_integridade_bronze.py
│   ├── VAL_002_deduplicacao_dre.py
│   └── VAL_003_conformidade_contas.py
└── 00_documentacao/
    └── evolucao_projeto.md    ← CONTEXTO (por que foi feito)
```

### Criterio de Classificacao

**Pergunta**: Este notebook investiga um problema/hipotese OU verifica uma regra implementada?

**Resposta "investiga"** → `04_investigacoes/`
- Analise exploratoria de dados brutos
- Investigacao de anomalias
- Teste de hipoteses sobre estrutura/qualidade
- Descoberta de padroes

**Resposta "verifica"** → `05_validacoes/`
- Testa se regra de negocio esta implementada
- Verifica se transformacao produziu resultado esperado
- Confirma que guardrail esta funcionando
- Mede conformidade com requisitos

## Investigacoes Inconclusivas

**Situacao**: EDA que nao gera decisao tecnica (hipotese refutada, exploracao sem achado relevante).

**O que fazer**:

1. **Manter o notebook** em `04_investigacoes/`
2. **Registrar a conclusao** no proprio notebook:
```markdown
## Conclusao
Hipotese refutada: [explicacao]
Nenhuma acao necessaria no pipeline.
Investigacao arquivada para referencia futura.
```

3. **Documentar em evolucao_projeto.md**:
```markdown
## YYYY-MM-DD — Investigacao: [Titulo]

**Contexto**: [O que motivou a investigacao]
**Resultado**: Hipotese refutada / Nenhuma evidencia encontrada
**Decisao**: Nenhuma alteracao no pipeline necessaria
**Aprendizado**: [O que foi descoberto sobre os dados, mesmo sem acao]
```

**Por que manter**: Demonstra rigor cientifico e evita re-investigar a mesma hipotese no futuro.

## Progressao: Validacao → Guardrail Automatico

Quando uma validacao se torna:
- Repetitiva (executada frequentemente)
- Estavel (sempre passa, regra consolidada)
- Critica (falha representa problema grave)

**Considere transformar em guardrail automatico** no codigo do pipeline.

**Consultar**: Skill `data-quality-guardrails` para implementacao.

**Exemplo de progressao**:
```
EDA_002 (investigacao)
    ↓ gera decisao
VAL_002 (validacao manual)
    ↓ executa 20+ vezes, sempre passa
Guardrail automatico no codigo Silver
    ↓ integrado ao pipeline
    - Valida automaticamente a cada execucao
    - Falha rapido se condicao violada
    - Documentado inline no codigo
```

## Time Travel: Mecanismo Auxiliar, Nao Fundamento

### Hierarquia de Evidencias

**1. Evidencia Primaria**: Output do notebook (consultas, graficos, metricas) commitado no Git
**2. Evidencia Secundaria**: Registro em evolucao_projeto.md
**3. Evidencia Auxiliar**: Time Travel para auditoria pontual

### Time Travel E Adequado Para

- **Auditoria regulatoria**: recuperar estado especifico para compliance
- **Debug de regressao**: "quando esta tabela quebrou?"
- **Comparacao antes/depois**: medir impacto de mudanca estrutural
- **Recuperacao**: reverter para versao anterior

### Time Travel NAO Deve Ser Usado Para

- Definir estrutura da camada de analises
- Tornar EDAs permanentemente reproduziveis sobre tabelas vivas
- Criar dependencias `VERSION AS OF 17, 18, 21...` nos notebooks
- Fundamentar a arquitetura do ciclo EDA-validacao

### Uso Correto em Notebook de EDA

**BOM** (contexto historico):
```python
# Estado analisado em 2026-08-16
# Dataset: catalog.schema.table
# Registros: 10.234.567
# Versao Delta: informacao registrada para auditoria futura
```

**RUIM** (dependencia operacional):
```python
# Este notebook REQUER VERSION AS OF 17 para funcionar
df = spark.sql("SELECT * FROM catalog.schema.table VERSION AS OF 17")
```

## Nomenclatura

Para convencoes de nomenclatura de notebooks EDA/validacao, consultar skill `naming-conventions`.

**Padroes gerais**:
- Prefixo: `EDA_` ou `VAL_`
- Numeracao sequencial: `001`, `002`, `003`
- Descricao: snake_case, especifica
- Exemplos:
  - `EDA_002_duplicidades_dre.py`
  - `VAL_002_deduplicacao_dre.py`

## Estrutura Interna dos Notebooks

Para ordem de celulas e organizacao interna de cada notebook EDA/validacao, consultar skill `notebook-structure`.

**Principios gerais**:
- Celula inicial: documentacao (contexto, objetivo)
- Celulas intermediarias: analise/validacao
- Celula final: conclusao/status

## Conexao com evolucao_projeto.md

### Formato Recomendado

Cada entrada documenta o ciclo completo:

```markdown
## YYYY-MM-DD — [Titulo da Decisao]

**Contexto**: EDA_nnn revelou [achado especifico com metricas]

**Decisao**: [O que sera implementado - regra clara]

**Implementado em**: [arquivo/linha - rastreabilidade]

**Validado por**: VAL_nnn - [resultado da validacao]

**Impacto**: [volumetria, qualidade, ou outro efeito mensuravel]

**Aprendizado**: [Insight sobre os dados ou processo]
```

### Rastreabilidade Completa

O registro cria cadeia rastreavel:
- **EDA_nnn** aponta para **evolucao_projeto.md** (entrada de data especifica)
- **evolucao_projeto.md** aponta para **codigo de transformacao** (arquivo/linha)
- **VAL_nnn** aponta para **evolucao_projeto.md** (mesma entrada)
- **Git** preserva todas as versoes historicas

## Valor Para Portfolio

Este ciclo demonstra para recrutadores:

1. **Pensamento estruturado**: distincao clara entre investigacao vs verificacao
2. **Rastreabilidade tecnica**: cada decisao nasce de evidencia, cada implementacao gera validacao
3. **Maturidade profissional**: ciclo completo de engenharia, nao apenas "codigo que funciona"
4. **Documentacao tecnica**: conexao entre descoberta, decisao, implementacao e verificacao
5. **Rigor cientifico**: investigacoes inconclusivas tambem sao documentadas

## Checklist de Aplicacao

Ao receber solicitacao relacionada ao ciclo EDA-validacao:

1. [ ] Notebook proposto e investigacao (EDA) ou verificacao (validacao)?
2. [ ] Pasta correta identificada (`04_investigacoes/` vs `05_validacoes/`)?
3. [ ] Ciclo completo esta sendo considerado (ACHADO → DECISAO → CODIGO → VALIDACAO)?
4. [ ] Conexao com evolucao_projeto.md foi planejada?
5. [ ] Investigacoes inconclusivas tem tratamento adequado?
6. [ ] Time Travel esta sendo usado como auxiliar (nao fundamento)?
7. [ ] Referencias cruzadas com outras skills foram feitas quando necessario?

## Integracao com Outras Skills

- **naming-conventions**: Convencoes de nomenclatura EDA_nnn vs VAL_nnn
- **notebook-structure**: Ordem de celulas e organizacao interna
- **data-quality-guardrails**: Progressao de validacao para guardrail automatico
- **technical-writing**: Tom e formato de documentacao em notebooks e evolucao_projeto.md
- **docs-sync**: Quais documentacoes atualizar apos decisao tecnica
