---
name: documentacao-artefatos
description: Use ao documentar, registrar, descrever, especificar, revisar ou auditar artefato tecnico existente (notebook, tabela, pipeline, regra de negocio) a partir do que existe no ambiente. Cobre coleta de fontes diretas (codigo, schema, tabela, comentario UC, git), classificacao de material, criterio de admissao de afirmacoes, tratamento de divergencias e lacunas. Nao cobre README, revisao de logica de codigo, logs, historico de investigacao ou decisoes (sao narrativos).
---

# Documentacao de Artefatos Tecnicos

**Versao:** 1.1.0 | **Data:** 2026-08-26 | **Dominio:** documentation | **Autor:** Pedro O. Silva

## Quando Usar Esta Skill

**USAR** ao:
- Documentar, registrar, descrever ou especificar artefato tecnico existente.
- Revisar, reescrever ou auditar documentacao de artefato tecnico existente.
 
## Dependencia

Carregar `padrao-escrita` antes de criar, revisar, reescrever ou auditar a documentacao. Esta skill define procedencia, admissao, lacunas e divergencias; `padrao-escrita` define tom, registro e formatacao.

**NAO USAR** para:
- Revisar logica de codigo → revisao-codigo-quatro-frentes cobre
- Mapear quais documentos atualizar apos mudancas → protocolo-atualizacao cobre
- Documentar logs, historico de investigacao ou evolucao do projeto → material narrativo nao entra

## Principios (Orientacao)

Estas ideias guiam toda decisao tomada no procedimento abaixo — leia antes de executar.

1. **Coleta nativa, decisao da skill.** As capacidades nativas do Genie Code (readAssetById, readTable, tableSearch, etc.) fazem a busca. Esta skill decide o que fazer com o material coletado.

2. **Alvo explicito antes de coletar.** Documento pode morar em: (a) celulas markdown dentro do notebook, ou (b) arquivo .md no repositorio (tecnico: implementacao/estrutura/schema, ou de negocio: regra/significado/criterio). Nunca assumir, nunca escolher pelo usuario, nunca produzir os dois. Se o pedido nao deixar claro, perguntar diretamente.

3. **Procedencia verificavel.** Toda afirmacao vem de fonte direta: o proprio artefato (codigo do notebook, schema, definicao de tabela, comentario de coluna no Unity Catalog, git). O criterio e que a afirmacao venha do artefato em si, nao de um texto que fala sobre ele. Estagio de maturidade e irrelevante: rascunho, desenvolvimento e producao valem igualmente.

4. **Material narrativo descartado.** Log, historico de investigacao, registro de evolucao, documento de decisao sao narrativos — descrevem um momento no tempo. Nao entram na coleta, nao sao reproduzidos nem parafraseados. Ponteiro para um deles so aparece se o documento tiver secao declarada para isso, e o arquivo e escolhido pelo usuario, nunca varrido.

5. **Admissao rigorosa.** Nao entra: afirmacao sem fonte direta correspondente, numero sem data e sem metodo de obtencao, exemplo hipotetico apresentado como ocorrido, processo ou fluxo que nao existe no repositorio, qualificador de maturidade nao verificavel, contagem que pode desincronizar (derive na hora ou nao afirme), conteudo que a plataforma ja exibe sozinha.

6. **Divergencia sem arbitragem.** Fontes que discordam nao sao arbitradas. A afirmacao nao entra e vira item numa lista apresentada ao usuario.

7. **Lacuna declarada e resultado correto.** Documento com lacuna declarada e resultado correto, nao falha. Nunca preencher com conhecimento proprio, jamais fazer inferencias. Listar o que faltou ao final.

8. **Tom sobrio.** Bloco curto: sobrio, tecnico, factual, verbos no presente.

## Fluxo de Decisao (Execucao)

O agente percorre os passos na ordem. Cada verificacao pode bloquear o passo atual.

