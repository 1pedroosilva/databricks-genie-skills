---
name: padrao-escrita
description: Use ao escrever, redigir, elaborar ou documentar documentacoes tecnicas (README, arquitetura, evolucao, celulas MD de notebooks, comentarios de codigo). Define tom sobrio e tecnico, verbos no presente, proibe emojis/icones, estabelece nivel de registro por tipo de documento. NAO use para revisar codigo funcional, criar assets, ou decisoes arquiteturais.

---

# Padrão de Escrita Técnica

**Versão:** 1.0.1 | **Data:** 2026-08-18 | **Domínio:** documentation | **Autor:** Pedro O. Silva

## Escopo

**APLICA-SE A**: Conteúdo versionado — arquivos que entram no Git do projeto:
* Documentação (README, arquitetura, evolução, dicionário de dados)
* Células markdown de notebooks
* Comentários de código
* Mensagens de commit

**NÃO SE APLICA**: Respostas do assistente na conversa com o usuário.

---

## Princípios Fundamentais

### Tom e Registro

* **Sóbrio, técnico, factual** — sem marketês, sem entusiasmo artificial
* **Sem jargões desnecessários** ou buzzwords
* **Português correto** com acentuação completa
* **Objetivo e direto** ao ponto
* **Verbos no presente**: Descrever o que o sistema/código faz, não contar o que foi feito
  - ✓ Correto: "Implementa", "carrega", "transforma"
  - ✗ Errado: "implementei", "carreguei", "fizemos"

### Formatação

#### PROIBIDO em Conteúdo Versionado

* **Emojis**: ✨ 🚀 📊 💡 🎯 ⚡ etc.
* **Ícones de status coloridos**: ✅ ❌ ⚠️ ⏳ 🔴 🟡 🟢 etc.
* **Símbolos decorativos**: ★ ☆ etc.
* **Setas como marcador ornamental** no início de linha, no lugar de bullet
* **Status expresso por símbolo** em vez de palavra: implementado, pendente, removido, em desenvolvimento

#### Uso Correto de Formatação

* **Negrito**: Apenas para ênfase técnica necessária (termos-chave, nomes de conceitos). NUNCA para ênfase emocional ou dramatização
* **Itálico**: Raramente, apenas para termos estrangeiros ou conceitos em introdução
* **Listas**: Claras, sem floreios. Use markdown padrão (`-` ou `*` para bullets, checkboxes `[ ]` apenas para tarefas acionáveis)
* **Marcadores monocromáticos** (✓ ✗): permitidos
* **Setas** (→): permitidas como operador de relação ou fluxo (bronze → silver → gold)
* **Estrutura de markdown**: separadores horizontais (---), headings, negrito em rótulo de definição, tabelas, blocos de código e listas são estrutura, não decoração, e não devem ser removidos

### Estrutura

* Parágrafos curtos e informativos
* Seções com títulos descritivos (não criativos)
* Hierarquia clara (H2 para seções principais, H3 para subseções)
* Sem repetições ou redundâncias

---

## Regras de Recência e Versionamento

### Proibições

**PROIBIDO usar elementos visuais ou formatação para indicar recência**:
* Ícones coloridos
* Negritos seletivos
* Seções de "últimas atualizações"
* Badges
* Cores
* Qualquer marcador visual não pode ser usado para inferir qual versão é mais recente

### Como Determinar Recência

**Recência se determina por**:
1. Campo de versão declarado
2. Diff de conteúdo no Git
3. Histórico de commits

### Exceções Válidas

**Versionamento explícito permitido em**:
* Campo de versão e data no cabeçalho YAML de skills (obrigatório para rastreabilidade)
* Documento cronológico dedicado (`evolucao_projeto.md`) - é o lugar certo para histórico

**README não é changelog**: Não incluir seções de "atualizações recentes", "última modificação", "histórico de mudanças" ou datas de última atualização em arquivos README de projeto. O histórico está no Git.

### Métricas e Resultados

Nenhuma métrica ou resultado de teste entra na documentação sem que:
1. O teste que a produziu tenha sido executado
2. Data de execução esteja registrada
3. Método de cálculo esteja documentado

## Níveis de Registro por Tipo de Documento

### README.md (Projeto)

**Objetivo**: Apresentar o projeto de forma profissional e direta.

**Tom**: Técnico-informativo, acessível sem ser infantil.

**Exemplo BOM**:
```markdown
Pipeline de ingestão e transformação de dados financeiros da CVM. 
Arquitetura medalhão com camadas bronze, silver e gold.
```

**Exemplo RUIM**:
```markdown
🚀 Projeto incrível que traz os dados da CVM de forma automatizada! ✨
```

