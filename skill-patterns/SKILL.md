---
name: skill-patterns
description: Use ao criar, desenvolver, elaborar ou definir skills novas para Databricks Genie Code. Define estrutura obrigatoria (YAML frontmatter + Markdown + domain), padroes de description (vocabulario rico com sinonimos, boundaries negativos explicitos, contexto detalhado, < 1024 chars ASCII), checklist de qualidade e organizacao por dominios. NAO use para editar skills existentes (use readAssetById primeiro), criar outros tipos de assets (notebooks, tabelas, arquivos), ou responder perguntas sobre como skills funcionam.

---

# Padrões para Criação de Skills

**Versão:** 1.3.2 | **Data:** 2026-08-19 | **Domínio:** workflow | **Autor:** Pedro O. Silva

## Quando Usar Esta Skill

**USAR** ao criar uma skill nova do zero (usuario solicita criacao de skill ou assistente identifica necessidade de nova skill).

**NAO USAR** para:
- Editar skills existentes (carregar a skill existente com readAssetById primeiro)
- Criar notebooks, tabelas, arquivos ou outros assets (usar skills especificas)
- Responder perguntas conceituais sobre skills (esse conteudo ja esta no contexto)

## Estrutura Obrigatoria de SKILL.md

### YAML Frontmatter

Dois campos obrigatorios:

```yaml
---
name: skill-name-kebab-case
description: Use ao [acao/tarefa com sinonimos naturais] [contexto especifico rico]. [O que a skill cobre em detalhe]. NAO use para [boundaries negativos explicitos].
---
```

**CRITICO - Metadados FORA do YAML:** Os campos `version`, `updated`, `domain` e `author` NAO vao no YAML frontmatter. Eles existem APENAS no corpo Markdown, na linha logo apos o titulo:

```markdown
# Titulo da Skill

**Versao:** 1.0.0 | **Data:** YYYY-MM-DD | **Dominio:** [categoria] | **Autor:** Nome
```

**Regras para `name`:**
- Kebab-case (minusculas, hifens)
- Descritivo e unico
- Alinhado com o dominio tematico da skill

**Regras para `description`:**
- **< 1024 caracteres** (limite tecnico)
- **ASCII puro** (design preference, mantido para consistencia visual)
- **Vocabulario rico com sinonimos naturais** - use multiplas palavras-chave para o mesmo conceito (ex: "revisar, validar, auditar, avaliar" em vez de apenas "revisar")
- **Boundaries negativos explicitos** ("NAO use para...")
- **Especificidade maxima** sobre quando triggar vs quando nao triggar
- **Contexto especifico detalhado** - descrever o QUE, QUANDO e ONDE a skill se aplica
- **CRITICO - Sintaxe YAML segura**: NUNCA usar `: ` (dois-pontos seguido de espaco) no meio do texto da description. Em YAML, esse padrao e ambiguo e quebra o parser (interpretado como inicio de nova chave). Use separadores alternativos: `--`, `-`, `->`, `;` ou envolva toda a description em aspas duplas.
- Formato sugerido: `Use ao [acao com sinonimos] [contexto especifico rico]. [Detalhes]. NAO use para [boundaries].`

### Corpo Markdown

- Secoes claras com headings H2/H3
- Instrucoes ao agente, nao ao usuario final
- Exemplos concretos (comandos, codigo, expected output)
- Paragrafos curtos e acionaveis
- Sem emojis ou icones

## Dominios e Separacao de Responsabilidades

Cada skill deve pertencer a um dominio unico para prevenir sobreposicao. Use o campo `domain` para organizar skills tematicamente:

| Domain | Skill Existente | Escopo |
|--------|----------------|--------|
| naming | nomenclaturas | Convencoes de nomenclatura para assets novos |
| code-structure | estrutura-notebooks | Estrutura e organizacao de notebooks novos |
| data-modeling | unity-catalog | Schemas, tabelas, volumes no Unity Catalog |
| workflow | skill-patterns | Criacao e manutencao de skills do Genie Code |
| code-quality | revisao-codigo-quatro-frentes | Revisao de corretude de codigo existente |
| code-quality | guardrails-pipelines | Implementacao de validacoes em pipelines |
| documentation | protocolo-atualizacao | Mapeamento de docs apos mudancas |
| documentation | padrao-escrita | Padroes de escrita tecnica |
| version-control | git-workflow | Commits, staging, mensagens de commit |
| architecture | arquitetura-medalhao | Decisoes estrategicas de pipeline |
| architecture | escolha-sql-pyspark | Decisao entre SQL e PySpark |
| project-management | ciclo-eda-validacao | Organizacao de EDA e validacoes |

