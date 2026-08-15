# Skills Personalizadas para Databricks Genie Code

User Skills para o Databricks Genie Code. Projeto de investigacao tecnica sobre como fazer o Skill Registry realmente carregar skills customizadas.

## O Problema

User Skills sao extensoes do Genie Code que voce cria no seu workspace (`/Users/<email>/.assistant/skills/`). A documentacao oficial explica o formato (YAML frontmatter + Markdown), mas nao explica como o Registry decide **qual skill carregar** quando voce faz uma pergunta.

Criei skills seguindo a doc e elas simplesmente nao triggavam. Existiam no workspace, mas o Genie Code nunca as usava. Depois de testar varias hipoteses, descobri 3 coisas que a doc nao deixa explicito:

1. **O matcher e sensivel a encoding** - Caracteres fora do ASCII basico (acentos, ç, etc) no campo `description` quebram o matching. Skill fica invisivel.

2. **Boundaries negativos importam** - Se sua description nao diz explicitamente o que ela NAO faz, queries ambiguas triggam multiplas skills ao mesmo tempo (nenhuma carrega direito).

3. **Verbos de acao ajudam especificidade** - Uma skill de "revisar codigo" e outra de "criar codigo novo" precisam de verbos diferentes na description (REVISAR vs CRIAR), senao competem.

Documentei o processo de investigacao completo em [INVESTIGATION_LOG.md](INVESTIGATION_LOG.md) - problema, hipoteses testadas, causa raiz, solucao.

## Skills

| Skill | Verbo de Acao | Proposito |
|-------|---------------|-----------|
| `nomenclaturas` | DEFINIR | Convencoes de nomenclatura para assets novos |
| `estrutura-notebooks` | CRIAR | Estrutura para notebooks novos do zero |
| `resiliencia-operacional` | IMPLEMENTAR | Padroes de resiliencia em codigo novo |
| `revisao-codigo-quatro-frentes` | REVISAR/AUDITAR | Corretude de codigo existente (4 dimensoes) |
| `unity-catalog` | CRIAR | Schemas, tabelas e volumes no UC |
| `protocolo-atualizacao` | ATUALIZAR | Documentacao apos mudancas de codigo |

**Separacao de Responsabilidades:**  
Cada skill possui um verbo de acao unico e boundaries negativos explicitos para prevenir sobreposicao:

```
DEFINIR ≠ CRIAR ≠ IMPLEMENTAR ≠ REVISAR ≠ ATUALIZAR
```

## Instalacao

1. Copie as pastas de skills para seu workspace Databricks:
```
/Workspace/Users/<seu-email>/.assistant/skills/
├── nomenclaturas/
├── estrutura-notebooks/
├── resiliencia-operacional/
├── revisao-codigo-quatro-frentes/
├── unity-catalog/
└── protocolo-atualizacao/
```

2. Aguarde 5-10 minutos para o registry do Genie Code indexar as skills

3. Teste o triggering em uma nova conversa:
```
"revisar este notebook"                          -> triggera revisao-codigo-quatro-frentes
"criar um notebook novo"                         -> triggera estrutura-notebooks
"nomear esta tabela"                             -> triggera nomenclaturas
"quais skills pessoais estão no skill registry?" -> triggera e lista todas as skills
```

## Exemplos de Uso

### Revisao de Codigo
```
Usuario: "revisar este notebook antes de producao"
Genie: [carrega revisao-codigo-quatro-frentes]
       Revisa em 4 dimensoes: correcao, premissas ocultas, codigo morto, custo
```



### Convencao de Nomenclatura
```
Usuario: "nomear este notebook bronze de dados CVM"
Genie: [carrega nomenclaturas]
       Sugere: 001_bronze_cvm_raw (segue numeracao + DRY + snake_case)
```

## Principios de Design

Estas skills seguem principios rigorosos de design aprendidos atraves de investigacao tecnica:

1. **Verbo de Acao Unico** - Cada skill tem um verbo (DEFINIR, CRIAR, IMPLEMENTAR, DECIDIR, REVISAR, ATUALIZAR)
2. **Boundaries Negativos Explicitos** - Cada description especifica o que ela NAO faz ("NAO use para...")
3. **ASCII Puro** - Todas as descriptions usam ASCII puro (sem acentos, caracteres especiais)
4. **Especificidade Maxima** - Clara sobre QUANDO triggar e quando NAO triggar

**Por Que Isso Importa:**  
Sem boundaries explicitos, queries como "revisar este notebook" podem triggar 4-5 skills parcialmente relacionadas em vez da skill correta. Veja [INVESTIGATION_LOG.md](INVESTIGATION_LOG.md) para a investigacao tecnica completa.

## Scripts de Validacao

### Verificar Conformidade ASCII
```python
description = "sua description aqui"
non_ascii = [char for char in description if ord(char) > 127]
print(f"Non-ASCII: {len(non_ascii)}" if non_ascii else "OK")
```

### Verificar Limite de Tamanho
```python
print(f"{len(description)}/1024 chars" if len(description) <= 1024 else "ACIMA DO LIMITE")
```

## Metricas de Qualidade

| Metrica | Status |
|---------|--------|
| Descriptions ASCII puro | 6/6 (100%) |
| Boundaries negativos explicitos | 6/6 (100%) |
| Verbos de acao unicos | 6/6 (100%) |
| Tamanho < 1024 chars | 6/6 (100%) |
| Triggering correto | 6/6 (100%) |

## Documentacao

* **[INVESTIGATION_LOG.md](INVESTIGATION_LOG.md)** - Investigacao tecnica completa: descoberta do problema, teste de hipoteses, analise de causa raiz, e licoes aprendidas
* **Skills Individuais** - Cada `SKILL.md` contem orientacao especifica do dominio (portugues, padroes de implementacao detalhados)

## Referencias

### Documentacao Databricks
* [User Skills (Databricks)](https://docs.databricks.com/en/generative-ai/agent-framework/user-skills.html)
* [Create User Skills (Databricks)](https://docs.databricks.com/en/generative-ai/agent-framework/create-user-skills.html)

### Especificacao Agent Skills
* [Agent Skills Specification](https://agentskills.io/specification.md)
* [Best Practices for Skill Creators](https://agentskills.io/skill-creation/best-practices.md)
* [Optimizing Descriptions](https://agentskills.io/skill-creation/optimizing-descriptions.md)

## Stack

- **Databricks**: Serverless Compute, Unity Catalog, Delta Lake, Databricks Repos
- **Linguagens**: Python (PySpark), SQL (Databricks SQL)
- **Padroes**: Medallion architecture, idempotencia, retry com backoff, checkpointing

## Contribuindo

Projeto de portfolio demonstrando investigacao tecnica e analise de causa raiz.

Este projeto foi desenvolvido com apoio do Genie Code e do Claude Code. Os padroes documentados -- estrategias de gravacao, antipadroes de resiliencia, achados da skill de revisao de codigo -- vem de uma auditoria real feita no workspace: consulta a tabelas, comparacao entre notebooks, e ao menos uma hipotese de bug descartada depois de checar os dados. O processo completo esta documentado em `INVESTIGATION_LOG.md`.

**Autor:** Pedro Silva  
**Contexto:** Transicao - Analista de Dados → Engenheiro de Dados  
**Ambiente:** Databricks (AWS, Serverless, Unity Catalog)

## Licenca

Licenca MIT - Veja arquivo LICENSE para detalhes

---

**Ultima atualizacao:** 2026-08-14  
**Versao:** 1.0.0