### Documentação Técnica (arquitetura.md, dicionario_dados.md)

**Objetivo**: Especificar implementação com precisão.

**Tom**: Técnico-descritivo, sem ambiguidade.

**Estrutura**: Definições → Implementação → Considerações

**Exemplo BOM**:
```markdown
Tabela `silver_balanco_patrimonial`: dados consolidados do balanço patrimonial. 
Chave: (`cd_cvm`, `dt_refer`, `versao`). 
Estratégia de gravação: `replaceWhere` por `dt_refer`.
```

**Exemplo RUIM**:
```markdown
Essa tabela guarda as informações dos balanços de forma organizada e limpa! 📊
```

### Evolução de Projeto (evolucao_projeto.md)

**Objetivo**: Registrar decisões e contexto para referência futura.

**Tom**: Analítico-reflexivo, factual.

**Estrutura**: Contexto → Decisão → Implementação → Resultado/Insight

**Exemplo BOM**:
```markdown
Identificado problema de duplicação ao usar `append` sem controle de versão. 
Decisão: migrar para `replaceWhere` com partição por `dt_refer`. 
Resultado: eliminação de duplicatas, reprocessamento mais rápido.
```

**Exemplo RUIM**:
```markdown
Tivemos um problema chato com duplicatas. Depois de muito pensar, 
escolhemos uma solução melhor! 💡
```

### Células Markdown em Notebooks

**Objetivo**: Documentar propósito e lógica do código.

**Tom**: Técnico-explicativo, conciso.

**Formato**: Título da célula como header, descrição breve se necessário.

**Exemplo BOM**:
```markdown
## Carregar configurações

Carrega dicionário de parâmetros do arquivo JSON. Fallback para valores 
default se arquivo não existir.
```

**Exemplo RUIM**:
```markdown
## 🎯 Vamos carregar as configs!

Aqui a gente pega todas as configurações necessárias para rodar o notebook...
```

### Comentários de Código

**Objetivo**: Explicar o POR QUÊ, não o O QUÊ (código já mostra o que faz).

**Tom**: Técnico-justificativo, mínimo necessário.

**Exemplo BOM**:
```python
# Mantém apenas versão mais recente para evitar duplicação no join downstream
```

**Exemplo RUIM**:
```python
# Aqui a gente filtra os dados para pegar só a versão mais nova
```

---

## Checklist Antes de Finalizar Qualquer Texto

Antes de publicar qualquer documentação, verificar:

1. **Contém emojis, ícones coloridos ou símbolos decorativos?** → Remover todos, substituir por palavras. Ver a lista em PROIBIDO em Conteúdo Versionado.
2. **Contém palavras sem acentuação?** → Corrigir para português correto
3. **Tom empolgado/marketeiro?** → Neutralizar para técnico-factual
4. **Usa negrito para ênfase emocional em vez de técnica?** → Remover ou substituir
5. **Jargões desnecessários?** → Simplificar sem perder precisão
6. **Repetições ou redundâncias?** → Eliminar
7. **Usa elementos visuais para indicar recência ou status?** → Remover, adicionar campo de versão explícito se necessário
8. **Usa formatação para indicar recência em README?** → Remover (histórico está no Git; exceção para cabeçalho de skill ou `evolucao_projeto.md`)
9. **Métricas ou resultados sem data, método ou teste executado?** → Remover ou documentar origem
10. **Transmite a informação necessária de forma clara e direta?** → OK para publicar

---

## Exemplos de Transformação

### Exemplo 1: README.md

**ANTES**:
```markdown
🚀 Incrível pipeline de dados que automatiza todo o processo de ingestão da CVM! 
Usa as melhores práticas do mercado com arquitetura medalhão. ✨
```

**DEPOIS**:
```markdown
Pipeline de ingestão automática de dados da CVM. Implementa arquitetura 
medalhão (bronze/silver/gold) com estratégias diferenciadas de gravação por camada.
```

### Exemplo 2: Célula de Notebook

**ANTES**:
```markdown
# Passo 1: Configuracao

Nessa etapa a gente le o arquivo de config pra pegar os parametros 
necessarios pro notebook funcionar direitinho.
```

**DEPOIS**:
```markdown
## Carregar Configurações

Lê parâmetros do arquivo JSON. Configura caminhos de tabelas, períodos 
de processamento e estratégias de gravação.
```

---

## Aplicação

* Carregar esta skill ANTES de criar ou editar qualquer documentação
* Aplicar os princípios sem precisar mencionar a skill ao usuário
* Se o texto produzido violar estes padrões, revisar antes de apresentar
* O usuário deve receber documentação já no padrão correto
