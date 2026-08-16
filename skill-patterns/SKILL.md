---
name: skill-patterns
description: Use APENAS ao CRIAR skills novas para Databricks Genie Code. Define estrutura obrigatoria (YAML frontmatter + Markdown), padroes de description (verbo de acao unico, boundaries negativos explicitos, < 1024 chars), separacao de responsabilidades e checklist de qualidade. NAO use para editar skills existentes (use readAssetById primeiro), criar outros tipos de assets (notebooks, tabelas, arquivos), ou responder perguntas sobre como skills funcionam (esse conteudo ja esta carregado).

---

# Padrões para Criação de Skills

**Versão:** 1.2.0 | **Data:** 2026-08-16 | **Autor:** Pedro O. Silva

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
# Metadados oficiais (obrigatorios pela plataforma)
name: skill-name-kebab-case
description: Use APENAS ao [VERBO DE ACAO] [contexto especifico]. [O que a skill cobre]. NAO use para [boundaries negativos explicitos].

# Metadados de distribuicao (versionamento interno)
version: 1.0.0
updated: YYYY-MM-DD
---
```

**Regras para `name`:**
- Kebab-case (minusculas, hifens)
- Descritivo e unico
- Alinhado com o verbo de acao

**Regras para `description`:**
- **< 1024 caracteres** (limite tecnico)
- **ASCII puro** (evitar caracteres acentuados para compatibilidade)
- **Verbo de acao UNICO** por skill (ver tabela abaixo)
- **Boundaries negativos explicitos** ("NAO use para...")
- **Especificidade maxima** sobre quando triggar vs quando nao triggar
- Formato: `Use APENAS ao [VERBO]. [Contexto]. NAO use para [boundaries].`

### Corpo Markdown

- Secoes claras com headings H2/H3
- Instrucoes ao agente, nao ao usuario final
- Exemplos concretos (comandos, codigo, expected output)
- Paragrafos curtos e acionaveis
- Sem emojis ou icones

## Verbos de Acao e Separacao de Responsabilidades

Cada skill deve ter um verbo de acao UNICO para prevenir sobreposicao:

| Verbo | Skill Existente | Escopo |
|-------|----------------|--------|
| DEFINIR | nomenclaturas | Convencoes de nomenclatura |
| CRIAR (notebooks) | estrutura-notebooks | Estrutura de notebooks novos |
| CRIAR (UC) | unity-catalog | Schemas, tabelas, volumes UC |
| CRIAR (skills) | skill-patterns | Skills novas do Genie Code |
| IMPLEMENTAR | resiliencia-operacional | Padroes de resiliencia em codigo |
| REVISAR/AUDITAR | revisao-codigo-quatro-frentes | Corretude de codigo existente |
| ATUALIZAR | protocolo-atualizacao | Documentacao apos mudancas |
| COMMITAR | git-workflow | Divisao de commits, staging |
| ESCREVER | padrao-escrita | Conteudo textual e documentacoes |

**Regra de Ouro:** Se duas skills usam o mesmo verbo, devem ter escopo DISJUNTO (ex: CRIAR notebooks vs CRIAR schemas UC vs CRIAR skills).

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
4. [ ] **Verbo de acao unico e nao-conflitante** com skills existentes?
5. [ ] **Description < 1024 chars**?
6. [ ] **Description em ASCII puro** (sem acentos)?
7. [ ] **Boundaries negativos explicitos** na description?
8. [ ] **Formato YAML frontmatter completo** (metadados oficiais + distribuicao com version e updated)?
9. [ ] **Corpo Markdown acionavel** (instrucoes ao agente, exemplos concretos)?
10. [ ] **Especificidade maxima** sobre quando triggar vs nao triggar?
11. [ ] **Separacao clara** de outras skills (sem sobreposicao de escopo)?

## Metricas de Qualidade (Target)

| Metrica | Target |
|---------|--------|
| Description < 1024 chars | 100% |
| ASCII puro | 100% |
| Boundaries negativos explicitos | 100% |
| Verbos de acao unicos | 100% |
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

### Passo 1: Definir Verbo de Acao e Escopo

- Identificar o verbo de acao (CRIAR, DEFINIR, IMPLEMENTAR, REVISAR, ATUALIZAR, etc)
- Verificar se ja existe skill com esse verbo
- Se sim, garantir que escopos sao disjuntos
- Documentar escopo especifico da nova skill

### Passo 2: Escrever Description

- Comecar com `Use APENAS ao [VERBO] [contexto]`
- Adicionar contexto especifico (1-2 frases)
- Adicionar boundaries negativos explicitos (`NAO use para...`)
- Validar tamanho < 1024 chars
- Converter caracteres acentuados para ASCII

### Passo 3: Estruturar Corpo Markdown

- Secao "Quando Usar Esta Skill" (casos de uso positivos e negativos)
- Secoes de orientacao tecnica (procedimentos, padroes, checklist)
- Exemplos concretos (BOM vs RUIM)
- Evitar emojis, icones, tom marketeiro

### Passo 4: Validar com Checklist

- Executar checklist de qualidade (11 itens acima, incluindo verificacao DRY)
- Corrigir falhas antes de criar o arquivo

### Passo 5: Criar Arquivo

- Path: `.assistant/skills/[skill-name]/SKILL.md`
- Usar `createAsset` (assetType: "file")
- Incluir versionamento inicial: `version: 1.0.0` e `updated: [data atual formato YYYY-MM-DD]`
- Nao criar pastas adicionais sem autorizacao

## Exemplo Completo

### SKILL.md de Referencia (skill-patterns)

```yaml
---
# Metadados oficiais (obrigatorios pela plataforma)
name: skill-patterns
description: Use APENAS ao CRIAR skills novas para Databricks Genie Code. Define estrutura obrigatoria (YAML frontmatter + Markdown), padroes de description (verbo de acao unico, boundaries negativos explicitos, < 1024 chars), separacao de responsabilidades e checklist de qualidade. NAO use para editar skills existentes, criar outros tipos de assets, ou responder perguntas sobre como skills funcionam.

