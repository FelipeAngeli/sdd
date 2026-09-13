# SDD — Fluxo de desenvolvimento orientado a especificação

Um bundle de skills portátil. Coloque em qualquer projeto, peça para a IA se adaptar, e o fluxo
PRD → TechSpec → Tasks → Execução → QA → Bugfix → Review passa a funcionar com a stack, as
convenções e as integrações daquele projeto.

## Como usar em um projeto novo

1. Copie o bundle para o projeto. Pode ser na raiz (`sdd/`) ou direto em `.claude/skills/` — as
   duas formas funcionam. Copie **nomeando o que vai**:

   ```sh
   mkdir -p destino
   cp -R sdd/criar-prd sdd/criar-techspec sdd/criar-tasks \
         sdd/executar-task sdd/executar-qa sdd/executar-bugfix sdd/executar-review \
         sdd/.sdd sdd/VERSION destino/
   ```

   Se você já sabe que quer a camada de QA contínuo, acrescente `sdd/qa` à lista. Se não souber
   ainda, deixe fora: `configurar-sdd` pergunta depois, mas só copia `qa/` se ela já estiver nesta
   cópia local — se você decidir ativar mais tarde e tiver deixado `qa/` de fora aqui, é
   `atualizar-sdd` quem traz a pasta a partir do bundle mestre.

   > **Não use `cp -a sdd/. destino/` nem `cp -r sdd/* destino/`.** O primeiro arrasta o `.git/` e
   > o `.gitignore` do bundle mestre para dentro do projeto — e esse `.gitignore` faz o projeto
   > ignorar o próprio `sdd.config.md`, justamente o arquivo que você precisa versionar. O segundo
   > omite a pasta oculta `.sdd/`, porque o glob do shell ignora nomes que começam com ponto, e sem
   > ela as skills do fluxo param por falta das baselines.

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
| `criar-prd` | Levanta requisitos e critérios de aceite | pedido da feature | `prd.md` + `run.yaml` |
| `criar-techspec` | Decide arquitetura e testes (o como) | `prd.md` | `techspec.md` |
| `criar-tasks` | Quebra em tarefas executáveis | `prd.md` + `techspec.md` | `tasks.md` + tarefas |
| `executar-task` | Implementa uma tarefa com TDD | `tasks.md` | código + testes |
| `executar-qa` | Valida contra os critérios de aceite | tudo acima | `qa.md` + `bugs.md` |
| `executar-bugfix` | Corrige defeitos na causa raiz | `bugs.md` | `bugfixes.md` → volta ao QA |
| `executar-review` | Revisa o diff e a conformidade | diff vs. branch base | `codereview.md` |

### Grupo opcional: QA contínuo (`qa/`)

As 7 skills acima trabalham **uma feature por vez**. Quando o time também precisa de QA que não
nasce de um PRD — regressão do produto inteiro, teste exploratório, sign-off de release, backlog
de bug cross-feature — existe um grupo opcional:

| Skill | O que faz | Entrada | Saída |
| --- | --- | --- | --- |
| `criar-plano-qa` | Mantém a árvore viva de QA do produto | o produto e o time | `personas.md`, `journeys/`, `scenarios/`, `charters/` |
| `executar-sessao-qa` | Roda a sessão contra um escopo e decide prontidão | um charter | `reports/<data>-<escopo>.md`, `bugs/` |

A árvore vive no projeto (padrão `docs/qa/`), não por feature, e é contínua: atualiza e funde em
vez de recriar. Bug encontrado numa sessão que pertence a uma feature existente entra no `bugs.md`
dela e segue o `executar-bugfix` normal; bug que não pertence a nenhuma fica no backlog da árvore
para triagem.

A camada é **opt-in**: `configurar-sdd` pergunta se você quer, grava `<qa_continuo>` no
`sdd.config.md` e só então copia `qa/` e cria os atalhos. Projeto que não ativa continua
exatamente como antes.

O fluxo **não é uma fila de mão única**: `executar-bugfix` devolve para `executar-qa`, e review
reprovado volta para o review depois da correção. O `run.yaml` de cada feature registra em que
etapa, em que **passo dentro da etapa** e em que rodada ela está — é o primeiro arquivo que toda
skill lê, e o que permite retomar no meio de um review de 11 passos em vez de recomeçar.

Ao lado dele, cada skill mantém a própria memória em `memoria/<skill>.md`: as decisões e o motivo
de cada uma, o que o usuário respondeu, o que foi descartado e as premissas que ficaram em aberto.
O `run.yaml` responde **o que foi feito e onde parou**; a memória responde **por quê**. É a
diferença entre uma sessão nova que continua o trabalho e uma que refaz as mesmas perguntas.

