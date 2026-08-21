# Log de Investigação: Roteamento de skill-patterns vs. skill-authoring

**Problema:** Skill personalizada `skill-patterns` não competia de forma confiável pelo gatilho de conversa com a skill nativa `skill-authoring` - na versão original, sequer chegava a carregar.
**Status:** Correção aplicada, reteste parcial concluído, validação final pendente.
**Causa Raiz:** Escopo da description restrito a "criação do zero", quando o conteúdo da skill cobre um padrão de qualidade aplicável a criação, edição e revisão.

**Nota sobre este log:** os testes não tiveram horário registrado no momento da execução. A sequência abaixo é numerada, não cronometrada - não há timestamps porque nenhum foi de fato capturado.

---

## 2026-08-20 - Verificação DRY e Achados de Documentação Não Registrados

Antes de qualquer teste, a investigação partiu direto para a documentação oficial (`agentskills.io/specification`, `docs.databricks.com/aws/en/genie-code/skills`, `docs.databricks.com/aws/en/agent-skills/`), em vez de re-levantar hipóteses já resolvidas. Cruzando com os docs locais existentes (`investigation_log.md`, `protocolo_teste_empirico.md`, `lessons_learned.md`): ASCII puro, boundaries negativos e verbo imperativo já haviam sido testados e refutados como requisitos técnicos - nenhum dos três foi reaberto aqui.

**Achados da leitura oficial que não constam em nenhum doc local até este log:**

1. **`skill-authoring` como skill nativa distinta.** Nenhum dos três docs locais menciona essa skill pelo nome. Só foi identificada ao vivo, olhando o registry via Genie Code (ver Teste 1 abaixo) - a documentação oficial consultada também não a cita nominalmente, só fala em categorias genéricas (notebooks, Unity Catalog, dashboards, pipelines, MLflow).

2. **Regras exatas do campo `name` no YAML**, da especificação oficial: máximo 64 caracteres, apenas minúsculas/números/hífen, não pode começar ou terminar com hífen, não pode ter hífen duplo, e **deve bater exatamente com o nome da pasta**. Essa última regra (nome = pasta) não foi verificada contra as 12 skills locais existentes - fica como pendência.

3. **Campo opcional `metadata:` (mapa YAML) previsto na especificação**, pensado exatamente para guardar autor/versão dentro do próprio frontmatter. O teste local já registrado em `investigation_log.md` (seção "Frontmatter Duplicado") testou campos soltos no nível raiz do YAML (`version`, `date`, `owner`), não o campo `metadata:` estruturado como a especificação prevê. Ou seja: existe uma hipótese nova, ainda não testada por nenhum protocolo local - "o campo `metadata:` estruturado quebra o registry do Genie Code da mesma forma que campos soltos quebravam, ou é tolerado por ser spec-compliant?"

4. **Orçamento de tokens da progressive disclosure**, citado explicitamente na especificação: ~100 tokens para `name`+`description` de cada skill (sempre carregados), corpo do `SKILL.md` recomendado abaixo de 5000 tokens / 500 linhas, resto sob demanda em `references/`. Os docs locais já orientavam a usar `references/`, mas sem essa referência numérica com fonte oficial.

5. **Ausência de regra de precedência documentada** entre skill nativa e skill customizada no mesmo domínio. Nem a documentação da Databricks nem a especificação aberta definem quem vence quando os escopos se sobrepõem - o comportamento observado nos testes abaixo (`skill-authoring` vencendo por padrão) é emergente da implementação do Genie Code, não uma regra publicada.

---

## 2026-08-20, Teste 1 - Versão Anterior (v1.3.2), Origem do Problema

Antes de qualquer revisão, a skill `skill-patterns` já existia em produção (v1.3.2). Prompt de teste genérico enviado ao Genie Code: "Preciso criar uma skill nova para otimização de códigos."

**Resultado:** a skill nativa `skill-authoring` carregou sozinha. Os Thoughts da resposta citam explicitamente uma instrução de sistema para carregar `skill-authoring` sempre que o usuário quer criar/desenvolver uma skill nova. `skill-patterns` não aparece mencionada em nenhum momento do raciocínio, apesar de esse ser exatamente o caso de uso central declarado na description dela.

