# Exemplo Completo de Verificacao DRY

## Cenario

Usuario pede: "criar uma skill sobre estrutura de notebooks"

## Passo A — Skills Globais

Consultar a skill global equivalente (ex.: uma skill global de autoria de skills, se existir).

Resultado possivel: a skill global cobre o formato YAML + Markdown de skills em geral, mas nao cobre estrutura de notebooks especificamente → topico nao duplicado nesse nivel.

## Passo B — User Skills

Listar as pastas em `.assistant/skills/`.

Resultado possivel: ja existe uma pasta `notebook-structure/` → a decisao correta e EDITAR a skill existente, nao criar uma nova.

## Passo C — Documentacao Oficial

Consultar a documentacao oficial da Databricks sobre boas praticas de notebooks, para garantir que a skill (nova ou editada) reflete orientacao atualizada e nao contradiz o que a plataforma recomenda.

## Decisao Final

Editar `notebook-structure/SKILL.md` em vez de criar uma skill nova — evita duplicidade de escopo e mantem uma unica fonte de verdade para o topico.

## Exemplo de Description Validada

```yaml
description: Use ao criar, desenvolver, elaborar ou definir skills novas para Databricks Genie Code. Define estrutura obrigatoria (YAML frontmatter + Markdown), padroes de description (vocabulario rico com sinonimos, boundaries negativos explicitos, contexto detalhado, menos de 1024 chars ASCII), checklist de qualidade e organizacao por dominios. NAO use para editar skills existentes, criar outros tipos de asset, ou responder perguntas sobre como skills funcionam.
```

**Validacao aplicada a este exemplo:**
- Tamanho dentro do limite (< 1024 chars)
- ASCII puro
- Vocabulario rico: "criar, desenvolver, elaborar, definir"
- Boundaries negativos presentes
- Dominio: workflow
- DRY: complementa a skill global de autoria, sem duplicar
