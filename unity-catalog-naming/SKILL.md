---
name: unity-catalog-naming
description: Use ao criar, definir, estruturar ou organizar schemas, tabelas ou volumes no Unity Catalog -- padroes de nomenclatura UC, organizacao por camadas e projetos. NAO use para revisar schemas/tabelas existentes, nem para definir GRANTs, ownership ou controle de acesso -- esta skill cobre apenas nomenclatura e organizacao de assets NOVOS.

---

# Unity Catalog - Tabelas e Schemas

**Versão:** 1.0.4 | **Data:** 2026-08-18 | **Domínio:** naming | **Autor:** Pedro O. Silva

## Organização por Camadas

* Schemas e tabelas organizados seguindo camadas bronze/silver/gold
* Numeração de tabelas: formato XXX (3 dígitos). CARREGUE a skill naming-conventions para detalhes do sistema de numeração e rastreabilidade entre camadas

## Nomenclatura de Schemas

### Padrão: `proj_[abreviacao_projeto]_[numero]_[camada]`

**Componentes**:
* `proj_` = prefixo fixo obrigatório
* `[abreviacao_projeto]` = sigla/abreviação do projeto em minúsculas (ex: `cvm`, `rh`, `financ`)
* `[numero]` = 2 dígitos identificando a camada (`01`=bronze, `02`=silver, `03`=gold)
* `[camada]` = nome da camada em minúsculas (`bronze`, `silver`, `gold`)

### Exemplos

**Projeto CVM**:
* `proj_cvm_01_bronze`
* `proj_cvm_02_silver`
* `proj_cvm_03_gold`

**Outro projeto**:
* `proj_outro_01_bronze`
* `proj_outro_02_silver`
* `proj_outro_03_gold`

### Benefícios

* Isolamento total entre projetos no Unity Catalog
* Ordenação visual clara
* Rastreabilidade
* Escalabilidade para múltiplos projetos

## Nomenclatura de Tabelas

### Padrão: `[numero]_[nome_autocontido]`

* Tabelas dentro dos schemas seguem o padrão de numeração XXX (mesmo padrão dos notebooks)
* Primeiro dígito identifica a camada (1=bronze, 2=silver, 3=gold)
* Dois dígitos seguintes: sequência dentro da camada (01-99)

### Exemplos

* `proj_cvm_01_bronze.101_dre_consolidada`
* `proj_cvm_02_silver.201_dre_dfp`
* `proj_cvm_03_gold.301_indicadores_financeiros`

### Benefícios

* Rastreabilidade clara entre camadas (101→201→301)
* Ordenação visual perfeita
* Alinhamento com numeração dos notebooks
* Fácil identificação da origem dos dados

---

**Ao implementar mudanças, consulte `docs-sync/SKILL.md` para atualizar documentações afetadas.**