**Conclusão:** com a versão original, o não-carregamento não foi um caso isolado de sorte de roteamento - foi o próprio ponto de partida da investigação. A partir daqui, decidiu-se revisar o conteúdo da skill.

## 2026-08-20 - Rework (v2.0.0)

Consulta à documentação oficial (`docs.databricks.com/aws/en/genie-code/skills`, `agentskills.io/specification`) confirmou que o Genie Code inclui skills nativas para notebooks, exploração no Unity Catalog, dashboards, pipelines e MLflow - nenhuma delas citada nominalmente como `skill-authoring`.

`skill-patterns` reescrita: YAML reduzido a `name`/`description` (únicos campos oficialmente obrigatórios), metadados (`versao`, `data`, `dominio`, `autor`) movidos para o corpo Markdown, conteúdo extenso dividido em `references/` (dominios-e-colisoes, checklist-qualidade, versionamento, troubleshooting-registry, exemplo-completo), separação explícita entre Princípios (orientação) e Procedimento (execução), fluxo de decisão em formato de diagrama.

Publicada no workspace pelo usuário para teste ao vivo.

## 2026-08-20 - Rodada 1 de Teste (v2.0.0)

Executados 4 prompts genéricos (sem citar nenhuma skill pelo nome) e 2 prompts de boundary negativo, cada um em conversa nova do Genie Code.

| # | Prompt (resumo) | Skill carregada | Observação |
|---|---|---|---|
| 1 | "Preciso criar uma skill nova para otimização de códigos." | `skill-authoring` | mesmo prompt do Teste 1; comportamento repetido mesmo após o rework |
| 2 | "Preciso criar uma skill nova pro Genie Code sobre nomear volumes no Unity Catalog." | `skill-authoring` | `skill-patterns` não aparece nem como candidata nos Thoughts |
| 3 | "Quero definir um padrão de description pra uma skill de revisão de testes automatizados." | `skill-patterns` | ambas aparecem listadas no registry dentro dos Thoughts; só `skill-patterns` é lida |
| 4 | "Como organizo o conteúdo de uma skill nova pra carregar mais rápido e não ficar gigante?" | `skill-authoring` | resposta usa estrutura genérica, não reflete o "Passo 6" específico de `skill-patterns` |
| 5 (boundary) | "Pode revisar a skill nomenclaturas e ajustar um trecho confuso da description dela?" | nenhuma das duas | correto - vai direto editar `nomenclaturas` |
| 6 (boundary) | "Cria um notebook novo pra mim, de bronze pra silver, pro projeto cvm?" | nenhuma das duas | correto - carrega `nomenclaturas`, `estrutura-notebooks`, `arquitetura-medalhao` |

**Achado adicional:** a skill nativa `skill-authoring` foi confirmada ao vivo, com description própria (traduzida): "Orientação para criar uma nova skill do Databricks Assistant: formato obrigatório do SKILL.md, layout de pastas e boas práticas de autoria. Ler esta skill sempre que o usuário quiser criar ou desenvolver uma skill nova." Essa skill não constava nominalmente na documentação consultada.

**Conclusão da Rodada 1:** em 3 dos 4 prompts positivos, `skill-authoring` carregou sozinha - incluindo repetição exata do Teste 1. `skill-patterns` só carregou no prompt que não mencionava criação, e sim padronização de description.

## 2026-08-20 - Hipótese Testada e Descartada

**Hipótese:** já que `skill-authoring` tem precedência consistente sobre o gatilho "criar skill nova", `skill-patterns` deveria ceder esse território explicitamente - boundary negativo dizendo "NAO use para criar ou estruturar skill nova do zero", reposicionando a skill só para o nicho de padronização/refinamento.

**Implementação:** description e boundaries reescritos (v3.0.0) nesse sentido, incluindo um gate `[0]` no fluxo de decisão que desviava qualquer pedido de criação para fora do escopo da skill.

