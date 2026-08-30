---
name: asset-placement
description: Use ao criar, desenvolver, adicionar, gravar ou estruturar notebooks, arquivos MD, scripts Python, arquivos de configuracao ou qualquer outro asset no workspace -- cobre protocolo de governanca para ONDE criar (identificar projeto, listar estrutura, decidir pasta, nunca assumir organizacao), proibicao de criar na pasta home do usuario, proibicao de criar pastas novas sem autorizacao, checklists obrigatorios pre/pos-criacao, e protocolo de remediacao para arquivos orfaos. NAO use para definir nomenclatura de assets (naming-conventions cobre), estrutura interna de notebooks (notebook-structure cobre), organizacao de schemas/tabelas UC (unity-catalog-naming cobre), ou decisoes arquiteturais de pipeline (medallion-architecture cobre).
---

# Localizacao de Assets no Workspace

**Versao:** 2.1.0 | **Data:** 2026-08-25 | **Dominio:** project-management | **Autor:** Pedro O. Silva

## Quando Usar Esta Skill

**USAR** ao criar, desenvolver, adicionar ou gravar qualquer asset no workspace (notebook, arquivo MD, script Python, arquivo de configuracao, etc.) -- para garantir que o asset seja criado no local correto dentro da estrutura do projeto.

**NAO USAR** para:
- Definir nomenclatura de assets → usar `naming-conventions`
- Estruturar conteudo interno de notebooks → usar `notebook-structure`
- Organizar schemas/tabelas no Unity Catalog → usar `unity-catalog-naming`
- Decisoes arquiteturais de pipeline → usar `medallion-architecture`
- Revisar ou auditar assets existentes → usar `code-review`

## Principios (Orientacao)

Estes principios orientam o comportamento de criacao de assets no workspace:

1. **Proibicao de criar estruturas novas.** Nunca criar pastas ou diretorios sem autorizacao expressa do usuario. A estrutura do projeto ja esta estabelecida -- novos assets devem ser criados dentro das pastas existentes.

2. **Identificacao previa da pasta correta.** Antes de criar um asset, identificar a pasta correta dentro da estrutura existente do projeto. Se houver duvida sobre onde criar, PERGUNTAR ao usuario antes de prosseguir.

3. **Nunca criar na pasta home/padrao do usuario.** Assets de projeto NUNCA devem ser criados diretamente em `/Users/<usuario>/` -- sempre dentro da estrutura do projeto especifico.

4. **Consistencia entre criacao e edicao.** Se o usuario pedir para "criar notebook X" e depois "editar notebook X", o agente deve lembrar onde criou e usar o mesmo path. Nao criar duplicatas em locais diferentes.

5. **Nunca assumir estrutura de projeto.** Cada projeto tem sua propria organizacao (pode ser arquitetura medalhao, pastas tematicas, estrutura plana, etc). SEMPRE identificar a estrutura do projeto atual antes de criar assets. Nunca usar estrutura de um projeto como template para outro.

## Procedimento (Execucao)

### Passo 1: Identificar o Projeto e Sua Estrutura

Antes de criar qualquer asset, SEMPRE:

1. **Identificar o projeto atual:**
   - Pelo path do arquivo/pasta atualmente aberto
   - Pelo contexto da conversa (usuario mencionou o projeto)
   - Se nao conseguir identificar o projeto → PARAR e PERGUNTAR ao usuario

2. **Identificar a estrutura do projeto:**
   - Listar as pastas existentes no projeto
   - Identificar a pasta correta para o tipo de asset que sera criado
   - Considerar:
     * Qual a funcao do asset (ingestao, transformacao, validacao, apoio, documentacao)?
     * Qual pasta existente melhor se alinha com essa funcao?
     * Ha convencoes de nomenclatura visiveis (numeracao, prefixos)?

3. **Decidir a pasta de destino:**
   - Se houver certeza absoluta sobre onde criar → Seguir para Passo 2
   - Se houver QUALQUER duvida → PARAR e PERGUNTAR ao usuario onde criar

**REGRA CRITICA:** Nunca assumir estrutura de projeto. Cada projeto tem sua propria organizacao (pode ser arquitetura medalhao, pastas tematicas, estrutura plana, etc). O que funciona para um projeto pode nao existir em outro.

### Passo 2: Checklist Pre-Criacao (Obrigatorio)

Antes de criar o asset, SEMPRE validar:

- [ ] Projeto atual identificado
- [ ] Estrutura do projeto listada
- [ ] Pasta de destino identificada com certeza absoluta
- [ ] Nenhuma pasta nova sera criada (estrutura ja existe)
- [ ] Se houver QUALQUER duvida sobre onde criar → PARAR e PERGUNTAR ao usuario

**Se qualquer item falhar:** PARAR antes de criar o asset.

