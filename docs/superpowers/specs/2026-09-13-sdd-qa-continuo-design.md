# SDD — Grupo de skills de QA contínuo (`qa/`)

**Data:** 2026-09-13
**Status:** Aprovado para plano de implementação

## Contexto e problema

O bundle SDD hoje (`criar-prd` → `criar-techspec` → `criar-tasks` → `executar-task` →
`executar-qa` → `executar-bugfix` → `executar-review`) cobre o ciclo de vida de **uma feature
por vez**. `executar-qa` já funciona bem como skill de QA para esse recorte: um QA humano pode
rodá-la isoladamente, sem passar pelas outras seis, assim que `prd.md`/`techspec.md`/`tasks.md`
existem e o código está pronto.

O que esse recorte não cobre é atividade de QA **desacoplada de uma feature/PRD única**:
regressão do produto inteiro, teste exploratório guiado por persona, triagem de defeito
cross-feature e sign-off de release. Todo artefato do fluxo atual (`run.yaml`, `memoria/`,
`bugs.md`) é chaveado por `tasks/prd-<nome-da-feature>/` — não existe unidade de trabalho para
"testar algo que não nasceu de um `criar-prd`" ou para "revarrer o produto inteiro antes de uma
release".

Objetivo deste spec: acrescentar duas skills novas — `criar-plano-qa` e `executar-sessao-qa` —
que cobrem esse gap, **sem alterar nenhuma das 7 skills existentes**, agrupadas fisicamente numa
pasta própria (`qa/`) para não crescer a listagem plana de `.claude/skills/` a cada nova adição.

**Inspiração:** as skills `qa-report` e `qa-execution` do repositório
`https://github.com/pedronauck/skills` (planejamento vivo em `docs/qa/` com personas, jornadas,
cenários e charters; execução de sessão persona-driven com matriz de cobertura e evidência
escrita de volta na árvore). O repositório **não declara licença** — nenhum texto de lá é
copiado; as ideias foram reescritas do zero em português, no padrão de tags/critical/checklist
que este bundle já usa.

**Achado técnico que molda o desenho:** o Claude Code **não descobre** `SKILL.md` aninhado num
subnível de `.claude/skills/` (ex.: `.claude/skills/qa/executar-sessao-qa/SKILL.md` não vira
skill invocável — é limitação documentada da ferramenta, não deste bundle). O bundle já resolve
esse mesmo problema hoje para `.sdd/configurar-sdd` e `.sdd/atualizar-sdd`, gravando um atalho em
`.claude/commands/<skill>.md` que aponta para o caminho real. Este spec reusa o mesmo mecanismo
para o grupo `qa/`, em vez de propor algo novo.

## Objetivos

1. Duas skills novas cobrem QA de produto/release, decoupled de uma feature única:
   `criar-plano-qa` (mantém a árvore viva de QA) e `executar-sessao-qa` (roda uma sessão
   persona-driven contra um escopo — branch, PR ou release).
2. Ficam agrupadas em `qa/` na raiz do bundle, e continuam invocáveis por linguagem
   natural/atalho no Claude Code através do mesmo mecanismo de atalho que `.sdd/` já usa —
   nenhuma mudança na forma como o Claude Code descobre skill nenhuma.
3. Novo modelo de artefato, próprio dessa camada: uma única árvore viva por produto (padrão
   `docs/qa/`, caminho configurável), com a mesma disciplina de `run.yaml`/`memoria/<skill>.md`
   retomável que as 7 skills já usam — só que na raiz da árvore, não por feature.
4. Bug encontrado numa sessão:
   - se atribuível a uma feature com pasta em `<artefatos_sdd>`, entra também no `bugs.md` dela e
     segue o `executar-bugfix` normal, sem alteração nenhuma nesse contrato;
   - se não for atribuível a nenhuma feature (achado exploratório/cross-cutting), fica no
     backlog da árvore, marcado como não atribuído, para triagem humana.
5. Camada **opt-in**: `configurar-sdd` pergunta uma vez se o projeto quer essa camada. Projeto
   que responde "não" nunca ganha a pasta `qa/` nem os atalhos, e nenhuma das 7 skills muda de
   comportamento.
