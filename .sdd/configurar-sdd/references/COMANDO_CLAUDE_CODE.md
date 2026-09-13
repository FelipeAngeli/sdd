# Atalhos do Claude Code para as skills fora do primeiro nível

O Claude Code só descobre skill que está **exatamente** em `.claude/skills/<nome>/SKILL.md`. Skill
que vive um nível abaixo disso não é descoberta sozinha — é limitação da ferramenta, não deste
bundle. Duas pastas do bundle estão nessa situação, cada uma por um motivo:

- `.sdd/` — infraestrutura (`configurar-sdd`, `atualizar-sdd`), oculta de propósito para não
  aparecer no meio das skills do dia a dia
- `qa/` — o grupo de QA contínuo (`criar-plano-qa`, `executar-sessao-qa`), agrupado de propósito
  para a listagem de skills não crescer a cada skill nova

O atalho devolve a chamada por linguagem natural para as duas, sem trazer as pastas de volta para
o primeiro nível.

## Como gravar

1. Descubra o caminho real de cada skill a partir da raiz do projeto. Nas duas instalações
   suportadas ele é:
   - bundle em `.claude/skills/` → `.claude/skills/<grupo>/<skill>/SKILL.md`
   - bundle na raiz do projeto → `sdd/<grupo>/<skill>/SKILL.md`

   onde `<grupo>` é `.sdd` ou `qa`.
2. Copie o modelo correspondente abaixo, trocando `<CAMINHO_SKILL>` por esse caminho.
3. Grave em `.claude/commands/<skill>.md`. Se o arquivo já existir e o caminho estiver correto,
   não mexa nele.
4. Os dois atalhos de `qa/` só são gravados quando `<qa_continuo>` do config está com
   `Ativo: sim`. Se a camada não está ativa, os arquivos não devem existir.

## Modelo — `.claude/commands/configurar-sdd.md`

```markdown
---
description: Configura ou reconfigura o fluxo SDD neste projeto
---

Leia `<CAMINHO_SKILL>` e siga integralmente as instruções dessa skill para configurar o SDD
neste projeto.
```

## Modelo — `.claude/commands/atualizar-sdd.md`

```markdown
---
description: Atualiza a copia do bundle SDD deste projeto a partir de um bundle mestre
---

Leia `<CAMINHO_SKILL>` e siga integralmente as instruções dessa skill para atualizar o bundle SDD
deste projeto.
```

## Modelo — `.claude/commands/criar-plano-qa.md`

```markdown
---
description: Mantem a arvore viva de QA continuo do produto
---

Leia `<CAMINHO_SKILL>` e siga integralmente as instruções dessa skill para planejar ou atualizar o
QA contínuo deste projeto.
```

## Modelo — `.claude/commands/executar-sessao-qa.md`

```markdown
---
description: Roda uma sessao de QA continuo contra um escopo (branch, PR ou release)
---

Leia `<CAMINHO_SKILL>` e siga integralmente as instruções dessa skill para executar a sessão de QA
deste escopo.
```
