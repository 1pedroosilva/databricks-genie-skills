---
name: git-workflow
description: Use APENAS ao COMMITAR mudancas no Git -- divisao de commits, staging parcial, mensagens de commit. Criterio tecnico para separacao (bisect/revert/review), staging com git add -p para arquivos mistos, evitar edicao manual de patches, sem tag BREAKING CHANGE em projetos pessoais, tom neutro. NAO use para operacoes Git gerais (status, checkout, pull, merge, conflitos) -- a skill global "git" cobre isso.

---

# Git Workflow - Padroes de Commit e Versionamento

**Versao:** 1.0.0 | **Data:** 2026-08-15 | **Autor:** Pedro O. Silva

## Principios Gerais

* **Criterio tecnico > criterio estetico**: Decisoes sobre divisao de commits baseadas em facilidade de bisect/revert/review, nao em "como parece melhor no historico"
* **Tom neutro**: Mensagens de commit diretas ao ponto, sem linguagem de alerta/urgencia (sem emojis, "CRITICO", "decisao fundamental")
* **Pragmatismo**: Evitar complexidade desnecessaria (ex: edicao manual de patches) quando alternativas simples existem

## Divisao de Commits

### Quando Dividir Commits

Dividir em commits separados quando as mudancas pertencem a **escopos tecnicos distintos**:

* **Escopo diferente**: Mudancas em repositorios/projetos diferentes (ex: reestruturacao interna vs extracao para repo externo)
* **Revert independente**: Possibilidade de reverter uma mudanca sem afetar a outra (ex: adicionar feature vs refactor de estrutura)
* **Review separado**: Mudancas que um revisor analisaria em contextos mentais diferentes (ex: logica de negocio vs infraestrutura)

### Quando NAO Dividir

* Mudancas que fazem parte do mesmo escopo tecnico (ex: adicionar funcao + seus testes)
* Mudancas que dependem umas das outras para funcionar (ex: renomear funcao + atualizar chamadas)
* Quando a divisao adiciona complexidade sem beneficio real de bisect/revert/review

**Beneficio:** Historico navegavel via `git bisect` e revert granular sem perder trabalho valido.

## Staging Parcial

### git add -p (Patch Mode)

Para arquivos com mudancas que pertencem a commits diferentes:

```bash
git add -p arquivo.md
```

**Interacao:**
* `y` = aceitar hunk (incluir no proximo commit)
* `n` = rejeitar hunk (deixar para commit posterior)
* `s` = split (tentar dividir hunk grande em partes menores)
* `e` = edit (edicao manual do patch - **EVITAR**, ver abaixo)
* `q` = quit (sair sem stagear mais nada)

### Estrategia Anti-Travamento

**CRITICO:** Edicao manual de patches (`e`) pode travar o workflow.

**Alternativa pragmatica:**
* Se `s` (split) nao funcionar, **nao usar `e`**
* Deixar o arquivo inteiro de fora do primeiro commit
* Adicionar o arquivo completo no segundo commit via `git add` normal

**Trade-off aceito:** Perda de "pureza" (arquivo aparece inteiro no segundo commit mesmo contendo mudancas tematicamente do primeiro) em troca de execucao sem travamento.

**Exemplo:** `evolucao_projeto.md` com entradas de 3 datas diferentes - se aparecer como 1 hunk grande, deixar inteiro para o commit 2 em vez de tentar editar manualmente.

## Mensagens de Commit

### Formato

```
<tipo>: <titulo conciso>

<corpo - detalhamento organizado>
* Ponto 1
* Ponto 2
* Ponto 3

<rodape - referencias opcionais>
```

**Tipos comuns:**
* `feat`: Nova funcionalidade
* `fix`: Correcao de bug
* `refactor`: Refatoracao sem mudar comportamento
* `docs`: Apenas documentacao
* `chore`: Tarefas de manutencao (build, configs)

### Tom e Linguagem

* **Direto ao ponto**: Sem adjetivos dramaticos, emojis ou enfase excessiva
* **Factual**: O que mudou e por que, sem marketing
* **Neutro**: Evitar "CRITICO", "urgente", "decisao fundamental"

**Exemplo BOM:**
```
refactor: Reorganiza pastas por ordem logica e adiciona pipeline BPP

Reestrutura pastas para refletir sequencia do fluxo de dados:
* 04_apoio → 05_apoio (infraestrutura vem apos analises)
* Cria 04_analises_exploratorias (preparacao para EDA)
```

**Exemplo RUIM:**
```
refactor: MUDANCA CRITICA! Reorganizacao completa da arquitetura

Decisao fundamental que transforma o projeto...
```

### Tags Especiais

**BREAKING CHANGE:**
* **Usar APENAS** quando ha quebra de contrato com consumidores externos (APIs publicas, bibliotecas compartilhadas)
* **NAO usar** em projetos de portfolio pessoal, refactorings internos ou reorganizacoes de estrutura

**Quando NAO usar:**
* Reorganizacao de pastas em projeto pessoal
* Extracao de codigo para outro repo (nao ha consumidor externo)
* Refactoring de arquitetura interna

## Referencias Git

### Formato

Quando commit referencia outro recurso (repositorio externo, issue, documentacao):

```
Refs: https://github.com/usuario/projeto
```

**Validar ANTES de commitar:**
* Repositorio existe e esta acessivel (HTTP 200)
* Link nao quebrado permanentemente no historico

### Decisao: Incluir ou Nao

* **Incluir**: Quando o commit depende de contexto externo necessario para entender a mudanca
* **Omitir**: Quando a referencia e opcional ou o commit e autocontido

## Checklist Pre-Commit

1. [ ] Commits divididos por escopo tecnico (nao estetico)?
2. [ ] Staging parcial necessario? Se sim, `git add -p` funciona ou precisa de alternativa?
3. [ ] Mensagem de commit factual, tom neutro, sem dramatizacao?
4. [ ] Tag `BREAKING CHANGE` removida se nao houver consumidor externo?
5. [ ] Referencias externas validadas (repos existem, links funcionam)?
6. [ ] Documentacao atualizada reflete as mudancas comitadas?

---

**Relacionadas:** Consulte a skill global "git" para operacoes Git gerais (status, checkout, pull, merge, resolucao de conflitos).