**Principio DRY:** Cada dominio pode ter multiplas skills, desde que seus escopos sejam claramente distintos e nao-sobrepostos. Desambiguacao acontece via description rica (vocabulario amplo) + boundaries negativos claros.

## Boundaries Negativos Explicitos

Cada skill deve especificar o que ela **NAO faz** para evitar false-positives:

**Exemplo BOM:**
```
NAO use para revisar ou validar nomenclatura de codigo existente.
```

**Exemplo RUIM:**
```
(sem boundaries negativos)
```

**Nota Tecnica:** Testes empiricos mostraram que boundaries negativos nao sao tecnicamente obrigatorios para triggering correto, mas sao **boas praticas de clareza** para humanos e agentes.

## Checklist de Qualidade para Skill Nova

Antes de criar a skill, validar:

1. [ ] **Verificacao DRY - Skills globais**: Consultou skills globais (readSkillFile) para confirmar que topico nao esta coberto?
2. [ ] **Verificacao DRY - User skills**: Listou user skills existentes (.assistant/skills/) para confirmar que nao ha duplicacao?
3. [ ] **Conformidade oficial**: Usou docSearch para validar contra documentacao oficial da Databricks?
4. [ ] **Description < 1024 chars**?
5. [ ] **Description em ASCII puro** (sem acentos)?
6. [ ] **Description com vocabulario rico** (sinonimos naturais para a tarefa principal)?
7. [ ] **Boundaries negativos explicitos** na description?
8. [ ] **Contexto especifico detalhado** na description (QUE, QUANDO, ONDE)?
9. [ ] **Domain definido** e alinhado com tabela de dominios existentes?
10. [ ] **Formato YAML frontmatter limpo** (apenas name e description, SEM version/updated/domain)?
11. [ ] **Corpo Markdown acionavel** (instrucoes ao agente, exemplos concretos)?
12. [ ] **Separacao clara** de outras skills do mesmo dominio (sem sobreposicao de escopo)?

## Metricas de Qualidade (Target)

| Metrica | Target |
|---------|--------|
| Description < 1024 chars | 100% |
| ASCII puro | 100% |
| Vocabulario rico (sinonimos) | 100% |
| Boundaries negativos explicitos | 100% |
| Domain definido | 100% |
| Triggering correto | 100% |

## Verificacao DRY e Sobreposicao

**CRITICO**: Antes de criar qualquer skill nova, validar que nao ha duplicacao com skills existentes.

### Regra de Decisao

**Consultar skills globais** (usar `readSkillFile`):
- Se skill global JA COBRE o topico completamente → NAO criar user skill, usar a global
- Se skill global cobre PARTE do topico → User skill deve COMPLEMENTAR (adicionar preferencias especificas do usuario), NUNCA REDEFINIR conhecimento da global
- Se skill global NAO EXISTE para o topico → OK criar user skill

**Consultar user skills existentes** (listar `.assistant/skills/`):
- Se user skill JA EXISTE com escopo identico → EDITAR skill existente (usar editAsset), NAO criar nova
- Se user skill tem verbo identico mas escopo disjunto → OK criar, garantir boundaries claros na description
- Se nenhuma user skill cobre o topico → OK criar

**Validar documentacao oficial** (usar `docSearch`):
- Buscar orientacoes oficiais da Databricks sobre o topico
- Garantir que skill segue padroes atualizados (nao obsoletos)
- Se documentacao contradiz conteudo proposto → CORRIGIR antes de criar

### Exemplo de Verificacao Completa

**Usuario pede**: "Criar skill sobre estrutura de notebooks"

**Passo A - Skills globais**:
```
readSkillFile("skills/skill-authoring/SKILL.md")
```
Resultado: Skill global cobre formato YAML + Markdown de skills, NAO cobre estrutura de notebooks → Topico nao duplicado

