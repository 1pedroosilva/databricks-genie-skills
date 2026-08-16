---
name: escolha-sql-pyspark
description: Use ao decidir entre escrever uma transformacao em SQL ou PySpark.

---

# Escolha entre SQL e PySpark

**Versão:** 1.0.0 | **Data:** 2026-08-16 | **Autor:** Pedro O. Silva

## Quando Usar Esta Skill

Carregar ao decidir entre escrever uma transformação em SQL ou PySpark. Ambas as linguagens compilam para o mesmo motor Spark no Databricks, mas cada uma tem casos de uso onde se destaca.

---

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
