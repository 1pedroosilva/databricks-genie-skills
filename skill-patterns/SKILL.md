---
name: skill-patterns
description: Use ao criar, desenvolver, elaborar, estruturar, editar, revisar ou auditar skills (novas ou existentes) para o Genie Code contra o padrao de qualidade aqui definido -- cobre validacao de description (dominio de atuacao, colisao com skills nativas da plataforma como notebooks, Unity Catalog, dashboards, pipelines e MLflow), checklist de qualidade, verificacao de duplicidade (DRY) contra skills globais e locais, e organizacao de conteudo entre SKILL.md e references/. NAO use para criar outros tipos de asset (notebooks, tabelas, jobs, pipelines), ou tirar duvidas conceituais sobre como skills funcionam em geral.
---

# Padroes para Criacao de Skills

**Versao:** 2.1.0 | **Data:** 2026-08-20 | **Dominio:** workflow | **Autor:** Pedro O. Silva

## Quando Usar Esta Skill

**USAR** ao criar, editar ou revisar uma skill (nova ou existente) — usuario pede explicitamente ou o agente identifica a necessidade durante a conversa.

**NAO USAR** para:
- Criar notebooks, tabelas, jobs ou outros assets → usar a skill especifica daquele asset
- Responder perguntas conceituais sobre skills → esse conteudo ja esta no contexto do agente

## Principios (Orientacao)

Estas ideias guiam toda decisao tomada no procedimento abaixo — leia antes de executar.

1. **Cabecalho YAML minimo.** So `name` e `description` sao campos oficialmente obrigatorios. O Genie Code, na pratica observada, e mais rigido que a especificacao geral do formato: campos extras no YAML (mesmo os opcionais previstos pela especificacao, como um mapa de metadados) tendem a tirar a skill do registry. Por isso, quaisquer outros dados (versao, data, dominio, autor) vao no corpo Markdown, nunca no YAML. Ver `references/troubleshooting-registry.md`.
2. **Orientacao separada de execucao.** O corpo da skill deve deixar claro o que e principio (o "porque", estavel, raramente muda) e o que e procedimento (o "como", passo a passo, o agente executa na ordem).
3. **DRY antes de criar.** Nunca criar uma skill sem antes checar se uma skill global ja cobre o topico, se uma user skill ja existente cobre o mesmo escopo, e se o dominio pretendido colide com uma skill nativa da plataforma.
4. **Enxugamento por padrao.** O `SKILL.md` carrega inteiro no contexto assim que a skill e ativada — diferente do `name`/`description`, que ficam sempre carregados para todas as skills. Conteudo extenso (tabelas de referencia, exemplos longos, troubleshooting) deve morar em `references/`, carregado sob demanda apenas quando o agente precisar.

## Fluxo de Decisao (Execucao)

```
PEDIDO DE SKILL NOVA
        |
        v
[1] DRY - skill global ja cobre o topico?
        |-- SIM, cobre tudo --------------> usar a skill global (FIM)
        |-- PARCIAL ------------------------> user skill deve complementar, nunca redefinir
        |-- NAO cobre
        v
[2] DRY - ja existe user skill com escopo identico?
        |-- SIM -----------------------------> EDITAR a existente, nao criar nova (FIM)
        |-- NAO
        v
[3] Validar dominio e colisao com skills nativas
    (ver references/dominios-e-colisoes.md)
        |-- colide -----------------------------> ajustar escopo ou redirecionar (FIM)
        |-- livre
        v
[4] Escrever description
    (rica em sinonimos, ASCII, < 1024 chars, boundaries negativos)
        |
        v
[5] Estruturar corpo: Principios (orientacao) + Procedimento (execucao)
        |
        v
[6] Decidir o que fica em SKILL.md vs references/
        |
        v
[7] Rodar checklist de qualidade
    (ver references/checklist-qualidade.md)
        |
        v
[8] Criar SKILL.md (+ references/ se necessario)
```

