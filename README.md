# Skills Personalizadas para Databricks Genie Code

User Skills de engenharia de dados para o Databricks Genie Code. Coleção de padrões práticos cobrindo arquitetura, qualidade de código, workflow Git e documentação técnica.

---

## O Que São Estas Skills

Skills do Databricks Genie Code existem em três níveis:

1. **Skills nativas da plataforma** — Databricks mantém skills para notebooks, Unity Catalog, dashboards, pipelines e MLflow
2. **Skills de workspace/time** — Compartilhadas por um time via workspace `/Shared/`
3. **Skills de usuário** — Individuais, no diretório pessoal `/Users/<email>/.assistant/skills/`

Este projeto opera no **nível de usuário por escolha deliberada**, focando em padrões de engenharia de dados não cobertos pelas skills nativas: decisões arquiteturais, qualidade de código, workflow de desenvolvimento e documentação técnica.

---

## Skills Disponíveis

Organizadas por domínio de atuação. Cada skill tem verbo de ação único e boundaries negativos explícitos para prevenir sobreposição.

### architecture — Arquitetura e Decisões Técnicas

* **arquitetura-medalhao**: Escolher estratégia de gravação (DELETE+APPEND vs replaceWhere vs MERGE), definir idempotência e reprocessabilidade, decidir versionamento de regras, avaliar Jobs vs Declarative Pipelines. Resolve a tensão entre dado bruto confiável (Bronze) e dado refinado útil (Gold) através de transformações determinísticas entre camadas.

* **escolha-sql-pyspark**: Decidir entre SQL ou PySpark para transformações baseado em complexidade, performance e manutenibilidade. Ambas compilam para o mesmo motor Spark, mas cada uma tem casos de uso onde se destaca.

### code-quality — Qualidade e Revisão de Código

* **revisao-codigo-quatro-frentes**: Revisar código existente em 4 dimensões — (1) correção semântica, (2) premissas ocultas, (3) código morto, (4) custo evitável. Gera relatório estruturado + plano de mudanças proposto, nunca edita código sozinho. Orientado para pipelines de dados onde premissas não validadas podem corromper dados a jusante.

* **guardrails-pipelines**: Implementar validações de qualidade, integridade estrutural, reconciliação quantitativa e resiliência operacional. Pipelines production-grade devem falhar de forma previsível, rastreável e recuperável — guardrails são contratos estruturais, não validações adicionadas no final.

### code-structure — Estrutura de Código

* **estrutura-notebooks**: Estrutura para notebooks novos do zero — ordem fixa de células iniciais (DOCUMENTAÇÃO, CARREGAR CONFIGURAÇÕES, INICIALIZAR ANOS A PROCESSAR, IMPORTS), separação de responsabilidades por célula, formato de arquivo (.py vs .ipynb para versionamento Git limpo).

### data-modeling — Modelagem de Dados

* **unity-catalog**: Organizar schemas, tabelas e volumes no Unity Catalog — padrões de nomenclatura UC, organização por camadas (bronze/silver/gold) e projetos. Numeração de tabelas (XXX formato de 3 dígitos) para rastreabilidade visual entre camadas.

### naming — Nomenclatura

* **nomenclaturas**: Convenções de nomenclatura para assets novos — notebooks, tabelas, DataFrames, variáveis, pastas. Numeração com rastreabilidade entre camadas (101_nome_base → 201_nome_base → 301_nome_base), princípio DRY, padrões de caixa (snake_case, PascalCase).

### documentation — Documentação

* **padrao-escrita**: Tom sobrio e técnico para documentação versionada (README, arquitetura, evolução, células markdown, comentários de código). Verbos no presente, sem emojis/ícones, nível de registro adequado por tipo de documento.

* **protocolo-atualizacao**: Mapear quais documentações (README, arquitetura, dicionário de dados, evolução do projeto) precisam atualização após mudanças de código ou arquitetura. Matriz de impactos para sincronizar documentação com implementação.

### version-control — Controle de Versão

* **git-workflow**: Divisão de commits por escopo técnico (bisect/revert/review), staging parcial (git add -p), mensagens de commit neutras, quando usar amend. Critério técnico > critério estético — decisões baseadas em facilidade de bisect/revert/review, não em "como parece melhor no histórico".

### project-management — Gestão de Projetos

* **ciclo-eda-validacao**: Organizar ciclo de descoberta e verificação em projetos de dados — QUANDO criar EDA vs validação, ONDE colocar (estrutura de pastas dedicadas), COMO registrar fluxo completo (ACHADO → DECISÃO → CÓDIGO → VALIDAÇÃO), COMO conectar etapas no arquivo de evolução do projeto. EDA não é camada de pipeline — é superfície de investigação ortogonal às transformações.