```
PEDIDO DE DOCUMENTACAO OU REVISAO DE ARTEFATO
        |
        v
[0] Alvo e destino estao explicitos?
        |-- NAO --> perguntar: notebook ou projeto/pipeline?
        |           se projeto/pipeline: tecnica ou de negocio?
        |-- SIM
        v
[1] Classificar o material que a busca nativa trouxe
    - fonte direta (codigo, schema, tabela, coluna, git)
    - narrativo (log, investigacao, evolucao, decisao)
        |
        v
[2] Descartar todo o material narrativo
        |
        v
[3] Para cada afirmacao candidata: existe fonte direta que a sustente?
        |-- NAO --> nao entra, vai para lista de lacunas
        |-- SIM
        v
[4] A afirmacao passa nos criterios de admissao?
        |-- NAO --> nao entra
        |-- SIM
        v
[5] Duas fontes discordam sobre ela?
        |-- SIM --> nao entra, vai para lista de divergencias
        |-- NAO
        v
[6] Redigir ou corrigir com o material aprovado
        |
        v
[7] Apresentar documento + lacunas + divergencias
```

## Criterios de Admissao (Verificaveis)

Cada regra abaixo e um criterio sim/nao. Afirmacao reprovada em qualquer item NAO entra no documento.

| Criterio | Teste |
|----------|-------|
| Existe fonte direta que sustente a afirmacao? | Codigo, schema, tabela, comentario UC, git = SIM. Texto que fala sobre o artefato = NAO. |
| O numero tem data e metodo de obtencao? | Se nao tem, nao entra. |
| O exemplo e real (ocorreu) ou hipotetico? | Hipotetico nao entra. |
| O processo/fluxo existe no repositorio? | Se nao existe, nao entra. |
| O qualificador de maturidade e verificavel? | "Em desenvolvimento" so entra se houver marcador verificavel (git branch, tag, comentario no codigo). |
| A contagem pode desincronizar? | Se pode (ex.: "tabela tem 3 colunas" quando colunas mudam), derive na hora ou nao afirme. |
| A plataforma ja exibe esse conteudo sozinha? | Se sim (ex.: schema completo de tabela), nao duplicar no documento. |
| Duas fontes discordam sobre essa afirmacao? | Se sim, nao entra — vai para lista de divergencias. |

## Exemplo de Aplicacao

**Cenario:** Documentar a tabela `catalog.schema.vendas`

**Coleta nativa trouxe:**
- Schema da tabela (fonte direta: Unity Catalog)
- Comentario da coluna `data_venda`: "Data da transacao" (fonte direta: Unity Catalog)
- Celula MD de notebook: "Esta tabela tem 1.2M linhas" (narrativo: numero sem data)
- Log de pipeline: "Processamento concluido em 5 min" (narrativo: log)

**Passo 3-5:** Verificar cada afirmacao:
- Schema → fonte direta, nao duplica (plataforma ja exibe), NAO ENTRA
- Comentario da coluna → fonte direta, verificavel, ENTRA
- "1.2M linhas" → sem data/metodo, NAO ENTRA (vai para lacunas)
- "5 min de processamento" → log, NAO ENTRA (descartado no passo 2)

**Passo 6-7:** Documento final:
```
## Tabela vendas

**Coluna data_venda:** Data da transacao.

### Lacunas
- Volume de dados: afirmacao "1.2M linhas" sem data de referencia ou metodo de obtencao.
```

## Aplicacao

- Carregar esta skill ao receber pedido de documentacao ou revisao de documentacao de artefato tecnico.
- Sempre perguntar alvo (notebook vs projeto, tecnico vs negocio) se nao estiver explicito.
- Nunca arbitrar divergencias — listar e deixar usuario decidir.
- Nunca preencher lacunas com inferencias — declarar a lacuna.
- Aplicar criterios de admissao como testes sim/nao, nao como recomendacoes de estilo.