### Passo 3: Se Precisar Criar Pasta Nova (Excepcional)

**Regra:** NUNCA criar pasta nova sem autorizacao expressa do usuario.

Se durante a execucao o agente identificar necessidade de criar pasta nova (ex.: usuario pediu para criar arquivo em pasta que nao existe):

1. PARAR a operacao imediatamente
2. INFORMAR ao usuario que a pasta nao existe
3. PERGUNTAR se o usuario autoriza a criacao da pasta OU se prefere usar pasta existente
4. AGUARDAR resposta explicita antes de prosseguir

### Passo 4: Protocolo de Arquivos Orfaos (Limpeza)

**Arquivo orfao**: Asset criado fora da estrutura do projeto (pasta home do usuario, local errado, path incorreto).

**Como identificar:**

* Arquivo criado diretamente em `/Users/<usuario>/` (sem pasta de projeto)
* Arquivo criado em pasta nova que nao deveria existir
* Notebook ou arquivo com path relativo (ex.: `pasta-x/notebook` criou pasta `pasta-x` na home)

**Quando ocorre:**

* Agente nao carregou esta skill antes de criar
* Skill foi lida incorretamente
* Falha no meio da execucao
* Path construido errado (relativo ou com barra inicial)

**Protocolo de limpeza obrigatorio:**

1. **IDENTIFICAR o orfao**: Checar se o asset foi criado no local correto
   - Path esperado: `projeto-raiz/pasta-destino/nome-arquivo`
   - Path real (verificar onde foi criado)

2. **DELETAR o orfao**: Nunca mover ou renomear - sempre deletar
   - Arquivos criados no lugar errado NAO devem ser preservados
   - Mover arquivos pode causar perda de metadados ou referencias quebradas

3. **RECRIAR no local correto**: Seguir Passos 1-2 desta skill
   - Carregar esta skill se ainda nao foi carregada
   - Identificar projeto e estrutura (Passo 1)
   - Validar checklist pre-criacao (Passo 2)
   - Criar asset

4. **VERIFICAR**: Confirmar que o asset foi criado no local esperado
   - Listar diretorio de destino
   - Confirmar que nenhum orfao permanece na home do usuario

**Regra de ouro**: Sempre limpar antes de recriar. Nunca deixar arquivos orfaos no workspace.

**Exemplo de limpeza:**

```bash
# Identificar orfao
ls -la /Workspace/Users/<usuario>/ | grep "notebook_orfao"

# Deletar orfao
rm /Workspace/Users/<usuario>/notebook_orfao.py

# Verificar limpeza
ls -la /Workspace/Users/<usuario>/

# Recriar no local correto
# meu-projeto/pasta-correta/notebook_correto
```

### Passo 5: Checklist Pos-Criacao (Obrigatorio)

Apos criar o asset, SEMPRE validar:

- [ ] Asset foi criado no local esperado
- [ ] Path final esta correto: `projeto-raiz/pasta-destino/nome-arquivo`
- [ ] Nenhum arquivo orfao permanece na pasta home (`/Users/<usuario>/`)
- [ ] Nenhuma pasta nova foi criada acidentalmente
- [ ] Asset e acessivel e operacional (consegue abrir/editar)

**Como verificar:**

```bash
# 1. Listar pasta de destino para confirmar asset criado
ls -la /Workspace/Users/<usuario>/meu-projeto/pasta-destino/

# 2. Verificar ausencia de orfaos na home
ls -la /Workspace/Users/<usuario>/ | grep -v "^d" | grep -v ".assistant"

# 3. Verificar ausencia de pastas novas criadas acidentalmente
ls -la /Workspace/Users/<usuario>/ | grep "^d"
```

**Se qualquer item falhar:**

1. PARAR imediatamente
2. Identificar o problema (asset no lugar errado, orfao criado, pasta nova)
3. Seguir Protocolo de Arquivos Orfaos (Passo 4) se necessario
4. Recriar seguindo Passos 1-2
5. Repetir este checklist pos-criacao

**Regra critica**: Nenhuma operacao de criacao esta completa ate este checklist passar por completo.

## Aplicacao

- Carregar esta skill ANTES de criar qualquer asset no workspace
- Sempre identificar projeto e estrutura primeiro (Passo 1)
- OBRIGATORIO: Rodar checklist pre-criacao (Passo 2) antes de criar
- OBRIGATORIO: Rodar checklist pos-criacao (Passo 5) apos criar
- Se houver duvida sobre onde criar, PARAR e PERGUNTAR antes de executar
- Nunca criar pasta nova sem autorizacao expressa
- Se asset foi criado no local errado: seguir Protocolo de Arquivos Orfaos (Passo 4) - deletar e recriar, nunca mover
- Nenhuma operacao de criacao esta completa ate o checklist pos-criacao passar por completo