**Passo B - User skills**:
```python
import os
skills = [d for d in os.listdir("/Users/1pedro.osilva@gmail.com/.assistant/skills/") 
          if os.path.isdir(os.path.join("/Users/1pedro.osilva@gmail.com/.assistant/skills/", d))]
```
Resultado: Encontra `estrutura-notebooks/` JA EXISTE → EDITAR, nao criar nova

**Passo C - Documentacao oficial**:
```
docSearch("Databricks notebook best practices")
```
Resultado: Validar que skill proposta alinha com recomendacoes oficiais

**Decisao final**: EDITAR `estrutura-notebooks`, NAO criar skill nova.

## Procedimento de Criacao

### Passo 1: Definir Domain e Escopo

- Identificar o dominio tematico (naming, code-quality, architecture, etc)
- Verificar se ja existe skill no mesmo dominio
- Se sim, garantir que escopos sao claramente distintos e nao-sobrepostos
- Documentar escopo especifico da nova skill com boundaries negativos
- Adicionar campo `domain` ao frontmatter

### Passo 2: Escrever Description

- Comecar com `Use ao [acao com sinonimos naturais] [contexto especifico rico]`
- Incluir multiplas palavras-chave/sinonimos para a tarefa principal
- Adicionar contexto detalhado: QUE, QUANDO, ONDE (1-3 frases)
- Adicionar boundaries negativos explicitos (`NAO use para...`)
- Validar tamanho < 1024 chars
- Converter caracteres acentuados para ASCII

### Passo 3: Estruturar Corpo Markdown

- Secao "Quando Usar Esta Skill" (casos de uso positivos e negativos)
- Secoes de orientacao tecnica (procedimentos, padroes, checklist)
- Exemplos concretos (BOM vs RUIM)
- Evitar emojis, icones, tom marketeiro

### Passo 4: Validar com Checklist

- Executar checklist de qualidade (13 itens acima, incluindo verificacao DRY)
- Corrigir falhas antes de criar o arquivo

### Passo 5: Criar Arquivo

- Path: `.assistant/skills/[skill-name]/SKILL.md`
- Usar `createAsset` (assetType: "file")
- Incluir metadados na linha apos titulo do corpo: **Versao:** 1.0.0 | **Data:** [data] | **Dominio:** [categoria] | **Autor:** Nome
- Nao criar pastas adicionais sem autorizacao

## Exemplo Completo

### SKILL.md de Referencia (skill-patterns)

```yaml
---
name: skill-patterns
description: Use ao criar, desenvolver, elaborar ou definir skills novas para Databricks Genie Code. Define estrutura obrigatoria (YAML frontmatter + Markdown + domain), padroes de description (vocabulario rico com sinonimos, boundaries negativos explicitos, contexto detalhado, < 1024 chars ASCII), checklist de qualidade e organizacao por dominios. NAO use para editar skills existentes, criar outros tipos de assets, ou responder perguntas sobre como skills funcionam.
---

# Padroes para Criacao de Skills

**Versao:** 1.0.0 | **Data:** 2026-08-16 | **Dominio:** workflow | **Autor:** Pedro O. Silva

[corpo markdown acionavel]
```

**Validacao:**
- Tamanho: 485 chars (< 1024 ✓)
- ASCII puro: ✓
- Vocabulario rico: "CRIAR skills novas" ✓
- Boundaries negativos: ✓
- Domain: workflow ✓
- Versionamento: ✓
- DRY: Complementa skill-authoring global (nao duplica) ✓

## Integracao com Skill de Autoria (skill-authoring)

A skill global `skill-authoring` da Databricks define formato basico e procedimento de criacao.

**Divisao de responsabilidades:**
- `skill-authoring` (global): Formato YAML + Markdown, procedimento generico de criacao
- `skill-patterns` (user skill): Padroes especificos do usuario (verbos unicos, boundaries, checklist de qualidade, metricas)

**Carregar ambas** ao criar skill nova: skill-authoring para formato, skill-patterns para padroes de qualidade.

## Atualizacao de Skills Existentes

Quando atualizar uma skill existente, incrementar `version` seguindo semantic versioning:

**MAJOR (X.0.0)**: Mudanca breaking
- Altera triggering (muda verbo de acao ou escopo principal)
- Remove funcionalidade documentada
- Muda boundaries de forma incompativel

**MINOR (x.Y.0)**: Adiciona funcionalidade
- Novos exemplos ou secoes
- Expansao de escopo compativel (nao breaking)
- Melhora de orientacoes existentes