### workflow — Workflow e Meta

* **skill-patterns**: Criar/editar skills seguindo padrão de qualidade — validação de description (domínio de atuação, colisão com skills nativas), checklist de qualidade, verificação DRY contra skills globais e locais, organização de conteúdo entre SKILL.md e references/. Cabecalho YAML mínimo (apenas name + description), metadados no corpo Markdown.

---

## Instalação

Skills podem ser criadas manualmente no workspace. Este repositório usa Databricks Git folder, método recomendado pela documentação para versionamento, com as pastas de skill na raiz.

### Pré-condição

O caminho `/Users/<seu-email>/.assistant/skills/` pode não existir antes da instalação.

### Procedimento

1. Crie um Databricks Git folder apontando para este repositório:
   - Nome do folder: `skills`
   - Caminho de criação: `/Users/<seu-email>/.assistant/`
   - URL do repositório: `https://github.com/<usuario>/databricks-genie-skills`

2. As pastas de skill ficam na raiz do repositório, então os caminhos resolvem como:
   ```
   /Users/<seu-email>/.assistant/skills/nomenclaturas/SKILL.md
   /Users/<seu-email>/.assistant/skills/estrutura-notebooks/SKILL.md
   ...
   ```

3. O registry lê o diretório em tempo real. Alterações no Git folder (via pull) entram em vigor imediatamente, sem etapa de deploy.

4. Teste o triggering em uma nova conversa:
   ```
   "revisar este notebook"          -> triggera revisao-codigo-quatro-frentes
   "criar um notebook novo"         -> triggera estrutura-notebooks
   "nomear esta tabela"             -> triggera nomenclaturas
   "commitar estas mudanças"        -> triggera git-workflow
   ```

---

## Exemplos de Uso

### Revisão de Código
```
Usuário: "revisar este notebook antes de produção"
Genie: [carrega revisao-codigo-quatro-frentes]
       Revisa em 4 dimensões: correção, premissas ocultas, código morto, custo
```

### Convenção de Nomenclatura
```
Usuário: "nomear este notebook bronze de dados CVM"
Genie: [carrega nomenclaturas]
       Sugere: 001_ingestao_cvm (segue numeração + DRY + snake_case)
```

---

## Princípios de Design

Estas skills seguem princípios de clareza e organização:

1. **Verbo de Ação Distinto** — Cada skill tem um verbo claro (DEFINIR, CRIAR, IMPLEMENTAR, REVISAR, ATUALIZAR, COMMITAR, DECIDIR, ORGANIZAR) para facilitar entendimento humano da separação de responsabilidades
2. **Boundaries Negativos Explícitos** — Cada description especifica o que ela NÃO faz ("NÃO use para...") como boa prática de documentação para clareza de escopo
3. **Especificidade Máxima** — Clara sobre QUANDO triggar e quando NÃO triggar

---

## Notas Técnicas — Processo de Desenvolvimento

Durante o desenvolvimento destas skills, investigamos requisitos não documentados do sistema de triggering do Databricks Genie Code.

### Investigação: Requisitos de Triggering

Skills customizadas não triggavam inicialmente. Após investigar o problema, formulei 3 hipóteses sobre requisitos não documentados do sistema de matching:

1. **ASCII puro obrigatório** — Caracteres não-ASCII (acentos, ç, etc.) quebrariam o matching
2. **Boundaries negativos obrigatórios** — Falta de "NÃO use para..." causaria false-positives
3. **Verbo imperativo obrigatório** — Formato específico seria necessário

Antes de publicar conclusões, testei cada hipótese isoladamente:

**Teste 1 (ASCII):** Adicionei 28 caracteres acentuados na description → Skill triggou normalmente  
**Teste 2 (Boundaries):** Removi "NÃO use para..." → Apenas skill correta triggou  
**Teste 3 (Verbo imperativo):** Removi "Use APENAS ao" → Skill triggou normalmente

**Resultado:** Todas as três hipóteses foram refutadas empiricamente.

### Conclusão

O sistema de triggering é mais robusto que o esperado:
* Suporta UTF-8 completo (acentos funcionam)
* Compreensão semântica sofisticada (não é keyword matching simples)
* Formato flexível (reconhece intent sem estrutura rígida)

Os padrões de design (verbo distinto, boundaries negativos, especificidade) permanecem como **boas práticas de clareza**, não requisitos técnicos.

