---
name: padrao-escrita
description: Use APENAS ao ESCREVER documentacoes tecnicas (README, arquitetura, evolucao, celulas MD de notebooks, comentarios de codigo). Define tom sobrio e tecnico, verbos no presente, proibe emojis/icones, estabelece nivel de registro por tipo de documento. NAO use para revisar codigo funcional, criar assets, ou decisoes arquiteturais.
version: 1.0.0
updated: 2025-01-30
---

# Padrão de Escrita Técnica

Esta skill define o padrão de escrita para TODAS as documentações e conteúdos textuais produzidos.

## Princípios Fundamentais

**Tom e registro**:
- Sóbrio, técnico, factual
- Sem marketês, sem entusiasmo artificial
- Sem jargões desnecessários ou buzzwords
- Português correto com acentuação completa
- Objetivo e direto ao ponto
- **Verbos no presente**: Descrever o que o sistema/código faz, não contar o que foi feito. "Implementa", "carrega", "transforma" — não "implementei", "carreguei"

**Formatação**:
- **PROIBIDO**: Emojis, ícones coloridos (✨ 🚀 📊 ✅ ❌ 💡 etc)
- Use negrito apenas para ênfase técnica necessária
- Use itálico raramente, apenas para termos estrangeiros ou conceitos
- Listas: claras, sem floreios

**Estrutura**:
- Parágrafos curtos e informativos
- Seções com títulos descritivos (não criativos)
- Hierarquia clara (H2 para seções principais, H3 para subseções)
- Sem repetições ou redundâncias

## Níveis de Registro por Tipo de Documento

### README.md (Projeto)
- **Objetivo**: Apresentar o projeto de forma profissional e direta
- **Tom**: Técnico-informativo, acessível sem ser infantil
- **Exemplo BOM**: "Pipeline de ingestão e transformação de dados financeiros da CVM. Arquitetura medalhão com camadas bronze, silver e gold."
- **Exemplo RUIM**: "🚀 Projeto incrível que traz os dados da CVM de forma automatizada! ✨"

### Documentação Técnica (arquitetura.md, dicionario_dados.md)
- **Objetivo**: Especificar implementação com precisão
- **Tom**: Técnico-descritivo, sem ambiguidade
- **Estrutura**: Definições → Implementação → Considerações
- **Exemplo BOM**: "Tabela `silver_balanco_patrimonial`: dados consolidados do balanço patrimonial. Chave: (`cd_cvm`, `dt_refer`, `versao`). Estratégia de gravação: `replaceWhere` por `dt_refer`."
- **Exemplo RUIM**: "Essa tabela guarda as informações dos balanços de forma organizada e limpa! 📊"

### Evolução de Projeto (evolucao_projeto.md)
- **Objetivo**: Registrar decisões e contexto para referência futura
- **Tom**: Analítico-reflexivo, factual
- **Estrutura**: Contexto → Decisão → Implementação → Resultado/Insight
- **Exemplo BOM**: "Identificado problema de duplicação ao usar `append` sem controle de versão. Decisão: migrar para `replaceWhere` com partição por `dt_refer`. Resultado: eliminação de duplicatas, reprocessamento mais rápido."
- **Exemplo RUIM**: "Tivemos um problema chato com duplicatas. Depois de muito pensar, escolhemos uma solução melhor! 💡"

### Células Markdown em Notebooks
- **Objetivo**: Documentar propósito e lógica do código
- **Tom**: Técnico-explicativo, conciso
- **Formato**: Título da célula como header, descrição breve se necessário
- **Exemplo BOM**: "## Carregar configurações\nCarrega dicionário de parâmetros do arquivo JSON. Fallback para valores default se arquivo não existir."
- **Exemplo RUIM**: "## 🎯 Vamos carregar as configs!\nAqui a gente pega todas as configurações necessárias para rodar o notebook..."

### Comentários de Código
- **Objetivo**: Explicar o POR QUÊ, não o O QUÊ (código já mostra o que faz)
- **Tom**: Técnico-justificativo, mínimo necessário
- **Exemplo BOM**: `# Mantém apenas versão mais recente para evitar duplicação no join downstream`
- **Exemplo RUIM**: `# Aqui a gente filtra os dados para pegar só a versão mais nova`

## Checklist Antes de Finalizar Qualquer Texto

1. ❌ Contém emojis ou ícones? → Remover todos
2. ❌ Contém palavras sem acentuação? → Corrigir para português correto
3. ❌ Tom empolgado/marketeiro? → Neutralizar para técnico-factual
4. ❌ Jargões desnecessários? → Simplificar sem perder precisão
5. ❌ Repetições ou redundâncias? → Eliminar
6. ✓ Transmite a informação necessária de forma clara e direta? → OK para publicar

## Exemplos de Transformação

**ANTES**: "🚀 Incrível pipeline de dados que automatiza todo o processo de ingestão da CVM! Usa as melhores práticas do mercado com arquitetura medalhão. ✨"

**DEPOIS**: "Pipeline de ingestão automática de dados da CVM. Implementa arquitetura medalhão (bronze/silver/gold) com estratégias diferenciadas de gravação por camada."

---

**ANTES**: "# Passo 1: Configuracao\nNessa etapa a gente le o arquivo de config pra pegar os parametros necessarios pro notebook funcionar direitinho."

**DEPOIS**: "## Carregar Configurações\nLê parâmetros do arquivo JSON. Configura caminhos de tabelas, períodos de processamento e estratégias de gravação."

## Aplicação

- Carregar esta skill ANTES de criar ou editar qualquer documentação
- Aplicar os princípios sem precisar mencionar a skill ao usuário
- Se o texto produzido violar estes padrões, revisar antes de apresentar
- O usuário deve receber documentação já no padrão correto