**PATCH (x.y.Z)**: Correcoes
- Typos ou erros de formatacao
- Clareza de texto (sem mudanca de comportamento)
- Exemplos melhores (mesma funcionalidade)

**Sempre sincronizar** campo **Data:** com a nova **Versao:** na linha de metadados do corpo Markdown.

**Exemplo** (linha **Versao:** no corpo):
- Skill em **Versao:** 1.2.3
- Adicionar nova secao de troubleshooting → **Versao:** 1.3.0 | **Data:** 2026-08-20
- Corrigir typo → **Versao:** 1.2.4 | **Data:** 2026-08-20
- Mudar dominio ou description de forma incompativel (breaking) → **Versao:** 2.0.0 | **Data:** 2026-08-20

Esses campos existem na linha de metadados do corpo Markdown, NAO no YAML frontmatter.

## Troubleshooting: Skills Nao Reconhecidas pelo Registry

### Sintomas

Skill existe em `.assistant/skills/[skill-name]/SKILL.md` mas nao aparece no registry do sistema ao tentar carregar com `readSkillFile`.

### Causas Identificadas

**1. Comentarios no YAML frontmatter**

```yaml
---
# Este comentario quebra o parsing  ❌
name: minha-skill
description: ...
---
```

**Solucao:** Remover TODOS os comentarios do YAML. Formato limpo (apenas name e description):

```yaml
---
name: minha-skill
description: ...
---
```

Metadados de versao vao na linha **Versao:** do corpo Markdown, nao no YAML.

**2. Campos extras ou nao-reconhecidos no YAML**

```yaml
---
name: minha-skill
description: ...
author: Pedro Silva  ❌ (campo nao-reconhecido)
category: data-engineering  ❌ (campo nao-reconhecido)
---
```

**Solucao:** Manter APENAS os 2 campos oficiais: `name`, `description`.

**3. Description sem verbo de acao no inicio**

```yaml
description: Esta skill ajuda na criacao de notebooks. Use quando precisar criar notebooks novos.  ❌
```

**Solucao:** Action-first phrasing:

```yaml
description: Use APENAS ao CRIAR notebooks novos do zero. Define estrutura de celulas iniciais. NAO use para editar notebooks existentes.  ✓
```

**4. Description sem boundaries negativos**

```yaml
description: Use APENAS ao CRIAR notebooks novos do zero. Define estrutura de celulas iniciais.  ⚠️ (falta boundaries)
```

**Solucao:** Adicionar boundaries explicitos:

```yaml
description: Use APENAS ao CRIAR notebooks novos do zero. Define estrutura de celulas iniciais. NAO use para editar notebooks existentes ou revisar estrutura de notebooks prontos.  ✓
```

### Diagnostico Rapido

**Passo 1:** Tentar carregar a skill com `readSkillFile("skills/[skill-name]/SKILL.md")`

**Passo 2:** Se erro ou nao reconhecida, abrir SKILL.md e verificar:
1. Comentarios no YAML? → Remover
2. Campos alem de name/description/version/updated/domain? → Remover
3. Description tem vocabulario rico (sinonimos)? → Enriquecer
4. Description tem boundaries negativos? → Adicionar
5. Description tem contexto especifico detalhado? → Expandir

**Passo 3:** Salvar e tentar carregar novamente. Skill deve aparecer no registry.

### Validacao Empirica Planejada

Experimento a realizar:
1. Escolher skill funcional (ex: nomenclaturas)
2. Adicionar comentario no YAML
3. Verificar se desaparece do registry
4. Remover comentario
5. Confirmar reaparecimento

**Objetivo:** Validar empiricamente que comentarios no YAML sao causa raiz.

---

## Aplicacao

- Carregar esta skill ANTES de criar qualquer skill nova
- SEMPRE executar Verificacao DRY (consultar skills globais, user skills, documentacao oficial)
- Aplicar checklist de qualidade (13 itens) sem mencionar a skill ao usuario
- Se skill proposta violar padroes OU duplicar skill existente:
  - Corrigir violacoes antes de criar, OU
  - Redirecionar para editar skill existente, OU
  - Orientar usuario a usar skill global em vez de criar user skill
- Usuario deve receber skill ja validada e no padrao correto
