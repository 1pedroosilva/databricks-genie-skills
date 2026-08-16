# Skills Personalizadas para Databricks Genie Code

User Skills para o Databricks Genie Code. Projeto de troubleshooting técnico: como fazer o Skill Registry realmente carregar skills customizadas.

## O Problema

User Skills são extensões do Genie Code que você cria no seu workspace (`/Users/<email>/.assistant/skills/`). A documentação oficial explica o formato (YAML frontmatter + Markdown), mas não explica como o Registry decide **qual skill carregar** quando você faz uma pergunta.

Criei skills seguindo a doc e elas simplesmente não triggavam. Existiam no workspace, mas o Genie Code nunca as usava. Depois de investigar, formulei 3 hipóteses sobre requisitos não documentados.

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

Protocolo completo (3 testes, evidências visuais, timing) documentado em `.backups/protocolo_teste_empirico.md`.

## Skills

| Skill | Verbo de Ação | Propósito |
|-------|---------------|-----------|
| `nomenclaturas` | DEFINIR | Convenções de nomenclatura para assets novos |
| `estrutura-notebooks` | CRIAR | Estrutura para notebooks novos do zero |
| `resiliencia-operacional` | IMPLEMENTAR | Padrões de resiliência em código novo |
| `revisao-codigo-quatro-frentes` | REVISAR/AUDITAR | Corretude de código existente (4 dimensões) |
| `unity-catalog` | CRIAR | Schemas, tabelas e volumes no UC |
| `protocolo-atualizacao` | ATUALIZAR | Documentação após mudanças de código |
| `git-workflow` | COMMITAR | Divisão de commits, staging parcial, mensagens de commit |

**Separação de Responsabilidades:**  
Cada skill possui um verbo de ação único e boundaries negativos explícitos para prevenir sobreposição:

```
DEFINIR ≠ CRIAR ≠ IMPLEMENTAR ≠ REVISAR ≠ ATUALIZAR ≠ COMMITAR
```

## Instalação

1. Copie as pastas de skills para seu workspace Databricks:
```
/Workspace/Users/<seu-email>/.assistant/skills/
├── nomenclaturas/
├── estrutura-notebooks/
├── resiliencia-operacional/
├── revisao-codigo-quatro-frentes/
├── unity-catalog/
├── protocolo-atualizacao/
└── git-workflow/
```

2. Aguarde 5-10 minutos para o registry do Genie Code indexar as skills

3. Teste o triggering em uma nova conversa:
```
"revisar este notebook"                          -> triggera revisao-codigo-quatro-frentes
"criar um notebook novo"                         -> triggera estrutura-notebooks
"nomear esta tabela"                             -> triggera nomenclaturas
"commitar estas mudanças"                        -> triggera git-workflow
"quais skills pessoais estão no skill registry?" -> triggera e lista todas as skills
```

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
       Sugere: 001_bronze_cvm_raw (segue numeração + DRY + snake_case)
```

## Princípios de Design

Estas skills seguem princípios de clareza e organização:

1. **Verbo de Ação Distinto** - Cada skill tem um verbo claro (DEFINIR, CRIAR, IMPLEMENTAR, REVISAR, ATUALIZAR, COMMITAR) para facilitar entendimento humano da separação de responsabilidades
2. **Boundaries Negativos Explícitos** - Cada description especifica o que ela NÃO faz ("NÃO use para...") como boa prática de documentação (testes mostraram que não são tecnicamente obrigatórios, mas ajudam clareza)
3. **Especificidade Máxima** - Clara sobre QUANDO triggar e quando NÃO triggar

**Nota:**  
Testes (documentados em `.backups/protocolo_teste_empirico.md`) mostraram que o sistema de triggering é mais robusto que eu pensava inicialmente. Requisitos como "ASCII puro obrigatório" ou "boundaries negativos tecnicamente necessários" foram testados e refutados. Esses padrões permanecem como boas práticas de clareza, não requisitos técnicos.

## Scripts de Validação

### Verificar Limite de Tamanho da Description
```python
description = "sua description aqui"
print(f"{len(description)}/1024 chars" if len(description) <= 1024 else "ACIMA DO LIMITE")
```

## Métricas de Qualidade

| Métrica | Status |
|---------|--------|
| Boundaries negativos explícitos (clareza) | 7/7 (100%) |
| Verbos de ação distintos (organização) | 6/7 (86%) |
| Tamanho < 1024 chars | 7/7 (100%) |
| Triggering correto | 7/7 (100%) |
| Testes (3 hipóteses testadas) | 3/3 (100%) |

## Documentação

* **[investigation_log.md](investigation_log.md)** - Investigação técnica inicial e hipóteses formuladas
* **[.backups/protocolo_teste_empirico.md](.backups/protocolo_teste_empirico.md)** - Protocolo de testes (3 testes isolados, evidências visuais, resultados)
* **[lessons_learned.md](lessons_learned.md)** - Lições sobre testar hipóteses antes de publicar conclusões técnicas
* **Skills Individuais** - Cada `SKILL.md` contém orientação específica do domínio (português, padrões de implementação detalhados)

## Referências

### Documentação Databricks
* [Extend Genie Code with agent skills (Databricks)](https://docs.databricks.com/aws/en/genie-code/skills)
* [Agent skills for AI coding assistants (Databricks)](https://docs.databricks.com/aws/en/agent-skills/)

### Especificação Agent Skills
* [Agent Skills Specification](https://agentskills.io/specification.md)
* [Best Practices for Skill Creators](https://agentskills.io/skill-creation/best-practices.md)
* [Optimizing Descriptions](https://agentskills.io/skill-creation/optimizing-descriptions.md)

## Stack

- **Databricks**: Serverless Compute, Unity Catalog, Delta Lake, Databricks Repos
- **Linguagens**: Python (PySpark), SQL (Databricks SQL)
- **Padrões**: Medallion architecture, idempotência, retry com backoff, checkpointing

## Contribuindo

Projeto de portfólio demonstrando investigação técnica e análise de causa raiz.

Este projeto foi desenvolvido com apoio do Genie Code e do Claude Code. Os padrões documentados -- estratégias de gravação, antipadrões de resiliência, achados da skill de revisão de código -- vêm de uma auditoria real feita no workspace: consulta a tabelas, comparação entre notebooks, e ao menos uma hipótese de bug descartada depois de checar os dados. O processo completo está documentado em `investigation_log.md`.

**Autor:** Pedro Silva  
**Contexto:** Transição - Analista de Dados → Engenheiro de Dados  
**Ambiente:** Databricks (AWS, Serverless, Unity Catalog)

## Licença

Licença MIT - Veja arquivo LICENSE para detalhes

---

**Última atualização:** 2026-08-15  
**Versão:** 1.0.0
