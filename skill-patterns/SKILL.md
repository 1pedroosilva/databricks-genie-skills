---
name: skill-patterns
description: Use ao criar, desenvolver, elaborar, estruturar, editar, revisar ou auditar skills (novas ou existentes) para o Genie Code contra o padrao de qualidade aqui definido -- cobre validacao de description (dominio de atuacao, colisao com skills nativas da plataforma como notebooks, Unity Catalog, dashboards, pipelines e MLflow), checklist de qualidade, verificacao de duplicidade (DRY) contra skills globais e locais, organizacao de conteudo entre SKILL.md e references/, composicao e teste empirico. NAO use para criar outros tipos de asset (notebooks, tabelas, jobs, pipelines), ou tirar duvidas conceituais sobre como skills funcionam em geral.
---

# Padroes para Criacao de Skills

**Versao:** 2.3.0 | **Data:** 2026-08-26 | **Dominio:** meta | **Autor:** Pedro O. Silva

## Quando Usar Esta Skill

**USAR** ao criar, editar ou revisar uma skill (nova ou existente) — usuario pede explicitamente ou o agente identifica a necessidade durante a conversa.

**NAO USAR** para:
- Criar notebooks, tabelas, jobs ou outros assets → usar a skill especifica daquele asset
- Responder perguntas conceituais sobre skills → esse conteudo ja esta no contexto do agente

## Principios (Orientacao)

Estas ideias guiam toda decisao tomada no procedimento abaixo — leia antes de executar.

1. **Cabecalho YAML minimo.** So `name` e `description` sao campos oficialmente obrigatorios. Se o Genie Code deste ambiente demonstrar incompatibilidade com campos opcionais, registre essa observacao como especifica do ambiente e valide empiricamente; nao a apresente como regra universal da especificacao.
2. **Orientacao separada de execucao.** O corpo da skill deve deixar claro o que e principio (o "porque", estavel, raramente muda) e o que e procedimento (o "como", passo a passo, o agente executa na ordem).
3. **DRY antes de criar.** Nunca criar uma skill sem antes checar se uma skill global ja cobre o topico, se uma user skill ja existente cobre o mesmo escopo, e se o dominio pretendido colide com uma skill nativa da plataforma.
4. **Organizacao condicional de conteudo.** O `SKILL.md` carrega inteiro no contexto assim que a skill e ativada. Conteudo que atende QUALQUER dos criterios abaixo DEVE ir para `references/` (criar a pasta e obrigatorio nesse caso):
   - Consultado em < 30% dos casos de uso (troubleshooting, edge cases raros)
   - Tabelas de dados com > 20 linhas (mapeamentos, codigos de erro, listas extensas)
   - Exemplos completos com > 50 linhas
   - Historico de versoes ou changelog
   Se NENHUM criterio for atendido, a pasta `references/` NAO deve existir — todo conteudo fica no `SKILL.md`.
5. **Menos e mais, mas sem apagar instrucoes eficazes.** Remover uma regra somente quando ela for redundante, nao testada ou causar efeito marginal demonstrado. Preservar procedimento, excecoes, exemplos e referencias que reduzam erros observados.

## Fluxo de Decisao (Navegacao Condicional)

O agente percorre UM dos caminhos abaixo, nao todos. Cada decisao pode encerrar o fluxo ou seguir adiante.

```
PEDIDO DE SKILL NOVA OU REVISAO
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
[3] Validar dominio, colisao e instrucoes vizinhas
        |-- colide ou contradiz ------------> ajustar escopo e documentar a decisao
        |-- livre
        v
[4] Escrever ou revisar description
        |
        v
[5] Preservar o aprendizado valido e marcar cada regra:
    manter | remover | simplificar | testar
        |
        v
[6] Estruturar corpo: Principios (orientacao) + Procedimento (execucao)
        |
        v
[7] Decidir o que fica em SKILL.md vs references/
        |
        v
[8] Criar ou editar SKILL.md e referencias necessarias
        |
        v
[9] Testar a skill sem ela, com a versao anterior e com a revisada
```

## Checklist Obrigatorio Pre-Criacao ou Revisao

**SEMPRE executar antes de criar ou editar arquivo SKILL.md**, independente do caminho tomado no fluxo de decisao acima.

Se o fluxo encerrou no Passo 1 (usar skill global) ou Passo 2 com redirecionamento → checklist NAO se aplica (nenhum arquivo sera criado/editado).

Se o fluxo encerrou no Passo 2 com edicao de skill existente OU chegou ao Passo 8 (criar nova) → checklist e OBRIGATORIO.

**Rodar checklist completo:** ver `references/checklist-qualidade.md`.

## Procedimento Detalhado (Execucao)

### Passo 1-2: Verificacao DRY

Consultar, nesta ordem: skills built-in da plataforma, skills de workspace e user skills. Regra de decisao completa e exemplo passo a passo em `references/exemplo-completo.md`.