# Metadados de distribuicao (versionamento interno)
version: 1.0.0
updated: 2026-08-16
---

# Padroes para Criacao de Skills

[corpo markdown acionavel]
```

**Validacao:**
- Tamanho: 447 chars (< 1024 ✓)
- ASCII puro: ✓
- Boundaries negativos: ✓
- Verbo unico: CRIAR (skills) ✓
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

**Sempre sincronizar** `updated` com a data da nova `version`.

**Exemplo**:
- Skill em `version: 1.2.3`
- Adicionar nova secao de troubleshooting → `version: 1.3.0, updated: 2026-08-20`
- Corrigir typo → `version: 1.2.4, updated: 2026-08-20`
- Mudar verbo de acao (breaking) → `version: 2.0.0, updated: 2026-08-20`

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

**Solucao:** Remover TODOS os comentarios do YAML. Formato limpo:

```yaml
---
name: minha-skill
description: ...
version: 1.0.0
updated: 2026-08-16
---
```

**2. Campos extras ou nao-reconhecidos no YAML**

```yaml
---
name: minha-skill
description: ...
author: Pedro Silva  ❌ (campo nao-reconhecido)
category: data-engineering  ❌ (campo nao-reconhecido)
---
```

**Solucao:** Manter APENAS os 4 campos oficiais: `name`, `description`, `version`, `updated`.

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
2. Campos alem de name/description/version/updated? → Remover
3. Description comeca com verbo de acao? → Reformular
4. Description tem boundaries negativos? → Adicionar

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
- Aplicar checklist de qualidade (11 itens) sem mencionar a skill ao usuario
- Se skill proposta violar padroes OU duplicar skill existente:
  - Corrigir violacoes antes de criar, OU
  - Redirecionar para editar skill existente, OU
  - Orientar usuario a usar skill global em vez de criar user skill
- Usuario deve receber skill ja validada e no padrao correto
