# Versionamento de Skills

Ao atualizar uma skill existente, incrementar a `Versao` (linha de metadados no corpo, nunca no YAML) seguindo semantic versioning.

## MAJOR (X.0.0) — mudanca que quebra compatibilidade

- Altera o triggering (muda o verbo de acao ou o escopo principal da description)
- Remove funcionalidade documentada
- Muda boundaries de forma incompativel com o uso anterior

## MINOR (x.Y.0) — adiciona funcionalidade

- Novos exemplos ou secoes
- Expansao de escopo compativel (nao quebra o uso anterior)
- Melhora de orientacoes existentes

## PATCH (x.y.Z) — correcoes

- Typos ou erros de formatacao
- Clareza de texto, sem mudanca de comportamento
- Exemplos melhores para a mesma funcionalidade

## Regra Fixa

Sempre sincronizar o campo `Data` com a nova `Versao` na linha de metadados do corpo Markdown.

**Exemplos:**
- Skill em Versao 1.2.3, adiciona nova secao de troubleshooting → Versao 1.3.0
- Skill em Versao 1.2.3, corrige um typo → Versao 1.2.4
- Skill muda dominio ou description de forma incompativel com o comportamento anterior → Versao 2.0.0