Nao confundir a existencia de uma skill built-in com uma regra especifica de documentacao. Se a capacidade nativa ja produz resultado satisfatorio, nao a replique apenas por preferencia.

### Passo 3: Dominio, Colisao e Composicao

O Genie Code ja vem com skills built-in para notebooks, exploracao no Unity Catalog, dashboards, pipelines e MLflow. Uma skill nova nao pode ocupar o mesmo escopo sem diferenciacao clara. Tabela completa em `references/_n_version_dominios-e-colisoes.md`.

Dependencias entre skills devem ser tratadas como orientacao de roteamento. A especificacao Agent Skills nao define `parent`, `depends_on` ou GATE como dependencia executavel. Descricao, mencao explicita com `@nome` e instrucoes claras aumentam a chance de composicao; teste empirico verifica se isso ocorreu.

### Passo 4: Escrever a Description

Formato sugerido:

```
Use ao [acao com sinonimos naturais] [contexto especifico: QUE, QUANDO, ONDE]. [Detalhe do que a skill cobre]. NAO use para [boundaries negativos explicitos].
```

Regras:
- **Concisao ideal**: 1-2 frases (limite tecnico: 1024 caracteres).
- **ASCII puro** (sem acentos) — preferencia de consistencia visual, nao regra tecnica universal.
- **Vocabulario rico**: varias palavras para a mesma acao quando isso melhorar o acionamento.
- **Boundaries negativos explicitos** somente quando evitarem colisao real.
- **Sintaxe YAML segura**: nunca usar `: ` no meio de valor nao protegido.

### Passo 5: Preservar e classificar o aprendizado

Para cada regra existente, registrar uma decisao:

- **manter**: evita erro observado ou protege uma propriedade fragil;
- **remover**: redundante, falsa, nao aplicavel ou causadora de efeito marginal;
- **simplificar**: mesma protecao com menos texto;
- **testar**: efeito incerto, manter provisoriamente ate comparar resultados.

Nao remover uma referencia ou exemplo apenas porque aumenta o tamanho. O criterio e ganho de contexto contra custo de contexto.

### Passo 6: Estruturar o Corpo

- `# Titulo`
- Linha de metadados: `**Versao:** x.y.z | **Data:** YYYY-MM-DD | **Dominio:** categoria | **Autor:** Nome`
- `## Quando Usar Esta Skill` (positivo e negativo)
- `## Principios` (orientacao — o porque, estavel)
- `## Procedimento` ou `## Fluxo de Decisao` (execucao — o como, passo a passo)
- Exemplos concretos, sem emojis, sem tom de marketing

### Passo 7: Decidir SKILL.md vs references/

**Avaliar cada secao contra os criterios abaixo:**

| Criterio | Acao |
|----------|------|
| Secao consultada em < 30% dos casos de uso | -> references/ |
| Tabela de dados com > 20 linhas | -> references/ |
| Exemplo completo com > 50 linhas | -> references/ |
| Historico de versoes / changelog | -> references/ |
| Troubleshooting de casos raros (< 30% dos usos) | -> references/ |
| Nenhum criterio atendido | -> SKILL.md (references/ NAO deve existir) |

### Passo 8: Criar ou Editar

- Caminho: `.assistant/skills/[skill-name]/SKILL.md` (+ `references/` se aplicavel)
- Nao criar pastas adicionais sem necessidade concreta.
- Ao atualizar skill existente, seguir semantic versioning — ver `references/versionamento.md`.
- Antes de salvar, confirme que dependencias referenciadas existem ou estejam explicitamente marcadas como ausentes.

### Passo 9: Testar

Executar tarefas representativas em tres condicoes:

1. Sem skill customizada.
2. Com a skill anterior.
3. Com a skill revisada.

Avaliar factualidade, completude, legibilidade, aparencia, obediencia, falsos acionamentos e custo de contexto. Registrar as correcoes feitas pelo usuario e transformar apenas erros recorrentes em novas regras.

## Referencias

- `references/dominios-e-colisoes.md` — tabela de dominios, colisao com skills nativas, principio DRY
- `references/checklist-qualidade.md` — checklist completo + metricas alvo
- `references/versionamento.md` — regras de semantic versioning para atualizar skills
- `references/troubleshooting-registry.md` — skill some do registry: causas confirmadas vs. em investigacao
- `references/exemplo-completo.md` — exemplo passo a passo de verificacao DRY e validacao de uma skill nova

## Aplicacao

- Carregar esta skill antes de criar, editar ou revisar qualquer skill.
- Sempre rodar a verificacao DRY (passos 1-2) antes de qualquer outra etapa.
- **OBRIGATORIO**: Rodar o checklist de qualidade antes de criar ou editar QUALQUER arquivo SKILL.md.
- Se a skill proposta violar padroes ou duplicar uma existente: corrigir antes de criar, redirecionar para edicao da existente, ou orientar o uso da skill global.
- Skills que vao gerar mudancas em arquivos de outros projetos devem propor a mudanca ao usuario antes de aplicar, nunca aplicar de forma autonoma.
