---
name: project-context
description: Use ao iniciar, criar, planejar, continuar, retomar ou fechar um projeto de dados. Busca a pasta do projeto e classifica o estado (ausente, incompleto, completo), conduz entrevista de definicoes, desenha o mapa do pipeline, cria a estrutura de pastas e os documentos fundamentais, recupera onde o projeto parou e declara o proximo passo. NAO use para criar notebooks, escrever codigo, implementar transformacoes ou construir tabelas -- o planejamento termina na estrutura e nos documentos, e a construcao e sessao separada. NAO use para propagar documentacao (docs-sync), nomear assets (naming-conventions), decidir localizacao de asset avulso (asset-placement), escolher estrategia de gravacao (medallion-architecture) ou documentar artefato existente (artifact-documentation).
---

# Contexto de Projeto

**Versão:** 2.0.0 | **Data:** 2026-08-30 | **Domínio:** project-management | **Autor:** Pedro O. Silva

## Quando Usar Esta Skill

**USAR** ao iniciar, criar, planejar, continuar, retomar ou fechar um projeto de dados — o usuário fala do projeto como um todo, não de um artefato específico dentro dele.

**NÃO USAR** para:
- Criar notebook, escrever código ou implementar transformação → `@notebook-structure`, `@medallion-architecture`
- Propagar atualizações de documentação após implementar mudanças → `@docs-sync`
- Nomear assets → `@naming-conventions`
- Decidir onde criar um asset avulso → `@asset-placement`
- Decidir estratégia de gravação ou camadas → `@medallion-architecture`
- Documentar artefato existente → `@artifact-documentation`

Teste rápido de escopo: **se o artefato a produzir é `.md` ou é pasta, é trabalho desta skill. Se é `.py`, não é.**

## Princípios

1. **Planejamento antecede código.** Esta skill entrega a estrutura, os documentos e o mapa do pipeline. Ela nunca escreve notebook, célula ou transformação, mesmo que o próximo passo óbvio seja esse. Declara e para.
2. **Skills não guardam estado.** Os quatro documentos persistentes sobrevivem entre sessões e são a única memória do projeto.
3. **A busca decide o modo, não o verbo do usuário.** "Vamos começar" num projeto que já existe é continuação. "Continua de onde paramos" numa pasta vazia é criação. O estado encontrado manda.
4. **O estado tem três valores, não dois.** Ausente, incompleto e completo pedem tratamentos diferentes. Tratar incompleto como ausente refaz entrevista já respondida; tratar como completo carrega um plano furado.
5. **Sem inferências. Perguntar.** Campo não respondido → `[PENDENTE]`. Nunca supor nome, escopo, fonte ou consumidor.
6. **Supersedência ativa.** Decisão nova que invalida decisão antiga marca a antiga como supersedida. Nunca apagar, nunca editar o corpo.

## Procedimento

### Passo 1: Identificar o tipo de pedido

Pedido de fechamento ("registra a sessão", "fecha a sessão", "atualiza o log") → ir direto ao **MODO D**. O projeto já está carregado; a busca é desnecessária.

Caso contrário → Passo 2.

### Passo 2: Buscar o projeto e classificar o estado

1. Consultar o Índice de Projetos no `.assistant_instructions.md` (fonte primária de path).
2. Se não estiver listado: `readAssetById(directory, path da home)` → procurar a pasta.
3. `readAssetById(directory, path do projeto)` → inventariar o que existe.

Classificar em um dos três estados:

| Estado | Condição | Modo |
|--------|----------|------|
| **Ausente** | Pasta não existe, ou existe sem nenhum dos documentos | A |
| **Incompleto** | Pasta existe, mas falta documento, falta pasta obrigatória, ou há campo `[PENDENTE]` | B |
| **Completo** | Os quatro documentos existem e não há `[PENDENTE]` aberto | C |

### MODO A: Projeto ausente — planejar do zero

1. **Entrevista de definições.** Perguntar, nesta ordem: nome, abreviação (usada nos schemas UC), objetivo em uma frase, pergunta que o projeto responde, fontes (nome + origem + periodicidade), escopo e não-escopo, quem consome o resultado. Campo não respondido → `[PENDENTE]`.
2. **Entrevista do pipeline.** Perguntar quais etapas o projeto terá, o que cada uma consome e o que produz, e onde as linhagens convergem. Este é o insumo do mapa.
3. **Criar a pasta do projeto.** Consultar `@asset-placement` para decidir onde.
4. **Criar o esqueleto de pastas** (ver Estrutura de Pastas).
5. **Escrever os quatro documentos.** Consultar `@technical-writing` para o padrão de redação. Usar os templates abaixo.
6. **Criar schemas e estrutura UC.** Consultar `@naming-conventions` e `@unity-catalog-naming` — a abreviação da entrevista nomeia os schemas.
7. **Adicionar o projeto ao Índice de Projetos** em `.assistant_instructions.md`, mediante autorização do usuário.
8. Aplicar a **Regra de Saída**.

