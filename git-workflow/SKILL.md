---
name: git-workflow
description: Use APENAS ao fazer COMMIT ou COMMITAR mudanças no Git -- quando o estado é commitável, divisão de commits por escopo técnico (bisect/revert/review), staging parcial (git add -p), o que nunca entra no commit, mensagens de commit tom neutro, amend. NÃO use para operações Git gerais (status, checkout, pull, merge, conflitos) -- a skill global git cobre isso.

---

# Git Workflow - Padrões de Commit e Versionamento

**Versão:** 1.1.0 | **Data:** 2026-08-16 | **Autor:** Pedro O. Silva

## Princípios Gerais

* **Critério técnico > critério estético**: Decisões sobre divisão de commits baseadas em facilidade de bisect/revert/review, não em "como parece melhor no histórico"
* **Tom neutro**: Mensagens de commit diretas ao ponto, sem linguagem de alerta/urgência (sem emojis, "CRÍTICO", "decisão fundamental")
* **Pragmatismo**: Evitar complexidade desnecessária (ex: edição manual de patches) quando alternativas simples existem
* **Histórico é permanente**: O que entra no commit fica. Revisão do que está staged é etapa obrigatória, não opcional

## Quando Commitar

### Critério

Commit acontece por **unidade de trabalho concluída**, não por tempo decorrido. O gatilho é:

* Algo que estava quebrado passou a funcionar
* Uma decisão foi tomada e implementada
* Uma unidade coerente ficou completa (função + teste, notebook + DDL correspondente)

**Consequência prática:** um dia produtivo pode gerar 3 commits; um dia de tentativa que não convergiu pode gerar 0. Frequência não é métrica.

### Quando NÃO Commitar

* **Estado quebrado no meio de refatoração**: terminar e commitar. Se precisar interromper, usar `git stash` em vez de commitar código não funcional
* **Commit como backup**: commit registra um estado que faz sentido, não um ponto de salvamento arbitrário. O histórico serve para navegar entre pontos onde o projeto funcionava

### Amend

Para corrigir o commit imediatamente anterior (arquivo esquecido, typo na mensagem):

```bash
git add arquivo_esquecido.py
git commit --amend --no-edit          # mantém a mensagem
git commit --amend                    # abre editor para reescrever a mensagem
```

**Regra:** `--amend` livremente antes do push. Depois do push, não — reescreve histórico já publicado.

## Divisão de Commits

### Quando Dividir Commits

Dividir em commits separados quando as mudanças pertencem a **escopos técnicos distintos**:

* **Escopo diferente**: Mudanças em repositórios/projetos diferentes (ex: reestruturação interna vs extração para repo externo)
* **Revert independente**: Possibilidade de reverter uma mudança sem afetar a outra (ex: adicionar feature vs refactor de estrutura)
* **Review separado**: Mudanças que um revisor analisaria em contextos mentais diferentes (ex: lógica de negócio vs infraestrutura)

### Quando NÃO Dividir

* Mudanças que fazem parte do mesmo escopo técnico (ex: adicionar função + seus testes)
* Mudanças que dependem umas das outras para funcionar (ex: renomear função + atualizar chamadas)
* Quando a divisão adiciona complexidade sem benefício real de bisect/revert/review

**Benefício:** Histórico navegável via `git bisect` e revert granular sem perder trabalho válido.

## O Que Nunca Entra no Commit

Histórico é permanente. Remover um arquivo em commit posterior não o tira do histórico — exige reescrita, que é trabalhosa e perigosa se já houve push.

### Categorias Bloqueadas

| Categoria | Exemplos | Razão |
| --- | --- | --- |
| **Credenciais** | tokens, PATs, `.env`, connection strings, chaves de API | Exposição permanente, mesmo em repo privado |
| **Dados** | `.zip`, `.csv`, `.parquet`, extrações, amostras | Infla o repo, e dado versionado em Git é antipadrão |
| **Artefatos de ambiente** | `__pycache__/`, `.ipynb_checkpoints/`, `.venv/`, `.DS_Store` | Ruído, conflito entre máquinas |
| **Instruções de agente** | `.agent_instructions/`, arquivos de configuração de assistente | Não é código do projeto |
| **Output de execução** | resultado de célula, logs, tabelas renderizadas | Diff ilegível, informação volátil |

### `.gitignore` Base

```gitignore
# Credenciais
.env
*.token
*.pem

# Dados
*.zip
*.csv
*.parquet
/landing/
/dados/

# Ambiente
__pycache__/
*.pyc
.venv/
.ipynb_checkpoints/
.DS_Store

# Agente
.agent_instructions/
```

### Notebooks

Notebook exportado como `.py` com separadores `# COMMAND ----------` versiona limpo — o formato não carrega output de célula. Formato `.ipynb` carrega, e produz diff ilegível. Preferir `.py` no repositório.

## Staging Parcial

### git add -p (Patch Mode)

Para arquivos com mudanças que pertencem a commits diferentes:

```bash
git add -p arquivo.md
```

**Interação:**
* `y` = aceitar hunk (incluir no próximo commit)
* `n` = rejeitar hunk (deixar para commit posterior)
* `s` = split (tentar dividir hunk grande em partes menores)
* `e` = edit (edição manual do patch - **EVITAR**, ver abaixo)
* `q` = quit (sair sem stagear mais nada)

### Estratégia Anti-Travamento

**CRÍTICO:** Edição manual de patches (`e`) pode travar o workflow.

**Alternativa pragmática:**
* Se `s` (split) não funcionar, **não usar `e`**
* Deixar o arquivo inteiro de fora do primeiro commit
* Adicionar o arquivo completo no segundo commit via `git add` normal

