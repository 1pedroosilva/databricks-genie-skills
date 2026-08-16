# Lições Aprendidas: Testar Hipóteses Antes de Publicar

**Data:** 2026-08-14 / 2026-08-15  
**Projeto:** Skills Personalizadas Databricks Genie Code  
**Contexto:** Troubleshooting técnico - requisitos de triggering não documentados

---

## O Problema

Durante investigação para fazer skills customizadas triggarem no Databricks Genie Code, observei padrões em skills funcionais:

1. Todas usavam ASCII puro (0 caracteres não-ASCII)
2. Todas especificavam boundaries negativos ("NÃO use para...")
3. Todas tinham verbo imperativo único no início da description

Documentei esses padrões como "requisitos empíricos não documentados" no README e investigation_log.md, e publiquei no GitHub.

**Problema:** Não tinha validado se esses padrões eram causa ou apenas correlação.

---

## Correção

### Fase 1: Questionar Conclusões

Antes de apresentar portfólio a recrutadores, validei conclusões anteriores. Questão crítica:

> Esses padrões são requisitos técnicos ou boas práticas que todas as skills funcionais coincidentemente seguiram?

### Fase 2: Protocolo de Teste

Criei protocolo para testar cada hipótese isoladamente:

**Protocolo:**
1. Backup completo do estado baseline funcional (garantir reversibilidade)
2. Teste 1: Adicionar 28 caracteres acentuados (testar H1: ASCII obrigatório)
3. Aguardar 10 minutos (período de re-indexação do registry)
4. Abrir nova conversa Genie Code (evitar cache de sessão)
5. Executar query de teste
6. Documentar resultado com evidência visual (screenshot)
7. Reverter para baseline
8. Repetir para H2 (boundaries) e H3 (verbo imperativo)

**Elemento crítico:** Uma variável por teste. Reversão entre testes.

### Fase 3: Executar Testes

**Teste 1 (ASCII):** Adicionei acentos. Skill triggou corretamente. H1 refutada.

**Teste 2 (Boundaries):** Removi "NÃO use para...". Apenas skill correta triggou. H2 refutada.

**Teste 3 (Verbo imperativo):** Removi "Use APENAS ao". Skill triggou normalmente. H3 refutada.

**Resultado:** Todas as três hipóteses empiricamente refutadas.

### Fase 4: Corrigir Documentação

1. Atualizei README.md com achados reais
2. Documentei protocolo completo em protocolo_teste_empirico.md
3. Reescrevi seções que apresentavam correlação como causalidade
4. Criei este documento para transparência

---

## Achados Principais

### 1. Observar Padrão Não Prova Causalidade

**Erro:** Observei que 8/8 skills funcionais usavam ASCII puro. Concluí que ASCII era obrigatório.

**Realidade:** Skills funcionavam por outras razões (description clara, verbo distinto, etc.). ASCII puro era coincidência.

**Lição:** Correlação não implica causalidade. Teste empírico é o único caminho para conclusões técnicas confiáveis.

### 2. Cargo Cult Programming

**Definição:** Seguir práticas sem entender se são requisitos técnicos ou convenções.

**Meu caso:** Documentei "ASCII obrigatório" porque skills funcionais usavam ASCII, não porque testei que não-ASCII falhava.

**Lição:** Sempre questionar: Isso é requisito técnico ou tradição/coincidência?

### 3. Validação Empírica é Fundamental

**Antes de publicar conclusões técnicas:**
- Observou o padrão?
- Testou a variável isoladamente?
- Documentou evidência visual?
- Reproduziu o resultado?

Só depois de todos os checkboxes: Publicar como "empiricamente verificado."

### 4. Transparência Sobre Perfeição

**Opção A (incorreta):** Deletar o erro, fingir que nunca aconteceu, republicar como se nada mudou.

**Opção B (correta):** Documentar o erro, o processo de descoberta, e transformar em demonstração de pensamento crítico.

Recrutadores querem engenheiros que:
- Questionam suas próprias conclusões
- Testam hipóteses rigorosamente
- Admitem erros e corrigem rapidamente
- Documentam processo completo (incluindo falhas)

**Este documento existe porque escolhi opção B.**

---

## Descoberta Real

### Sistema de Triggering é Mais Robusto Que Documentado

**Funciona com:**
- UTF-8 completo (acentos, caracteres especiais)
- Formato flexível (reconhece intent sem estrutura rígida)
- Compreensão semântica (não é keyword matching simples)

**Não requer:**
- ASCII puro
- Boundaries negativos (boa prática, não requisito técnico)
- Verbo imperativo em formato específico

### Padrões Que Permanecem (Como Boas Práticas, Não Requisitos)

1. Verbo de ação distinto - Clareza para humanos lendo registry
2. Boundaries negativos - Documentação explícita de escopo
3. Máxima especificidade - Facilita matching semântico (mas não garante)

---

## Aplicação a Engenharia de Dados

Esta experiência aplica diretamente a trabalho de engenharia de dados:

### Troubleshooting de Pipelines
- Observar "pipeline X falha toda segunda-feira" não significa segunda-feira causa falha
- Testar isoladamente: desabilitar scheduler, rodar manualmente segunda, verificar dependências externas
- Documentar causa raiz real (ex: job batch upstream roda só segunda)

### Otimização de Queries
- Ver "query Y está lenta" e adicionar índice não prova que índice resolveu
- Testar: explain plan antes/depois, verificar se índice foi usado, medir latência real
- Conclusão: Índice ajudou? Outro fator? Ambos?

### Validação de Regras de Negócio
- Stakeholder diz "tabela X sempre tem valores positivos" não significa regra é verdadeira
- Testar: `SELECT COUNT(*) FROM X WHERE value <= 0`
- Se resultado > 0: Regra é aspiração, não realidade. Código deve tratar.

---

## Métricas de Sucesso

**Versão inicial (incorreta):**
- Skills triggam corretamente
- Conclusões técnicas publicadas sem validação

**Versão corrigida (rigorosa):**
- Skills triggam corretamente
- Protocolo de teste empírico executado (3 testes isolados)
- Hipóteses testadas e refutadas com evidência
- Documentação corrigida com achados reais
- Processo completo documentado para transparência

---

## Conclusão

Cometi erro clássico de engenharia: documentar inferências como fatos.

Depois fiz o que bons engenheiros fazem: questionei conclusões, testei rigorosamente, corrigi rapidamente, e documentei processo completo.

**Lição final:** Erro corrigido com rigor demonstra mais competência técnica que conclusão não validada apresentada como verdade.

---

**Autor:** Pedro Silva  
**Contexto:** Transição de carreira - Analista de Dados → Engenheiro de Dados  
**Data:** 2026-08-15
