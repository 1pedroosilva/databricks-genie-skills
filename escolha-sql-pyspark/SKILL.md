---
name: escolha-sql-pyspark
description: Use ao decidir, escolher, avaliar ou definir entre escrever uma transformacao em SQL ou PySpark. Criterios de decisao baseados em complexidade, performance, manutencao. NAO use para implementar codigo SQL ou PySpark -- esta skill cobre apenas a DECISAO da linguagem.

---

# Escolha entre SQL e PySpark

**Versão:** 1.0.3 | **Data:** 2026-08-18 | **Domínio:** architecture | **Autor:** Pedro O. Silva

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