**Lição:** Observar padrões em skills funcionais não prova causalidade. Testar empiricamente é fundamental.

Protocolo completo (3 testes isolados, evidências visuais, timing) documentado em [docs/protocolo_teste_empirico.md](docs/protocolo_teste_empirico.md). Reflexões sobre o processo em [docs/lessons_learned.md](docs/lessons_learned.md).

---

### Padrão de Referência a Skills em Custom Instructions

Este projeto adota o padrão de listar skills dentro do `.assistant_instructions.md` (mencionando `@nome-da-skill` + descrição + regra de quando aplicar) conforme exemplo do repositório oficial de referência da Databricks Solutions.

**Fonte:** Repositório [`databricks-solutions/genie-code-skills-demo`](https://github.com/databricks-solutions/genie-code-skills-demo/blob/b5bc5d25a4b624959dfa19a62ad83f5d1119dca5/instructions/.assistant_instructions.md) (commit `b5bc5d2`, acessado em agosto de 2026), arquivo `instructions/.assistant_instructions.md`, seção "Approach A: Direct Skills Reference (no MCP)":

> When creating or modifying SDP pipeline tables, always use the following skills:
> 
> 1. **@table-governance** -- Apply table and column documentation standards (COMMENT, TBLPROPERTIES, column descriptions, UC tags). Apply this first for every table.
> 2. **@sdp-basics** -- Apply SDP naming conventions, audit columns, data quality constraints, and SQL formatting
> [...]
> 
> For every table you create:
> - Always apply the table-governance and sdp-basics skills
> - If the table contains customer or personal data, also apply the pii-management skill

**Nota Importante:** Este padrão NÃO está documentado nas páginas oficiais de documentação (`docs.databricks.com`). O repositório `databricks-solutions/genie-code-skills-demo` contém o aviso "Databricks support does not cover this content" — é um repositório de referência mantido pelo time Databricks Solutions, não documentação oficial suportada.

---

## Documentação

* **[docs/investigation_log.md](docs/investigation_log.md)** — Investigação técnica inicial sobre triggering (hipóteses formuladas, tentativas, achados)
* **[docs/investigation_log_skill_patterns.md](docs/investigation_log_skill_patterns.md)** — Log de desenvolvimento da skill-patterns (colisão com skills nativas, validação de description, princípio DRY)
* **[docs/protocolo_teste_empirico.md](docs/protocolo_teste_empirico.md)** — Protocolo de testes sobre requisitos de triggering (3 testes isolados, evidências visuais, timing, resultados)
* **[docs/lessons_learned.md](docs/lessons_learned.md)** — Lições sobre testar hipóteses em sistemas opacos, falsificabilidade, armadilhas de confirmação
* **Skills Individuais** — Cada `SKILL.md` contém orientação específica do domínio (português, padrões de implementação detalhados)

---

## Referências

### Documentação Databricks
* [Extend Genie Code with agent skills (Databricks)](https://docs.databricks.com/aws/en/genie-code/skills)
* [Agent skills for AI coding assistants (Databricks)](https://docs.databricks.com/aws/en/agent-skills/)

### Especificação Agent Skills
* [Agent Skills Specification](https://agentskills.io/specification.md)
* [Best Practices for Skill Creators](https://agentskills.io/skill-creation/best-practices.md)
* [Optimizing Descriptions](https://agentskills.io/skill-creation/optimizing-descriptions.md)

---

## Stack

- **Databricks**: Genie Code (AI coding assistant), Databricks Git folders
- **Formato**: Agent Skills Specification (YAML frontmatter + Markdown)
- **Versionamento**: Git
- **Documentação**: Markdown

---

## Sobre o Projeto

Projeto de portfólio demonstrando padrões práticos de engenharia de dados e rigor no processo técnico (investigação, teste de hipóteses, documentação).

As skills cobrem o ciclo completo de desenvolvimento: decidir arquitetura (arquitetura-medalhao, escolha-sql-pyspark) → implementar com qualidade (guardrails-pipelines, estrutura-notebooks) → revisar (revisao-codigo-quatro-frentes) → versionar (git-workflow) → documentar (padrao-escrita, protocolo-atualizacao) → criar novas skills (skill-patterns).

Este projeto foi desenvolvido com apoio do Genie Code e do Claude Code.

**Autor:** Pedro Silva  
**Contexto:** Transição - Analista de Dados → Engenheiro de Dados  
**Ambiente:** Databricks (Genie Code, Git folders)

---

## Licença

Licença MIT - Veja arquivo LICENSE para detalhes