## Procedimento Detalhado (Execucao)

### Passo 1-2: Verificacao DRY

Consultar, nesta ordem: skills globais da plataforma, depois as user skills em `.assistant/skills/`. Regra de decisao completa e exemplo passo a passo em `references/exemplo-completo.md`.

### Passo 3: Dominio e Colisao com Skills Nativas

O Genie Code ja vem com skills nativas para notebooks, exploracao no Unity Catalog, dashboards, pipelines e MLflow. Uma skill nova nao pode ocupar o mesmo escopo dessas sem diferenciacao clara. Tabela completa de dominios existentes e criterio de nao-sobreposicao em `references/dominios-e-colisoes.md`.

### Passo 4: Escrever a Description

Formato sugerido:

```
Use ao [acao com sinonimos naturais] [contexto especifico: QUE, QUANDO, ONDE]. [Detalhe do que a skill cobre]. NAO use para [boundaries negativos explicitos].
```

Regras:
- **< 1024 caracteres**
- **ASCII puro** (sem acentos) — preferencia de consistencia visual, nao e regra tecnica do parser
- **Vocabulario rico**: varias palavras para a mesma acao (ex.: "revisar, validar, auditar, avaliar")
- **Boundaries negativos explicitos**: "NAO use para..."
- **Sintaxe YAML segura**: nunca usar `: ` (dois-pontos + espaco) no meio do texto da description — quebra o parser YAML. Usar `--`, `-`, `->` ou `;`.

### Passo 5: Estruturar o Corpo

- `# Titulo`
- Linha de metadados: `**Versao:** x.y.z | **Data:** YYYY-MM-DD | **Dominio:** categoria | **Autor:** Nome`
- `## Quando Usar Esta Skill` (positivo e negativo)
- `## Principios` (orientacao — o porque, estavel)
- `## Procedimento` ou `## Fluxo de Decisao` (execucao — o como, passo a passo)
- Exemplos concretos, sem emojis, sem tom de marketing

### Passo 6: Decidir SKILL.md vs references/

Regra pratica: se uma secao so e consultada em uma minoria dos casos de uso da skill (troubleshooting, tabela de dados auxiliares, exemplo longo, historico de versoes), ela vai para `references/nome-do-topico.md` e o `SKILL.md` so aponta para o arquivo com um link relativo. O que fica no `SKILL.md`: principios, fluxo de decisao resumido, e o passo a passo minimo para executar a tarefa mais comum.

### Passo 7: Checklist de Qualidade

Rodar checklist completo antes de criar o arquivo. Ver `references/checklist-qualidade.md`.

### Passo 8: Criar o Arquivo

- Caminho: `.assistant/skills/[skill-name]/SKILL.md` (+ `references/` se aplicavel)
- Nao criar pastas adicionais sem necessidade concreta
- Ao atualizar skill existente, seguir semantic versioning — ver `references/versionamento.md`

## Referencias

- `references/dominios-e-colisoes.md` — tabela de dominios, colisao com skills nativas, principio DRY
- `references/checklist-qualidade.md` — checklist completo + metricas alvo
- `references/versionamento.md` — regras de semantic versioning para atualizar skills
- `references/troubleshooting-registry.md` — skill some do registry: causas confirmadas vs. em investigacao
- `references/exemplo-completo.md` — exemplo passo a passo de verificacao DRY e validacao de uma skill nova

## Aplicacao

- Carregar esta skill antes de criar, editar ou revisar qualquer skill
- Sempre rodar a verificacao DRY (passos 1-2) antes de qualquer outra etapa
- Se a skill proposta violar padroes ou duplicar uma existente: corrigir antes de criar, redirecionar para edicao da existente, ou orientar o uso da skill global
- Skills que vao gerar mudancas em arquivos de outros projetos (fora do proprio `.assistant/skills/`) devem propor a mudanca ao usuario antes de aplicar, nunca aplicar de forma autonoma
