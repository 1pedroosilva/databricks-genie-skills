---
name: estrutura-notebooks
description: Use ao criar, desenvolver, estruturar ou montar notebooks NOVOS do zero -- ordem fixa de celulas iniciais (DOCUMENTACAO, CARREGAR CONFIGURACOES, INICIALIZAR ANOS A PROCESSAR, IMPORTS), separacao de responsabilidades por celula, formato de arquivo (.py vs .ipynb). NAO use para revisar, editar ou validar estrutura de notebooks existentes.

---

# Estrutura de Notebooks

**Versão:** 1.0.4 | **Data:** 2026-08-18 | **Domínio:** code-structure | **Autor:** Pedro O. Silva

## Formato de Arquivo

**REGRA OBRIGATÓRIA**: Notebooks devem ser SEMPRE criados no formato `.py` (Python Source / Databricks format).

**Justificativa - Versionamento Git Limpo**:
* **Diff legível**: Formato texto puro, linha por linha, ideal para code review em Pull Requests
* **Sem noise de metadata**: `.ipynb` (JSON) reescreve `execution_count` e outputs a cada save, poluindo diffs mesmo sem mudança lógica
* **Padrão de mercado**: Times que usam Databricks Repos + CI/CD geralmente padronizam em `.py` exatamente por isso
* **Histórico git como narrativa**: Para projetos de portfólio onde commits incrementais contam a história técnica, `.py` trabalha a favor
* **Menos conflitos de merge**: Metadados JSON são fonte clássica de conflitos chatos em colaboração

**Observação - Metadados do Workspace**:
* Data de criação, histórico de execução, execution_count: metadados internos do Databricks Workspace
* **Não aparecem no GitHub/portfólio** - audiência externa só vê o código commitado
* Para portfólio técnico, o que importa é o histórico git (commits, PRs, evolução do código)

**PROIBIDO**: Criar notebooks em formato `.ipynb` (Jupyter Notebook)
* Diffs poluídos com metadata JSON não-relevante
* Dificulta revisão de código (mudanças reais se perdem no noise)
* Conflitos de merge frequentes em outputs e execution_count

---

## Limitação Técnica Importante

**CÉLULAS MARKDOWN NÃO TÊM CAMPO DE TÍTULO**

* Apenas células de **código** (Python, SQL, Scala, R) possuem campo editável de título ao lado da numeração
* Células de **Markdown** não suportam títulos por limitação técnica da ferramenta Databricks
* Esta é uma característica da plataforma, não uma escolha de padrão
* Portanto: NUNCA cobrar ou esperar títulos em células markdown

## Células Iniciais

### Célula 1 - Documentação
* **Tipo**: Markdown
* **Conteúdo**: Explicação do objetivo, conteúdo e função do notebook
* **Título**: Não aplicável - células markdown não têm campo de título

### Célula 2 - Carregar Configurações
* **Tipo**: Código (Python/Scala/R)
* **Título**: "Carregar configurações"
* **Conteúdo**: SOMENTE o `%run` do módulo de configuração compartilhado — nada mais
* **SEM linha de comentário explicativo**: o `%run` já é autoexplicativo, não repetir em comentário o que o título já diz

#### Carregamento de Módulos Compartilhados
* **Padrão obrigatório**: `%run ./nome_do_modulo` (caminho relativo)
* **PROIBIDO**: Usar caminho absoluto (`/Workspace/Users/...`) ou `open()` + `exec()`
* **Motivo**: Caminhos absolutos quebram portabilidade entre workspaces/contas e expõem e-mail/usuário no código
* **Exemplos**:
 - Notebook em `01_bronze/` carregando `05_apoio/config_parametros.py`: `%run ../05_apoio/config_parametros`
 - Notebook em `05_apoio/` carregando módulo na mesma pasta: `%run ./config_parametros`

### Célula 3 - Inicializar Anos a Processar
* **Tipo**: Código (Python/Scala/R)
* **Título**: "Inicializar Anos a Processar"
* **Conteúdo**: capturar `ANOS_PROCESSAR` explicitamente do retorno de `inicializar_anos_processar()`, com guardrail (`raise` se vazio)
* **Comentário permitido**: aqui, diferente da Célula 2 e da Célula 4, uma linha de comentário explicando a captura explícita do retorno é esperada (não é autoexplicativo como um `%run` ou um `import`)

### Célula 4 - Imports
* **Tipo**: Código (Python/Scala/R)
* **Título**: "Imports"
* **Regra**: Todos os imports do notebook devem estar concentrados aqui
* **NUNCA**: Espalhar imports pelo notebook
* **SEM linha de comentário explicativo**: imports são autoexplicativos, não comentar o óbvio

**Ordem obrigatória e fixa das células iniciais**: Documentação → Carregar Configurações → Inicializar Anos a Processar → Imports. Todo notebook segue essa sequência, sem exceção.

## Demais Células

### Estrutura Padrão

Cada célula deve:

* **Ter título em MAIÚSCULO**

* **Seguir padrão de 2 linhas de cabeçalho**:
 - Linha 1: `# df_[nome]: [descrição do que a célula faz]`
 - Linha 2: `# [explicação técnica/de negócio do motivo/abordagem]`

* **Fluxo de dados**:
 - Ler de um ou mais dataframes/tabelas
 - Aplicar transformação
 - Gerar um novo dataframe de saída

* **Nomenclatura de saída**: CARREGUE a skill nomenclaturas para regras de nomenclatura de DataFrames

## Separação de Responsabilidades

**CRÍTICO**: Uma célula não deve fazer muitas coisas.

### Separar transformações em células distintas:

* **Células que leem dados** (de tabelas ou outros dataframes)
* **Células que transformam** (filtros, limpezas, conversões)
* **Células que consultam** (queries analíticas)
* **Células que fazem cálculos** (métricas, KPIs)
* **Células que agregam** (GROUP BY, sumarizações)
* **Células que gravam** (geralmente as últimas) - salvam dataframes resultantes em tabelas

### Resiliência e Estrutura de Células

**IMPORTANTE**: um `try/except` não pode atravessar células (cada célula é compilada isoladamente — um `try:` sem `except` na mesma célula é `SyntaxError`). Ao adicionar tratamento de erros/retry/logging a um pipeline:
* Cada etapa (extração, transformação, gravação) continua sendo uma **função**, definida na sua própria célula
* O loop com `try/except` que chama essas funções fica **inteiro em uma única célula de orquestração** — não se abre em uma célula e fecha em outra
* Isso preserva a separação de responsabilidades por célula E mantém o código executável

* Padrão completo e exemplo: CARREGUE a skill guardrails-pipelines, seção 3 (Resiliência Operacional).

## Comentários e Documentação

* **Orientação sobre comentários**: CARREGUE a skill padrao-escrita para orientação sobre como escrever comentários de código (tom, estrutura, foco no porquê, exemplos)

---

**Ao implementar mudanças, consulte `protocolo-atualizacao/SKILL.md` para atualizar documentações afetadas.**
