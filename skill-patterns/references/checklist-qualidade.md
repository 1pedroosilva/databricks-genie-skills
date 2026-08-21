# Checklist de Qualidade para Skill Nova

Validar antes de criar o arquivo:

1. [ ] Verificacao DRY - skills globais: topico nao esta coberto por uma skill global?
2. [ ] Verificacao DRY - user skills: nao ha duplicacao com `.assistant/skills/` existente?
3. [ ] Colisao com skill nativa da plataforma verificada (notebooks, Unity Catalog, dashboards, pipelines, MLflow)?
4. [ ] Description < 1024 caracteres?
5. [ ] Description em ASCII puro (sem acentos)?
6. [ ] Description com vocabulario rico (sinonimos naturais para a acao principal)?
7. [ ] Boundaries negativos explicitos na description?
8. [ ] Contexto especifico detalhado na description (QUE, QUANDO, ONDE)?
9. [ ] Dominio definido e alinhado com a tabela de dominios existente?
10. [ ] YAML frontmatter limpo — apenas `name` e `description`, nada mais?
11. [ ] Corpo Markdown separa Principios (orientacao) de Procedimento (execucao)?
12. [ ] Conteudo extenso movido para `references/`, mantendo o `SKILL.md` enxuto?
13. [ ] Exemplos usam identificadores genericos, nao nomes reais de artefatos de um projeto especifico?

## Metricas de Qualidade (Alvo)

| Metrica | Alvo |
|---------|------|
| Description < 1024 chars | 100% |
| ASCII puro na description | 100% |
| Vocabulario rico (sinonimos) | 100% |
| Boundaries negativos explicitos | 100% |
| Dominio definido | 100% |
| YAML com apenas 2 campos | 100% |
| SKILL.md principal < 500 linhas | 100% |
