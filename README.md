# SDD — Fluxo de desenvolvimento orientado a especificação

Um bundle de skills portátil. Coloque em qualquer projeto, peça para a IA se adaptar, e o fluxo
PRD → TechSpec → Tasks → Execução → QA → Bugfix → Review passa a funcionar com a stack, as
convenções e as integrações daquele projeto.

## Como usar em um projeto novo

1. Copie a pasta inteira para o projeto. Pode ser na raiz (`sdd/`) ou direto em
   `.claude/skills/` — as duas formas funcionam.

   > **Atenção ao copiar:** o bundle contém a pasta oculta `.sdd/`, e `cp -r sdd/* destino/`
   > **não** a copia — o glob do shell ignora nomes que começam com ponto. Use
   > `cp -a sdd/. destino/`, ou copie a pasta inteira de uma vez. Sem a `.sdd/`, as skills do
   > fluxo param por falta das baselines.

2. Peça para a IA configurar, de um destes dois jeitos:
   - **Qualquer ferramenta de IA:** aponte o caminho e peça, ex.: *"leia
     `sdd/.sdd/configurar-sdd/SKILL.md` e siga as instruções para configurar o SDD neste projeto"*
   - **Claude Code, com o bundle em `.claude/skills/`:** aponte o caminho na primeira vez
     (`.claude/skills/.sdd/configurar-sdd/SKILL.md`). Depois disso existe o atalho
     `/configurar-sdd`.
3. A skill `configurar-sdd` lê os arquivos de regra do projeto, explora o repositório, pergunta
   só o que não conseguiu detectar, grava o `sdd.config.md` e — no Claude Code — cria o atalho
   `/configurar-sdd`.
4. A partir daí, use as skills do fluxo normalmente.

## O fluxo

| Skill | O que faz | Entrada | Saída |
| --- | --- | --- | --- |
| `configurar-sdd` | Adapta o bundle ao projeto | o repositório | `sdd.config.md` |
| `criar-prd` | Levanta requisitos (o quê e por quê) | pedido da feature | `prd.md` |
| `criar-techspec` | Decide arquitetura e testes (o como) | `prd.md` | `techspec.md` |
| `criar-tasks` | Quebra em tarefas executáveis | `prd.md` + `techspec.md` | `tasks.md` + tarefas |
| `executar-task` | Implementa uma tarefa com TDD | `tasks.md` | código + testes |
| `executar-qa` | Valida contra os requisitos | tudo acima | `qa.md` + `bugs.md` |
| `executar-bugfix` | Corrige defeitos na causa raiz | `bugs.md` | `bugfixes.md` |
| `executar-review` | Revisa código e conformidade | o diff | `codereview.md` |

## Estrutura

```
criar-prd/  criar-techspec/  criar-tasks/
executar-task/  executar-qa/  executar-bugfix/  executar-review/
.sdd/
  _shared/          baselines de segurança e qualidade, carregadas pelas skills
  configurar-sdd/   a skill de adaptação e o esqueleto do arquivo de configuração
sdd.config.md       gerado por projeto
```

As 7 pastas do fluxo ficam na raiz porque são o que você usa no dia a dia — no Claude Code, são
exatamente as 7 skills que aparecem em `.claude/skills/`. Tudo que é infraestrutura vive em
`.sdd/`, oculta, fora do caminho.

O `sdd.config.md` fica visível de propósito: **é o único arquivo que muda de projeto para
projeto**, e você deve lê-lo, editá-lo e versioná-lo.

Os caminhos são todos relativos (`../sdd.config.md`, `../.sdd/_shared/...`), então os dois modos
de instalação — na raiz do projeto ou em `.claude/skills/` — continuam funcionando sem alteração
nenhuma nas skills.

## O que é fixo e o que é configurável

**Fixo** (vale para qualquer projeto): o fluxo em si, TDD, corrigir causa raiz em vez de sintoma,
a baseline de segurança e a de qualidade, e a regra de nunca baixar o threshold de cobertura para
fazer o gate passar.

**Configurável** (vem do `sdd.config.md`): linguagem, framework, arquitetura e ordem de camadas,
comandos de lint/teste/cobertura e o threshold, rastreador de issues, ferramenta de design, tipo
de produto e método de validação visual, acessibilidade e i18n, áreas read-only, convenções de
commit/branch/PR e de comentário, e onde documentar.

## Reconfigurar depois

Rode `configurar-sdd` de novo — no Claude Code, `/configurar-sdd`. Ela atualiza o
`sdd.config.md` seção por seção, preservando o que você editou à mão, e mostra o que muda antes
de gravar. As skills não são reescritas em nenhum momento.

## Versionamento

No bundle mestre, `sdd.config.md` é ignorado pelo git, porque é gerado por projeto. Dentro de um
projeto que usa o SDD, versione o `sdd.config.md` normalmente: ele documenta as convenções
acordadas pelo time.