6. Mesmas convenções não-negociáveis do resto do bundle: bloco de tags padrão
   (`<config>`/`<baseline_seguranca>`/`<baseline_qualidade>`/`<regras_confianca>`), mesma
   disciplina de passo-a-passo em `run.yaml`, mesmo checklist de qualidade no fim de cada skill,
   mesma severidade de bug (Crítica/Alta/Média/Baixa) já usada em `executar-qa`/`executar-bugfix`
   — sem introduzir uma segunda escala de severidade.

## Não-objetivos

- Não migrar nem alterar a árvore por feature existente (`tasks/prd-<slug>/`) — a árvore nova é
  aditiva, nunca substitui artefato do fluxo atual.
- Não criar uma terceira skill de correção de bug, nem duplicar a disciplina de TDD/causa-raiz
  já escrita em `executar-bugfix` — reuso é a decisão tomada (ver "Integração com o fluxo
  existente").
- Não copiar texto de `pedronauck/skills` — inspiração de ideias, conteúdo próprio.
- Não contornar a limitação de descoberta aninhada do Claude Code nem pedir suporte novo à
  ferramenta — usar o mecanismo de atalho já existente no bundle.
- Não tornar essa camada obrigatória — projeto sem QA contínuo continua funcionando exatamente
  como hoje.
- Não cobrir teste de carga/performance como frente nova — fica para um spec futuro, se surgir
  necessidade.

## Arquitetura da solução

| Camada | O quê | Onde vive | Quem escreve |
| --- | --- | --- | --- |
| **Skills novas** | `criar-plano-qa` e `executar-sessao-qa` | `qa/<nome>/SKILL.md` na raiz do bundle | Mantenedor do template |
| **Atalhos Claude Code** | `.claude/commands/criar-plano-qa.md` e `.claude/commands/executar-sessao-qa.md` | Gerados por `configurar-sdd`, mesmo mecanismo de `.sdd/` | `configurar-sdd` |
| **Árvore viva de QA** | `run.yaml` + `memoria/` + personas/jornadas/cenários/charters/bugs/relatórios | Pasta configurável (`<qa_continuo>` do config, padrão `docs/qa/`) **no projeto**, não no bundle | As duas skills novas, ao rodar |
| **Configuração** | Novo bloco `<qa_continuo>` | `sdd.config.md`, ao lado dos blocos existentes | `configurar-sdd` |

### Descoberta e invocação no Claude Code

`qa/criar-plano-qa/SKILL.md` e `qa/executar-sessao-qa/SKILL.md` não são descobertos sozinhos pelo
Claude Code por estarem um nível abaixo de `.claude/skills/`. `configurar-sdd` grava, condicionado
a `<qa_continuo>` estar ativo:

```md
---
description: Mantém a árvore viva de QA contínuo do produto
---

Leia `<CAMINHO_SKILL>` e siga integralmente as instruções dessa skill para planejar/atualizar o QA
contínuo deste projeto.
```

em `.claude/commands/criar-plano-qa.md`, e o equivalente para `executar-sessao-qa.md` — mesmo
template que já existe para `configurar-sdd`/`atualizar-sdd`, só trocando a descrição e o caminho.
Fora do Claude Code, ou se o usuário não quiser os atalhos, as duas skills continuam funcionando
por caminho explícito, como qualquer skill deste bundle.

### Inventário de arquivos

**Novos:**

- `qa/criar-plano-qa/SKILL.md`
- `qa/criar-plano-qa/references/TEMPLATE_PERSONAS.md`
- `qa/criar-plano-qa/references/TEMPLATE_JOURNEY.md`
- `qa/criar-plano-qa/references/TEMPLATE_SCENARIO.md`
- `qa/criar-plano-qa/references/TEMPLATE_CHARTER.md`
- `qa/executar-sessao-qa/SKILL.md`
- `qa/executar-sessao-qa/references/TEMPLATE_REPORT.md`
- `qa/executar-sessao-qa/references/TEMPLATE_BUG.md`

**Modificados:**

- `.sdd/configurar-sdd/SKILL.md` — nova etapa de pergunta (`<qa_continuo>` ativo?), etapa 6
  (cópia para `.claude/skills/`) e etapa 7 (atalhos) passam a incluir `qa/` condicionalmente.
- `.sdd/configurar-sdd/references/CONFIG_TEMPLATE.md` — novo bloco `<qa_continuo>`.
- `.sdd/configurar-sdd/references/COMANDO_CLAUDE_CODE.md` — generalizado: hoje descreve só os
  atalhos de `.sdd/*`; passa a documentar o mesmo procedimento para qualquer skill fora do
  primeiro nível, com `.sdd/*` e `qa/*` como as duas instâncias atuais.
- `.sdd/atualizar-sdd/SKILL.md` — etapa 2 (comparação) passa a incluir `qa/` quando presente na
  cópia local, ao lado das 7 pastas de fluxo e de `.sdd/`.
- `README.md` — nova seção descrevendo o grupo opcional `qa/`; instruções de cópia passam a
  mencionar `qa/` como parte condicional da cópia.
- `VERSION` — sobe ao final da implementação.

**Não tocados:** `criar-prd`, `criar-techspec`, `criar-tasks`, `executar-task`, `executar-qa`,
`executar-bugfix`, `executar-review`, e os `_shared/*_BASELINE.md`.

## Esquema — novo bloco `<qa_continuo>` em `sdd.config.md`

```md
<qa_continuo>
Ativo: [sim | não]
Pasta raiz da árvore viva de QA: [padrão: `docs/qa/`]
Arquivos/pastas: run.yaml · memoria/<skill>.md · personas.md · journeys/J-<slug>.md ·
  scenarios/<AREA>-<slug>.md · charters/CH-<slug>.md · bugs/BUG-<AAAAMMDD>-<slug>.md ·
  reports/<AAAA-MM-DD>-<escopo>.md
</qa_continuo>
```

Campo ausente ou `Ativo: não` é o guard que as duas skills novas checam antes de qualquer ação —
mesmo padrão de guard que as 7 skills já usam para `sdd.config.md` ausente. Severidade de bug
**não** ganha campo próprio aqui: reusa a mesma escala (Crítica/Alta/Média/Baixa) já implícita em
`executar-qa`/`executar-bugfix`/`executar-review`, para que um bug nascido numa sessão de QA
contínuo e um bug nascido no `executar-qa` de uma feature sejam triados com a mesma régua.

## Skill nova: `criar-plano-qa`

**Gatilhos de descrição:** "planeja o QA do produto", "mapeia jornadas de usuário", "cria/atualiza
o plano de QA contínuo", "prepara ciclo de regressão/exploratório".

**Localização dos arquivos** (conforme `<qa_continuo>`): `run.yaml` e `memoria/criar-plano-qa.md`
na raiz da árvore; `personas.md`; `journeys/J-<slug>.md`; `scenarios/<AREA>-<slug>.md`;
`charters/CH-<slug>.md`.

**Fluxo:**

1. **Resolver/inicializar a árvore** — ler `<qa_continuo>`; se `Ativo: não` ou o campo não
   existir, parar e apontar para `configurar-sdd`. Ler o `run.yaml` próprio da árvore (não o de
   nenhuma feature) e a `memoria/criar-plano-qa.md`; retomar no primeiro passo não concluído se
   houver entrada `em_andamento`.
2. **Estabelecer/atualizar personas** — atualizar `personas.md` só quando a audiência mudou;
   perguntar ao usuário perfis reais de quem usa o produto (não personas de marketing).
3. **Mapear jornadas como fluxograma** — criar/atualizar `journeys/J-<slug>.md` com um diagrama
   Mermaid por jornada: entrada → ações → desvios → estado final, incluindo o caminho de
   abandono.
4. **Derivar cenários** — gerar `scenarios/<AREA>-<slug>.md` a partir de cada fluxograma, citando
   a jornada de origem. Cenário deriva de jornada, nunca o contrário.
5. **Planejar charters de sessão** — criar `charters/CH-<slug>.md` ligando persona + jornada +
   tour (o que exercitar além do caminho principal) + time-box, para o tipo de ciclo pedido
   (smoke/direcionado/completo).
6. **Reconciliar duplicatas (obrigatório antes de gravar)** — jornada, cenário ou charter que
   descreve o mesmo comportamento de um já existente se funde no id mais antigo: campos
   mesclados, referências atualizadas, duplicata removida. Evita que duas sessões paralelas
   criem dois IDs para a mesma coisa.
7. **Validar completude** — toda jornada em escopo tem fluxograma, ao menos um charter e cenários
   com status real (não "a definir").
8. **Fechar** — `run.yaml`: `status: concluida`, `veredito` (nº de jornadas/cenários/charters),
   `proxima_acao` apontando para `executar-sessao-qa`. `memoria/criar-plano-qa.md`: personas
   descartadas, jornadas fundidas e por quê, premissas em aberto.

<critical>Cenário sem jornada de origem é lacuna a sinalizar ao usuário, nunca a preencher por
conta própria.</critical>
<critical>Nunca reescreva `personas.md`/jornada/cenário do zero quando já existem — atualize e
funda, do jeito que a <memoria> do resto do bundle já é append-only.</critical>

## Skill nova: `executar-sessao-qa`

**Gatilhos de descrição:** "roda uma sessão de QA", "faz a regressão antes da release", "testa
como um usuário real", "valida se está pronto para lançar".

**Localização dos arquivos:** `run.yaml`/`memoria/executar-sessao-qa.md` na raiz da árvore;
`reports/<AAAA-MM-DD>-<escopo>.md`; `bugs/BUG-<AAAAMMDD>-<slug>.md`.

**Fluxo:**

1. **Preparação** — ler `README.md`/cenários/bugs abertos da árvore e o `run.yaml`; fixar o
   escopo (branch, PR ou release) e as pré-condições: ambiente acessível, autenticação real,
   build em paridade com produção. Pré-condição ausente marca a sessão `Blocked`, nunca
   "contorna".
2. **Montar a matriz de cobertura** — a partir do(s) charter(s) relevantes ao escopo: persona ×
   jornada × tour × time-box. Criar `reports/<data>-<escopo>.md` com todas as linhas `Pending`
   antes de começar.
3. **Percorrer cada jornada como a persona** — do início ao fim real, **sempre por uma superfície
   que um usuário real alcança** (nunca atalho de dev-tools), capturando evidência a cada
   checkpoint.
4. **Rodar o tour do charter e os edge cases selecionados.**
5. **Aplicar lentes de risco adicionais quando fizer sentido** (acessibilidade, i18n,
   segurança) — reusando `SECURITY_BASELINE.md`/`QUALITY_BASELINE.md` e as extensões do config,
   igual ao que `executar-qa` já faz por feature.
6. **Deduplicar e registrar achados** em `bugs/BUG-<AAAAMMDD>-<slug>.md`, com a severidade
   Crítica/Alta/Média/Baixa. Achado que já existe funde no id mais antigo.
7. **Encaminhar cada bug** — se atribuível a uma feature com pasta em `<artefatos_sdd>`: anexar
   uma entrada equivalente no `bugs.md` dela (mesmo formato que `executar-qa` já grava), com um
   link de volta ao ID da árvore viva, e apontar o usuário para `executar-bugfix` daquela feature.
   Se não for atribuível a nenhuma feature: manter no backlog da árvore, marcado **não
   atribuído**, para triagem humana — esta skill nunca corrige código.
8. **Fechar** — a sessão só termina com a matriz sem nenhuma linha `Pending`. Relatório final com
   veredito e contagem por severidade; declara prontidão de release apenas se toda evidência
   obrigatória existir. `run.yaml` fechado (mesma disciplina de rodada das outras skills: entrada
   nova por sessão, nunca sobrescreve a anterior); `memoria/executar-sessao-qa.md` com o que não
   virou bug e por quê.

<critical>Toda verificação passa por uma superfície que um usuário real alcança — sem atalho de
dev-tools, sem UI otimista como confirmação.</critical>
<critical>Achado duplicado não deduplicado é erro desta skill, não do usuário — deduplicar é
etapa obrigatória, não best-effort.</critical>
<critical>Nunca declare prontidão de release com evidência obrigatória faltando.</critical>

## Integração com o fluxo existente

`executar-bugfix` **não muda de contrato**: continua recebendo `--prd <slug>` e lendo
`bugs.md` daquela feature. Quando `executar-sessao-qa` atribui um achado a uma feature existente,
ela escreve nesse mesmo `bugs.md`, no mesmo formato que `executar-qa` já usa — `executar-bugfix`
roda exatamente como hoje, sem precisar saber se a origem foi uma rodada de QA por feature ou uma
sessão de QA contínuo. Rastreabilidade nos dois sentidos: a entrada em `bugs.md` cita o ID de
origem em `docs/qa/bugs/`, e o registro na árvore viva cita a feature para onde foi encaminhado.

Nenhuma linha de `criar-prd`, `criar-techspec`, `criar-tasks`, `executar-task`, `executar-qa`,
`executar-bugfix` ou `executar-review` muda.

## Fluxo de uso ponta a ponta (exemplo)

1. Projeto já tem o SDD configurado (7 skills + `.sdd/`). Time de QA pede para ativar QA
   contínuo; `configurar-sdd` roda de novo, pergunta e grava `<qa_continuo>` com
   `Ativo: sim` e pasta `docs/qa/`, copia `qa/` para `.claude/skills/` e grava os dois atalhos.
2. QA chama `/criar-plano-qa`. A skill não encontra árvore ainda, cria `docs/qa/` com
   `personas.md`, mapeia as 5 jornadas principais do produto com fluxograma Mermaid, deriva
   cenários, monta 2 charters (smoke e regressão completa).
3. Antes de uma release, QA chama `/executar-sessao-qa` com escopo "release 2.4". A sessão
   percorre as 5 jornadas como as personas mapeadas, encontra 2 bugs: um no fluxo de checkout
   (atribuível à feature `pagamento-pix`, que tem pasta em `tasks/prd-pagamento-pix/`) e um na
   busca (não atribuível a nenhuma feature rastreada — é regressão antiga).
4. O bug do checkout entra em `tasks/prd-pagamento-pix/bugs.md`; o usuário chama
   `executar-bugfix --prd pagamento-pix` normalmente. O bug da busca fica em
   `docs/qa/bugs/BUG-20260913-busca-lenta.md`, marcado não atribuído, para triagem humana.
5. Relatório final em `docs/qa/reports/2026-09-13-release-2.4.md` declara release não pronta
   (2 bugs abertos, 1 de severidade Alta) até o checkout ser corrigido e a sessão rodar de novo.

## Validação

Como este é um bundle de prompts/templates, a validação é por inspeção e simulação, no mesmo
padrão do spec anterior deste bundle:

1. **Simulação a seco:** aplicar mentalmente `criar-plano-qa` → `executar-sessao-qa` contra um
   produto fictício com 3-4 jornadas, incluindo pelo menos um bug atribuível e um não atribuível,
   e conferir que os dois casos do passo 7 de `executar-sessao-qa` funcionam.
2. **Sweep de termos proibidos:** as duas skills novas passam pelo mesmo `grep` de termos de
   stack hard-coded que o resto do bundle já usa (ver plano de 2026-09-12).
3. **Checklist de paridade de convenção:** as duas skills novas têm o mesmo bloco de tags padrão,
   a mesma disciplina de `run.yaml`/`memoria` passo-a-passo, e terminam com "Checklist de
   qualidade", igual às 7 existentes.
4. **Conferir que nenhuma das 7 skills existentes foi tocada** (`git diff` vazio nelas).

## Riscos e premissas

- **Premissa:** o time mantém `docs/qa/` versionado junto do repositório do projeto (não do
  bundle) — é "living doc", análogo a `tasks/`.
- **Risco:** produto sem superfície pública estável para uma persona percorrer (ex.: biblioteca
  interna, CLI sem ambiente hospedado) não se encaixa no modelo de `executar-sessao-qa`.
  Mitigação: a skill lê `<ui_projeto>`, e quando não há superfície de usuário final aplicável,
  passa a exercitar o contrato diretamente (chamada de função/endpoint/comando), do mesmo jeito
  que `executar-qa` já faz hoje — nenhuma pergunta nova de config é criada para isso, reusa
  `<ui_projeto>`.
- **Risco:** árvore viva crescer descontroladamente ao longo do tempo. Mitigação: a etapa 6 de
  `criar-plano-qa` e a etapa 6 de `executar-sessao-qa` tornam a deduplicação obrigatória, não
  best-effort.
- **Premissa:** `pedronauck/skills` serviu só de inspiração conceitual; nenhum trecho de texto de
  lá aparece neste bundle.
- **Risco:** duas escalas de severidade coexistindo (a do bundle e uma nova, importada) — mitigado
  neste spec ao decidir explicitamente reusar a escala existente (Crítica/Alta/Média/Baixa) em vez
  de importar a rubrica de 5 níveis do `pedronauck/skills`.
