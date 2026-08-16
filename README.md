# Skills Personalizadas para Databricks Genie Code

User Skills para o Databricks Genie Code. Projeto de troubleshooting técnico: como fazer o Skill Registry realmente carregar skills customizadas.

---

## O Problema

User Skills são extensões do Genie Code que você cria no seu workspace (`/Users/<email>/.assistant/skills/`). A documentação oficial explica o formato (YAML frontmatter + Markdown), mas não explica como o Registry decide **qual skill carregar** quando você faz uma pergunta.

Criei skills seguindo a doc e elas simplesmente não triggavam. Existiam no workspace, mas o Genie Code nunca as usava. Depois de investigar, formulei 3 hipóteses sobre requisitos não documentados.

---

## Testes

Antes de publicar, testei cada hipótese:

**Teste 1 (ASCII obrigatório):** Adicionei 28 caracteres acentuados na description  
**Resultado:** Skill triggou normalmente. Hipótese errada.

**Teste 2 (Boundaries negativos obrigatórios):** Removi "NÃO use para..."  
**Resultado:** Apenas skill correta triggou, sem false-positives. Hipótese errada.

**Teste 3 (Verbo imperativo obrigatório):** Removi "Use APENAS ao" do início  
**Resultado:** Skill triggou normalmente. Hipótese errada.

### Descoberta

**O sistema de triggering é mais robusto que eu pensava:**
- Suporta UTF-8 completo (acentos funcionam)
- Compreensão semântica sofisticada (não é keyword matching simples)
- Formato flexível (reconhece intent sem estrutura rígida)

**Lição:** Observar padrões em skills funcionais não prova causalidade. Testar é fundamental.

Protocolo completo (3 testes, evidências visuais, timing) documentado em [docs/protocolo_teste_empirico.md](docs/protocolo_teste_empirico.md).

---

## Skills

| Skill | Verbo de Ação | Propósito |
|-------|---------------|-----------|
| `arquitetura-medalhao` | DECIDIR | Estratégia arquitetural para pipelines novos |
| `escolha-sql-pyspark` | DECIDIR | Entre SQL ou PySpark para transformações |
| `estrutura-notebooks` | CRIAR (notebooks) | Estrutura para notebooks novos do zero |
| `git-workflow` | COMMITAR | Divisão de commits, staging parcial, mensagens de commit |
| `guardrails-pipelines` | IMPLEMENTAR | Validações e resiliência em pipelines |
| `nomenclaturas` | DEFINIR | Convenções de nomenclatura para assets novos |
| `padrao-escrita` | PADRONIZAR | Escrita técnica em documentação |
| `protocolo-atualizacao` | ATUALIZAR | Documentação após mudanças de código |
| `revisao-codigo-quatro-frentes` | REVISAR/AUDITAR | Corretude de código existente (4 dimensões) |
| `skill-patterns` | CRIAR (skills) | Padrões para criação de skills novas |
| `unity-catalog` | CRIAR (UC assets) | Schemas, tabelas e volumes no UC |

**Separação de Responsabilidades:**  
Cada skill possui um verbo de ação único e boundaries negativos explícitos para prevenir sobreposição:

```
DEFINIR ≠ CRIAR ≠ REVISAR ≠ ATUALIZAR ≠ COMMITAR ≠ PADRONIZAR ≠ IMPLEMENTAR ≠ DECIDIR
```

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

1. **Verbo de Ação Distinto** - Cada skill tem um verbo claro (DEFINIR, CRIAR, IMPLEMENTAR, REVISAR, ATUALIZAR, COMMITAR, DECIDIR) para facilitar entendimento humano da separação de responsabilidades
2. **Boundaries Negativos Explícitos** - Cada description especifica o que ela NÃO faz ("NÃO use para...") como boa prática de documentação (testes mostraram que não são tecnicamente obrigatórios, mas ajudam clareza)
3. **Especificidade Máxima** - Clara sobre QUANDO triggar e quando NÃO triggar

**Nota:**  
Testes (documentados em [docs/protocolo_teste_empirico.md](docs/protocolo_teste_empirico.md)) mostraram que o sistema de triggering é mais robusto que eu pensava inicialmente. Requisitos como "ASCII puro obrigatório" ou "boundaries negativos tecnicamente necessários" foram testados e refutados. Esses padrões permanecem como boas práticas de clareza, não requisitos técnicos.

---

## Documentação

* **[docs/investigation_log.md](docs/investigation_log.md)** - Investigação técnica inicial e hipóteses formuladas
* **[docs/protocolo_teste_empirico.md](docs/protocolo_teste_empirico.md)** - Protocolo de testes (3 testes isolados, evidências visuais, resultados)
* **[docs/lessons_learned.md](docs/lessons_learned.md)** - Lições sobre testar hipóteses antes de publicar conclusões técnicas
* **Skills Individuais** - Cada `SKILL.md` contém orientação específica do domínio (português, padrões de implementação detalhados)

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

Projeto de portfólio demonstrando investigação técnica e análise de causa raiz.

Este projeto foi desenvolvido com apoio do Genie Code e do Claude Code. A investigação seguiu método empírico: observação do comportamento do Skill Registry, formulação de 3 hipóteses sobre requisitos não documentados, e teste isolado de cada uma. As 3 hipóteses foram refutadas, revelando que o sistema de triggering é mais robusto e flexível que o inicialmente presumido. O processo completo está documentado em [docs/investigation_log.md](docs/investigation_log.md) e [docs/protocolo_teste_empirico.md](docs/protocolo_teste_empirico.md).

**Autor:** Pedro Silva  
**Contexto:** Transição - Analista de Dados → Engenheiro de Dados  
**Ambiente:** Databricks (Genie Code, Git folders)

---

## Licença

Licença MIT - Veja arquivo LICENSE para detalhes
