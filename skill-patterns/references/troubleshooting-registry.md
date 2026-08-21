# Skill Nao Reconhecida pelo Registry

## Sintoma

A skill existe em `.assistant/skills/[nome]/SKILL.md`, mas nao aparece no registry do Genie Code ao tentar carrega-la.

## Causas — Status de Confirmacao

**Confirmado por observacao empirica direta, teste controlado formal ainda pendente:**

1. **Comentarios no YAML frontmatter** — uma linha `# comentario` dentro do bloco YAML quebra o parsing.
2. **Campos alem de `name` e `description`** — mesmo campos previstos como opcionais pela especificacao aberta de Agent Skills (ex.: um mapa de metadados, licenca, compatibilidade) parecem nao ser tolerados pelo parser do Genie Code na pratica observada. Isso e uma caracteristica desta implementacao especifica, nao uma regra do formato em geral.
3. **Description sem verbo de acao no inicio** — descriptions que comecam de forma generica ("Esta skill ajuda...") parecem ter pior taxa de acionamento correto.
4. **Description sem boundaries negativos** — reduz a precisao do triggering, mas nao ha confirmacao de que impeca o carregamento em si.

**Importante:** o item 2 (campos extras quebram o registry) esta sendo tratado como regra pratica de seguranca, mas o teste controlado que provaria isso de forma definitiva — pegar uma skill funcional, adicionar um campo extra, confirmar que ela some do registry, remover o campo, confirmar que ela reaparece — ainda nao foi executado formalmente. Ate que esse teste rode, tratar como forte indicio, nao como fato estabelecido.

## Diagnostico Rapido

1. Tentar carregar a skill pelo mecanismo normal do agente.
2. Se nao for reconhecida, abrir o `SKILL.md` e verificar, em ordem:
   - Ha comentarios no bloco YAML? Remover.
   - Ha campos alem de `name`/`description`? Remover ou mover para o corpo Markdown.
   - A description comeca com um verbo de acao?
   - A description tem boundaries negativos explicitos?
3. Salvar e tentar carregar novamente.

## Experimento Pendente

Para fechar a validacao formal do item 2:
1. Escolher uma skill funcional e confirmada no registry.
2. Adicionar um campo extra no YAML (ex.: um campo de autor).
3. Verificar se a skill desaparece do registry.
4. Remover o campo.
5. Confirmar que a skill reaparece.

Resultado deste teste deve atualizar este arquivo, movendo o item 2 de "confirmado por observacao" para "confirmado por teste controlado" ou revisando a causa se o teste nao reproduzir o problema.