**Trade-off aceito:** Perda de "pureza" (arquivo aparece inteiro no segundo commit mesmo contendo mudanças tematicamente do primeiro) em troca de execução sem travamento.

**Exemplo:** `evolucao_projeto.md` com entradas de 3 datas diferentes - se aparecer como 1 hunk grande, deixar inteiro para o commit 2 em vez de tentar editar manualmente.

### Revisão Obrigatória Antes do Commit

Independente do método de staging:

```bash
git status                  # o que está staged vs não staged
git diff --staged           # exatamente o que vai entrar no commit
```

`git add .` é conveniente e é como arquivo indevido acaba versionado. Quando usado, `git diff --staged` deixa de ser recomendação e passa a ser obrigatório.

## Mensagens de Commit

### Formato

```
<tipo>(<escopo>): <título conciso>

<corpo - detalhamento organizado>
* Ponto 1
* Ponto 2
* Ponto 3

<rodapé - referências opcionais>
```

**Tipos comuns:**
* `feat`: Nova funcionalidade
* `fix`: Correção de bug
* `refactor`: Refatoração sem mudar comportamento
* `docs`: Apenas documentação
* `chore`: Tarefas de manutenção (build, configs)

### Escopo

O escopo entre parênteses identifica a área afetada. Em projetos com arquitetura em camadas, mapeia direto para a camada:

* `bronze`, `silver`, `gold` - camadas do pipeline
* `apoio` - infraestrutura, DDL, config, orquestração
* `docs` - quando a documentação é o escopo, não o tipo
* nome do módulo ou domínio, em projetos sem camadas

**Benefício:** `git log --oneline` fica legível por área, e `git log --grep="(silver)"` filtra o histórico de uma camada.

**Exemplos:**
```
feat(bronze): Adiciona ingestão da BPP
fix(silver): Corrige tipo de CD_CVM na projeção explícita
refactor(apoio): Centraliza detecção de anos em config_parametros
docs(arquitetura): Reconcilia estratégia de gravação da Bronze
```

Escopo é opcional quando a mudança é transversal ao projeto inteiro.

### Tom e Linguagem

* **Direto ao ponto**: Sem adjetivos dramáticos, emojis ou ênfase excessiva
* **Factual**: O que mudou e por quê, sem marketing
* **Neutro**: Evitar "CRÍTICO", "urgente", "decisão fundamental"
* **Verbo no presente**: "Adiciona", "Corrige", "Remove" — não "Adicionado" nem "Adicionei"

**Exemplo BOM:**
```
refactor(apoio): Reorganiza pastas por ordem lógica do fluxo

Reestrutura pastas para refletir sequência do fluxo de dados:
* 04_apoio → 05_apoio (infraestrutura vem após análises)
* Cria 04_analises_exploratorias (preparação para EDA)
```

```
feat(bronze): Adiciona pipeline de ingestão da BPP

* Notebook 103_cvm_dfp_bpp lendo da Landing Zone
* Tabela proj_cvm_01_bronze.103_bpp_dfp
* COLUNAS_ESSENCIAIS_BPP em config_parametros
```

Reorganização de estrutura e nova funcionalidade são revertíveis de forma independente — pela regra de divisão, são dois commits, não um.

**Exemplo RUIM:**
```
refactor: MUDANÇA CRÍTICA! Reorganização completa da arquitetura

Decisão fundamental que transforma o projeto...
```

```
update
```

```
ajustes gerais e correções
```

Mensagem genérica anula o valor do histórico: o commit deixa de ser navegável e vira só um ponto no tempo.

### Tags Especiais

**BREAKING CHANGE:**
* **Usar APENAS** quando há quebra de contrato com consumidores externos (APIs públicas, bibliotecas compartilhadas)
* **NÃO usar** em projetos de portfólio pessoal, refactorings internos ou reorganizações de estrutura

**Quando NÃO usar:**
* Reorganização de pastas em projeto pessoal
* Extração de código para outro repo (não há consumidor externo)
* Refactoring de arquitetura interna

## Referências Git

### Formato

Quando commit referencia outro recurso (repositório externo, issue, documentação):

```
Refs: https://github.com/usuario/projeto
```

**Validar ANTES de commitar:**
* Repositório existe e está acessível (HTTP 200)
* Link não quebrado permanentemente no histórico

### Decisão: Incluir ou Não

* **Incluir**: Quando o commit depende de contexto externo necessário para entender a mudança
* **Omitir**: Quando a referência é opcional ou o commit é autocontido

## Checklist Pré-Commit

1. [ ] O estado é commitável (algo funciona ou uma decisão foi concluída)?
2. [ ] `git diff --staged` revisado — nada indevido entrou?
3. [ ] Credencial, dado, artefato de ambiente ou output de célula fora do staging?
4. [ ] Commits divididos por escopo técnico (não estético)?
5. [ ] Staging parcial necessário? Se sim, `git add -p` funciona ou precisa de alternativa?
6. [ ] Mensagem no formato `<tipo>(<escopo>): <título>`, verbo no presente?
7. [ ] Mensagem factual, tom neutro, sem dramatização?
8. [ ] Tag `BREAKING CHANGE` removida se não houver consumidor externo?
9. [ ] Referências externas validadas (repos existem, links funcionam)?
10. [ ] Se o diff incluir arquivo .md que cite caminho, nome de pasta, notebook, tabela ou skill, confirmar que o que ele cita está presente no mesmo commit ou já existe no main?
11. [ ] Documentação atualizada reflete as mudanças commitadas?

---

**Relacionadas:** Consulte a skill global "git" para operações Git gerais (status, checkout, pull, merge, resolução de conflitos).