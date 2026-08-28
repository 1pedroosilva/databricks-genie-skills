# Dominios e Colisao com Skills Nativas

## Skills Nativas do Genie Code (nao editaveis)

A documentacao oficial da Databricks confirma que o Genie Code vem com skills nativas para estes fluxos de trabalho:

- Escrever codigo em notebooks
- Explorar dados no Unity Catalog
- Construir dashboards
- Criar pipelines
- Trabalhar com MLflow

Essas skills nao sao editaveis pelo usuario. A documentacao nao define uma regra formal de precedencia quando uma skill custom se sobrepoe a uma dessas — a responsabilidade de evitar o conflito e do autor da skill custom, atraves de escopo bem delimitado e boundaries negativos explicitos na description.

**Verificacao obrigatoria ao criar skill nova:** o escopo pretendido toca em notebooks, Unity Catalog, dashboards, pipelines ou MLflow? Se sim, a description precisa deixar claro o recorte especifico que a skill custom cobre (ex.: "convencoes de nomenclatura para notebooks novos", nao "notebooks" de forma generica) para reduzir competicao com a skill nativa equivalente.

## Tabela de Dominios (Skills Locais Existentes)

| Dominio | Skill | Escopo |
|---------|-------|--------|
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
| project-management | ciclo-eda-validacao | Organizacao de EDA e validacoes |

## Principio DRY

Um dominio pode ter multiplas skills, desde que os escopos sejam claramente distintos e nao-sobrepostos. A desambiguacao acontece por description rica (vocabulario amplo) e boundaries negativos claros — nao por regra de prioridade fixa.

**Regra de decisao:**
- Skill global ja cobre o topico por completo → nao criar user skill, usar a global
- Skill global cobre parte do topico → user skill deve complementar (adicionar preferencias especificas), nunca redefinir o que a global ja ensina
- Skill global nao existe para o topico → ok criar user skill
- User skill ja existe com escopo identico → editar a existente, nao criar nova
- User skill tem verbo de acao identico mas escopo disjunto → ok criar, com boundaries claros na description