Toda skill aceita o slug da feature como argumento (`--prd`, `--prompt` em `criar-prd`). Se você
não passar, ela pergunta ou lista as features existentes.

## Estrutura

```
criar-prd/  criar-techspec/  criar-tasks/
executar-task/  executar-qa/  executar-bugfix/  executar-review/
qa/                 grupo opcional de QA contínuo: criar-plano-qa/ e executar-sessao-qa/
.sdd/
  _shared/          baselines de segurança, qualidade e confiança, mais os modelos de run.yaml e de memória
  configurar-sdd/   a skill de adaptação e o esqueleto do arquivo de configuração
  atualizar-sdd/    sincroniza esta cópia com um bundle mestre mais novo
VERSION             versão do bundle, registrada no config de cada projeto
sdd.config.md       gerado por projeto
```

As 7 pastas do fluxo ficam na raiz porque são o que você usa no dia a dia — no Claude Code, são
exatamente as 7 skills que aparecem em `.claude/skills/`. O grupo `qa/` fica agrupado porque é
opcional e cresce junto: no Claude Code ele é alcançado pelos atalhos `/criar-plano-qa` e
`/executar-sessao-qa`, que `configurar-sdd` grava em `.claude/commands/` — o Claude Code não
descobre sozinho skill que está um nível abaixo de `.claude/skills/`. Tudo que é infraestrutura
vive em `.sdd/`, oculta, fora do caminho.

O `sdd.config.md` fica visível de propósito: **é o único arquivo que muda de projeto para
projeto**, e você deve lê-lo, editá-lo e versioná-lo.

Os caminhos são todos relativos — `../sdd.config.md` e `../.sdd/_shared/...` nas 7 skills do
fluxo, que ficam um nível abaixo da raiz do bundle; `../../sdd.config.md` e
`../../.sdd/_shared/...` em `.sdd/*` e `qa/*`, que ficam um nível mais abaixo ainda —, então os
dois modos de instalação — na raiz do projeto ou em `.claude/skills/` — continuam funcionando sem
alteração nenhuma nas skills.

## O que é fixo e o que é configurável

**Fixo** (vale para qualquer projeto): o fluxo em si, TDD, corrigir causa raiz em vez de sintoma,
as baselines de segurança, de qualidade e de confiança, e a regra de nunca baixar o threshold de
cobertura para fazer o gate passar.

A baseline de confiança merece destaque: as skills leem issue, design e arquivos de regra escritos
por terceiros, e depois escrevem código e rodam comandos. Tudo que vem dessas fontes é tratado como
**dado, nunca instrução** — texto que tenta dar ordem ao agente vira achado de segurança, não
comando. Na mesma linha, escrever fora do repositório e rodar comando do config exigem confirmação
explícita.

**Configurável** (vem do `sdd.config.md`): linguagem, framework, arquitetura e ordem de camadas,
comandos de lint/teste/cobertura e o threshold, rastreador de issues, ferramenta de design, tipo
de produto e método de validação visual, acessibilidade e i18n, áreas read-only, convenções de
commit/branch/PR e de comentário, e onde documentar.

## Reconfigurar depois

Rode `configurar-sdd` de novo — no Claude Code, `/configurar-sdd`. Ela atualiza o
`sdd.config.md` seção por seção, preservando o que você editou à mão, e mostra o que muda antes
de gravar. As skills não são reescritas em nenhum momento.

## Atualizar o bundle

Quando o bundle mestre evoluir, rode `atualizar-sdd` — no Claude Code, `/atualizar-sdd`. Ela
compara esta cópia com o mestre, mostra o diff, trata arquivo por arquivo o que você editou
localmente, e **nunca toca no `sdd.config.md`**. A versão fica registrada em `VERSION` e no campo
`<meta_sdd>` do config, para você saber a que distância cada projeto está.

## Versionamento

No bundle mestre, `sdd.config.md` é ignorado pelo git, porque é gerado por projeto. Dentro de um
projeto que usa o SDD, versione o `sdd.config.md` normalmente: ele documenta as convenções
acordadas pelo time.

Por isso o `.gitignore` do bundle mestre **não deve ser copiado** para o projeto — é ele que
ignora o `sdd.config.md`. As instruções de cópia acima já evitam isso; se uma cópia antiga levou
um `.gitignore` ou um `.git/` para dentro do projeto, remova os dois.