**Objeção levantada pelo usuário:** um pedido real como "gostaria de criar uma nova skill no padrão xpto" combina os dois sinais - criação e referência a um padrão específico - e é exatamente o tipo de caso em que `skill-patterns` deveria poder ser acionada. Um boundary negativo que excluísse qualquer menção a "criar" bloquearia esse caso. A hipótese foi julgada como redundância negativa com efeito contrário ao pretendido.

**Decisão:** reverter para o texto de v2.0.0. Boundary negativo explícito contra o verbo "criar" descartado como estratégia.

## 2026-08-20 - Correção Aplicada (v2.1.0)

Questão levantada na revisão do boundary revertido: por que excluir "editar skills existentes" do escopo, se o conteúdo da skill (checklist, regras de description, verificação de domínio) é um padrão de conformidade aplicável independentemente de a skill ser nova ou já existir?

Avaliada como válida - a exclusão de edição/revisão era herdada da v1.3.2 original sem justificativa técnica, e contradizia o próprio dado da Rodada 1 (o caso vencedor de `skill-patterns` foi um pedido de padronização, não de criação pura).

**Correção final**, aplicada sobre o conteúdo lido diretamente do workspace (não sobre a cópia local, que havia divergido durante a tentativa revertida):

- `description`: unificada para cobrir "criar, desenvolver, elaborar, estruturar, editar, revisar ou auditar" skills, novas ou existentes
- `USAR`: "criar, editar ou revisar uma skill (nova ou existente)"
- `NAO USAR`: reduzido às duas exclusões de domínio genuinamente distinto (criar outros tipos de asset; dúvidas conceituais gerais) - removida a divisão artificial entre "editar" e "revisar"

## 2026-08-20 - Rodada 2 de Teste (v2.1.0, parcial)

Reteste de 3 dos 4 prompts positivos da Rodada 1, após publicação da v2.1.0.

| Prompt (resumo) | Rodada 1 (v2.0.0) | Rodada 2 (v2.1.0) | Mudança |
|---|---|---|---|
| "Como organizo o conteúdo de uma skill nova pra carregar mais rápido?" | só `skill-authoring` | só `skill-patterns` | `skill-patterns` passou a carregar; Thoughts cita diretamente o "Passo 6: Decidir SKILL.md vs references/" do conteúdo dela |
| "Quero definir um padrão de description pra uma skill de revisão de testes automatizados." | só `skill-patterns` | `skill-authoring` + `skill-patterns` (3 passos) | passou de carregamento isolado para co-carregamento das duas |
| "Preciso criar uma skill nova pro Genie Code sobre nomear volumes no Unity Catalog." | só `skill-authoring` | só `skill-authoring` | sem mudança - `skill-patterns` não aparece nos Thoughts |

**Observação:** o terceiro prompt da Rodada 2 permanece um caso onde `skill-authoring` carrega isolada, mesmo com o escopo de `skill-patterns` agora incluindo o verbo "criar" sem exclusão. Não testado neste log: um prompt que combine criação com um padrão específico nomeado - o cenário exato usado como contra-exemplo para descartar a hipótese da seção anterior.

## Pendências

1. Reteste do Teste 1/prompt 1 da Rodada 1 ("otimização de códigos") e do par de prompts de boundary negativo (5 e 6), para confirmar que a correção não introduziu regressão.
2. Teste dedicado ao cenário "criar skill no padrão X" - ainda não executado, é o caso que motivou a correção de escopo mas não foi verificado diretamente.
3. Amostra pequena (7 prompts distintos até aqui, 1 sessão) - resultados registrados como observação de comportamento, não como regra de triggering comprovada.
4. Testar isoladamente o campo `metadata:` (mapa YAML estruturado, spec-compliant) como variável própria - ainda não diferenciado de "campos soltos no root do YAML" nos testes locais existentes.
5. Verificar se as 12 skills locais atuais têm `name` idêntico ao nome da própria pasta, conforme regra da especificação oficial - não conferido neste log.

---

**Investigador:** Pedro Silva (testes ao vivo) + assistente (leitura de documentação, escrita das revisões, análise dos resultados)
**Ferramentas:** Databricks CLI (leitura ao vivo de `SKILL.md`), Genie Code (execução dos prompts de teste)
