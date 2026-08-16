---
name: escolha-sql-pyspark
description: Use ao decidir entre escrever uma transformacao em SQL ou PySpark.
version: 1.0.0
date: 2026-08-12
owner: Pedro O. Silva
---

---
skill_name: escolha-sql-pyspark
display_name: Escolha entre SQL e PySpark
version: 1.0.0
author: Pedro Silva
last_updated: 2026-08-14
description: |
  Critérios de decisão para escolher entre SQL e PySpark em transformações
  de dados no Databricks.
tags:
  - sql
  - pyspark
  - decision-framework
---

# Escolha entre SQL e PySpark

## Quando Preferir SQL

* **Consultas analíticas diretas** (agregações, joins, filtros simples)
* **Delta Lake operations nativas** (MERGE, UPDATE, DELETE)
* **Melhor performance** com motor Photon otimizado para SQL
* **Maior legibilidade** para analistas de negócio
* **Queries ad-hoc e exploratórias**

## Quando Preferir PySpark

* **Lógica complexa e iterativa** (loops, condicionais, controle programático)
* **Integração com bibliotecas Python** (ML, pandas, APIs)
* **Transformações complexas** em múltiplas etapas
* **Necessidade de variáveis** e reutilização de código

## Regra Geral

**Combinar ambos conforme o caso de uso** - ambos compilam para o mesmo motor Spark no Databricks.

**Escolher baseado em**:
* Legibilidade
* Manutenibilidade
* Contexto do usuário final (analistas vs engenheiros)

---

**Ao implementar mudanças, consulte `protocolo-atualizacao/SKILL.md` para atualizar documentações afetadas.**
