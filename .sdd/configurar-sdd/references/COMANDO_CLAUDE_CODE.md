# Atalho `/configurar-sdd` para o Claude Code

Este é o conteúdo a gravar em `.claude/commands/configurar-sdd.md` do projeto. Ele existe por um
motivo só: a pasta `.sdd/` é oculta e fica um nível abaixo de `.claude/skills/`, então o Claude
Code não descobre a skill `configurar-sdd` sozinho. O atalho devolve a chamada por linguagem
natural sem trazer a pasta de volta para o meio das 7 skills do fluxo.

## Como gravar

1. Descubra o caminho real desta skill a partir da raiz do projeto. Nas duas instalações
   suportadas ele é:
   - bundle em `.claude/skills/` → `.claude/skills/.sdd/configurar-sdd/SKILL.md`
   - bundle na raiz do projeto → `sdd/.sdd/configurar-sdd/SKILL.md`
2. Copie o modelo abaixo, trocando `<CAMINHO_SKILL>` por esse caminho.
3. Grave em `.claude/commands/configurar-sdd.md`. Se o arquivo já existir e o caminho estiver
   correto, não mexa nele.

## Modelo

```markdown
---
description: Configura ou reconfigura o fluxo SDD neste projeto
---

Leia `<CAMINHO_SKILL>` e siga integralmente as instruções dessa skill para configurar o SDD
neste projeto.
```