### MODO B: Projeto incompleto — completar o plano

1. **Diagnosticar as lacunas.** Listar o que falta: documento ausente, pasta obrigatória ausente, campo `[PENDENTE]`, notebook existente no workspace que não está no mapa do pipeline, fonte em uso que não está nas definições.
2. **Declarar as lacunas ao usuário** antes de perguntar qualquer coisa.
3. **Perguntar apenas o que falta.** Não reabrir campo já respondido. Não repetir a entrevista completa.
4. **Preencher.** Criar o documento ou pasta faltante, substituir o `[PENDENTE]` pela resposta, atualizar o mapa.
5. Aplicar a **Regra de Saída**.

### MODO C: Projeto completo — recuperar e orientar

1. **Ler `definicoes_projeto.md` por inteiro.** É curto e é o contrato do projeto.
2. **Ler `mapa_pipeline.md` por inteiro.** É o desenho do que existe e do que falta construir.
3. **Ler `decisoes_arquiteturais.md` por inteiro.** Contém as decisões vigentes.
4. **Ler as últimas 40 linhas de `evolucao_projeto.md`** com `readAssetById` (usar startLine/endLine). Se o arquivo tiver 40 linhas ou menos, ler tudo. Justificativa: 40 linhas cobrem cerca de 5 resumos de sessão, suficiente para trajetória recente e estado atual.
5. **Declarar:** onde o projeto parou, estado atual e próximo passo.
6. **Aguardar confirmação ou correção do usuário** antes de qualquer trabalho.
7. Aplicar a **Regra de Saída**.

### MODO D: Fechar sessão

1. **Levantar o que foi feito.** Listar ações e mudanças da sessão.
2. **Perguntar o que ficou pendente e qual é o próximo passo.** Não inferir. Sem resposta → `[PENDENTE]`.
3. **Verificar supersedência.** Se a sessão produziu decisão arquitetural:
   - a. Ler as decisões vigentes em `decisoes_arquiteturais.md`.
   - b. Avaliar se a nova invalida, contradiz ou substitui alguma.
   - c. Se invalidar: marcar a antiga como SUPERSEDIDA (ver Templates). Sem certeza: PERGUNTAR.
   - d. Decisão supersedida nunca é apagada nem editada no corpo.
4. **Atualizar o mapa do pipeline** se etapas foram construídas, removidas ou renumeradas.
5. **Escrever o resumo no log de evolução.** Usar o template do Documento 4.
6. **Encaminhar para `@docs-sync`** se houve mudança que exija propagar para a documentação técnica.

## Regra de Saída

Todo modo termina declarando o próximo passo. O próximo passo cai em um de dois lugares:

**Dentro desta skill** — pasta faltante, documento faltante, campo `[PENDENTE]` que já tem resposta, decisão a registrar, mapa desatualizado. A skill executa.

**Fora desta skill** — notebook, transformação, tabela, qualquer arquivo `.py`. A skill **declara e encerra a sessão de planejamento**. Não escreve a primeira célula, não esboça, não começa.

Formato da declaração de saída:

```
Proximo passo: [descricao]. Isso e sessao de construcao — usa @notebook-structure,
@naming-conventions e @medallion-architecture.
```

## Templates

Todos vivem em `00_documentacao/` na raiz do projeto.

### Documento 1: Definições (`definicoes_projeto.md`)

Escrito uma vez, na criação. Raramente muda. É o contrato do projeto.

```markdown
# Definicoes do Projeto: [Nome]

**Abreviacao:** [abbr]
**Objetivo:** [uma frase]
**Pergunta que responde:** [pergunta]

## Fontes
| Fonte | Origem | Periodicidade |
|-------|--------|---------------|
| [nome] | [origem] | [periodo] |

## Escopo
- Inclui: [itens]
- Nao inclui: [itens]

## Consumidores
[quem consome o resultado]
```

### Documento 2: Mapa do Pipeline (`mapa_pipeline.md`)

O desenho do que será construído. Atualizado sempre que uma etapa nasce, morre ou muda de posição. É o documento que a sessão de construção consulta para saber o que escrever.

```markdown
# Mapa do Pipeline: [Nome]

## Linhagem

[diagrama em texto — uma linha por caminho, convergencias explicitas]

## Etapas

| Notebook | Camada | Le | Escreve | Status |
|----------|--------|----|---------|--------|
| [1xx_nome] | bronze | [origem] | [tabela] | planejado \| construido |
```

Regras:
- `Status` só tem dois valores. Notebook construído é fato verificável no workspace.
- Numeração segue `@naming-conventions` (`1xx_` bronze, `2xx_` silver, `3xx_` gold).
- Não detalhar lógica de transformação aqui — isso vive no notebook e em arquitetura.md.

### Documento 3: Decisões Arquiteturais (`decisoes_arquiteturais.md`)

