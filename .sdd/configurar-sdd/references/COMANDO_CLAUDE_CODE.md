# Atalhos do Claude Code para as skills de infraestrutura

`configurar-sdd` e `atualizar-sdd` vivem na pasta oculta `.sdd/`, um nível abaixo de
`.claude/skills/` — então o Claude Code não as descobre sozinho. Os atalhos devolvem a chamada por
linguagem natural sem trazer essas pastas de volta para o meio das 7 skills do fluxo.

## Como gravar

1. Descubra o caminho real de cada skill a partir da raiz do projeto. Nas duas instalações
   suportadas ele é:
   - bundle em `.claude/skills/` → `.claude/skills/.sdd/<skill>/SKILL.md`
   - bundle na raiz do projeto → `sdd/.sdd/<skill>/SKILL.md`
2. Copie o modelo abaixo, trocando `<CAMINHO_SKILL>` por esse caminho.
3. Grave em `.claude/commands/<skill>.md`. Se o arquivo já existir e o caminho estiver correto,
   não mexa nele.

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