Cada decisão é uma entrada numerada (DEC-001, DEC-002...). Decisão que muda não é editada — é marcada como supersedida.

Vigente:
```markdown
### DEC-001 -- [titulo]
**Data:** YYYY-MM-DD
**Contexto:** [o que motivou]
**Decisao:** [o que foi decidido]
**Alternativas consideradas:** [lista]
**Justificativa:** [por que esta]
```

Supersedida:
```markdown
### DEC-001 -- ~~[titulo]~~

**SUPERSEDIDA em YYYY-MM-DD por DEC-00X.**

[conteudo original preservado sem alteracao]
```

Regras:
- Riscado no título + linha SUPERSEDIDA em negrito no início. O corpo permanece legível, nunca riscado.
- O documento é lido por inteiro no MODO C — manter conciso.
- Não detalhar implementação técnica aqui — isso vai em arquitetura.md via `@docs-sync`.

### Documento 4: Log de Evolução (`evolucao_projeto.md`)

Resumo curto de cada sessão. Se for longo, não é preenchido.

```markdown
## YYYY-MM-DD -- Sessao

**Feito:** [1-3 linhas]
**Estado atual:** [1 linha]
**Proximo passo:** [1 linha]
**Pendencias:** [lista ou [PENDENTE]]
```

Regras:
- Não detalhar decisões aqui — vão em `decisoes_arquiteturais.md`.
- Não detalhar implementação técnica — vai em arquitetura.md via `@docs-sync`.
- Entrada nova no topo ou no fim, seguindo o padrão já existente no arquivo.

## Estrutura de Pastas

### Obrigatórias (todo projeto novo)

```
projeto/
|-- 00_documentacao/
|   |-- tecnica/                  (vazia na criacao)
|   |-- negocio/                  (vazia na criacao)
|   |-- definicoes_projeto.md
|   |-- mapa_pipeline.md
|   |-- decisoes_arquiteturais.md
|   `-- evolucao_projeto.md
|-- 01_bronze/                    (vazia na criacao)
|-- 02_silver/                    (vazia na criacao)
|-- 03_gold/                      (vazia na criacao)
`-- README.md
```

### Condicionais (criadas só quando o projeto precisar)

| Pasta | Quando criar |
|-------|--------------|
| `04_exploracao/` | Investigação exploratória antes de implementar transformação |
| `05_apoio/` | DDL, config de parâmetros, orquestrador, download de fonte externa |
| `06_testes/` | Testes de integração ou notebooks de teste |

### Não criar (artefatos de DAB/tooling)

- `.databricks/`, `databricks.yml`, `config/`, `resources/` — DAB bundle
- `.github/workflows/` — GitHub Actions CI
- `tests/` — pytest (DAB)
- `ruff.toml`, `.ruff_cache/`, `.pytest_cache/` — tooling
- `LICENSE` — repositório

### Consistência de numeração

A numeração das pastas 01-03 bate com o 1º dígito do notebook (`1xx_`, `2xx_`, `3xx_`) e com a numeração do schema UC (`proj_x_01_bronze`, `proj_x_02_silver`, `proj_x_03_gold`). Pastas 04-06 não são camadas de pipeline e não seguem esse padrão.

## Skills Adjacentes

| Skill | Relação | Quando |
|-------|---------|--------|
| `@asset-placement` | consultar | MODO A, passo 3 — onde criar a pasta |
| `@technical-writing` | consultar | MODO A e B — padrão de redação dos documentos |
| `@naming-conventions` | consultar | MODO A, passo 6 — nomear schemas e numerar etapas |
| `@unity-catalog-naming` | consultar | MODO A, passo 6 — organizar schemas UC |
| `@docs-sync` | encaminhar | MODO D — propagar mudança para doc técnica |
| `@notebook-structure` | encaminhar | Regra de Saída — construção de notebook |
| `@medallion-architecture` | encaminhar | Regra de Saída — estratégia de gravação |

**Consultar** é buscar o padrão para produzir o próprio artefato desta skill. **Encaminhar** é declarar que o trabalho é de outra e sair.

## Checklist Final de Validação

Antes de encerrar a sessão, confirmar:

1. [ ] O estado do projeto foi classificado (ausente, incompleto, completo) antes de qualquer ação?
2. [ ] No MODO B, apenas as lacunas foram perguntadas — nenhum campo já respondido foi reaberto?
3. [ ] Todo campo sem resposta ficou marcado `[PENDENTE]`, sem nenhuma suposição?
4. [ ] Os quatro documentos existem e o mapa do pipeline reflete o estado real do workspace?
5. [ ] Nenhum arquivo `.py` foi criado, escrito ou esboçado nesta sessão?
6. [ ] O próximo passo foi declarado explicitamente?
7. [ ] Se o próximo passo é código, a sessão foi encerrada com o encaminhamento em vez de continuar?
8. [ ] No MODO D, a supersedência foi verificada contra as decisões vigentes?