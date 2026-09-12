# Verificação final do bundle SDD generalizado

Task 13 do plano `2026-09-12-sdd-global-template`. Este documento registra os sweeps automatizados
(Steps 1-3, já executados antes desta sessão), as duas simulações a seco (Steps 4-5) e o checklist
de paridade com a versão Flutter original (Step 6).

Diretório verificado: raiz deste repositório (bundle mestre do SDD), branch `main`.

---

## Steps 1-3 — Sweeps automatizados

Executados antes desta sessão. Resultados confirmados:

### Step 1 — sweep de termos proibidos

```bash
grep -rniE 'flutter|\bdart\b|pubspec|\bbloc\b|cubit|modular|dartz|\bdio\b|mocktail|nestjs|golden|talkback|voiceover|cy-loop|gpt-5|LIG-[0-9]|Liga Coop|tool/coverage|make (ci|coverage|analyze|test|format)' \
  criar-* executar-* configurar-sdd/SKILL.md _shared/ README.md 2>/dev/null \
  | grep -viE 'ex\.?:|por exemplo|exemplo:'
```

Saída: **vazia**. Confirmado limpo.

**Observação adicional feita nesta sessão:** o alvo literal do comando (`configurar-sdd/SKILL.md`,
sem incluir `configurar-sdd/references/`) deixa `configurar-sdd/references/CONFIG_TEMPLATE.md` fora
do sweep original. Rodei o mesmo padrão manualmente contra esse arquivo para fechar a lacuna:

```bash
grep -rniE 'flutter|\bdart\b|pubspec|\bbloc\b|cubit|modular|dartz|\bdio\b|mocktail|nestjs|golden|talkback|voiceover|cy-loop|gpt-5|LIG-[0-9]|Liga Coop|tool/coverage|make (ci|coverage|analyze|test|format)' \
  configurar-sdd/references/
```

Único hit: `configurar-sdd/references/CONFIG_TEMPLATE.md:38: Ferramenta de validação visual: [ex.:
Playwright screenshot; ex.: golden test; ou "não aplicável"]` — está dentro de uma cláusula `ex.:`
(um exemplo genérico de técnica de teste visual usada em várias stacks, não específico de
Flutter), então teria sido filtrado pelo `grep -v` original mesmo se o caminho tivesse sido
incluído. Não é um achado de conteúdo, é uma lacuna de cobertura do comando do Step 1 em si — sem
consequência neste caso, mas registrada para transparência.

Também rodei um sweep suplementar com termos que não estavam na lista original (`riverpod`,
`provider`, `getx`, `widget_test`, `integration_test`, `flutter_bloc`, `sqflite`, `hive`,
`shared_preferences`, `appium`, `detox`, `xcode`, `gradle`, `android`, `ios`, `widget`,
`nestjs`) contra todo o bundle (`criar-* executar-* configurar-sdd/ _shared/ README.md`): saída
**vazia**.

### Step 2 — referência ao config pelo caminho certo

```bash
for d in criar-prd criar-techspec criar-tasks executar-task executar-qa executar-bugfix executar-review; do
  c=$(grep -c '\.\./sdd\.config\.md' "$d/SKILL.md")
  echo "$d: $c"
done
```

Todas as 7 skills retornam 3 ocorrências (acima do mínimo de 2 exigido). Confirmado.

### Step 3 — nenhum caminho antigo de config

```bash
grep -rn '\.claude/sdd\.config\|@\.claude/CLAUDE\.md\|@rules\.md\|@AGENTS\.md' \
  criar-* executar-* configurar-sdd/ _shared/ 2>/dev/null
```

Saída: **vazia**. Confirmado — esses caminhos hoje só aparecem como algo que `configurar-sdd`
*procura* (`CLAUDE.md`, `.claude/CLAUDE.md`, `AGENTS.md`, `rules.md`, `.cursorrules`,
`CONTRIBUTING.md`, `CODEOWNERS`, na seção "1. Ler as regras existentes do projeto"), nunca como
caminho fixo referenciado pelas skills do fluxo.

---

## Step 4 — Simulação a seco: projeto web React

Perfil fictício:

```
package.json: react, vite, vitest, @testing-library/react
scripts: dev, build, lint (eslint), test (vitest), test:coverage (sem threshold)
estrutura: src/components, src/hooks, src/pages, src/lib
sem CLAUDE.md, sem rules.md, sem commitlint, sem template de PR
```

Percorrendo `configurar-sdd/SKILL.md` seção por seção:

**Seção 1 — regras existentes.** Nenhum de `CLAUDE.md`, `.claude/CLAUDE.md`, `AGENTS.md`,
`rules.md`, `.cursorrules`, `CONTRIBUTING.md`, `CODEOWNERS` existe no perfil. Resultado:
`<limites_projeto>` → "Regras invioláveis do projeto: nenhuma encontrada" / "Arquivos de regra
lidos: nenhum encontrado". A ausência é registrada como informação, exatamente como a skill
instrui ("Registre quais arquivos foram encontrados — a ausência deles também é informação").

**Seção 2/3 — exploração e classificação de confiança.**

| Campo do `sdd.config.md` | Classificação | Evidência |
|---|---|---|
| Linguagem(ns) | **detectado** (com ressalva) | `package.json` + extensões reais dos arquivos em `src/` (o perfil não informa `.ts`/`.tsx` vs `.js`/`.jsx` — numa exploração real isso viria da extensão dos arquivos, não só do manifesto) |
| Framework principal | **detectado** | `package.json`: `react`, `vite` |
| Padrão arquitetural | **detectado** (inferência razoável a partir da estrutura) | `src/components, src/hooks, src/pages, src/lib` — organização por tipo técnico (não por feature); a skill pede para "inferir o padrão", não exigir prova formal |
| Ordem de dependência entre camadas | **detectado/incerto** | inferível da convenção React (`lib` → `hooks` → `components` → `pages`), mas é inferência de convenção, não fato lido literalmente — um projeto real poderia violar essa ordem |
| Gerenciador de pacotes | **ausente** | nenhum lockfile mencionado no perfil — em um repo real seria detectado por `package-lock.json`/`pnpm-lock.yaml`/`yarn.lock`; aqui, no perfil dado, não há como classificar sem supor |
| Framework de teste | **detectado** | `package.json`: `vitest`, `@testing-library/react` |
| Biblioteca de mock/dublê | **detectado por convenção** | Vitest inclui `vi.mock` nativo; nenhuma lib de mock adicional listada |
| Lint | **detectado** | script `lint` (eslint) |
| **Format** | **ausente** | não há script `format` nem menção a Prettier no perfil — **vira pergunta** ("há format configurado? Prettier, ou o eslint cobre formatação?") |
| Testes | **detectado** | script `test` (vitest) |
| Cobertura (comando) | **detectado** | script `test:coverage` |
| Cobertura (threshold) | **ausente → pergunta com sugestão** | perfil diz explicitamente "sem threshold"; a skill instrui: "Threshold de cobertura, quando o projeto não define um (sugira um valor e peça confirmação)" |
| E2E | **ausente → "não aplicável"** | nenhuma dependência de Playwright/Cypress no `package.json` do perfil |
| Build | **detectado** | script `build` (vite build) |
| Gate agregado | **ausente → preenchido por default do template**, sem pergunta | template já prevê o fallback "não existe — rodar lint, testes e cobertura em sequência" |
| Scanner de vulnerabilidade | **incerto** | depende do gerenciador de pacotes (ver acima); default seria `npm audit`/`pnpm audit` conforme o lockfile, que o perfil não informa |
| Convenção de commit/branch/PR | **ausente → pergunta** | perfil diz "sem commitlint, sem template de PR" — tema explicitamente listado como "quase sempre pergunta" |
| `<ui_projeto>` — tem UI? | **detectado: sim** | `react`, `src/pages`, `src/components` |
| `<ui_projeto>` — tipo | **detectado: web** | Vite + React é stack de aplicação web por browser |
| Ferramenta de validação visual | **ausente → pergunta** | nenhuma lib de screenshot/Storybook/Chromatic no perfil |
| Acessibilidade aplicável | **incerto → pergunta** | não há evidência positiva (`jest-axe`, `axe-core`, ARIA linter) nem negativa; como é produto web, uma pergunta razoável seria sugerir "sim" (WCAG 2.1 AA) e pedir confirmação, no mesmo espírito da pergunta de threshold |
| i18n aplicável | **ausente → pergunta** | nenhuma lib de i18n (`react-i18next`, `next-intl`) no perfil |

**Confirmando os quatro pontos pedidos no Step 4:**

1. **`<stack_projeto>` e `<comandos_projeto>` saem detectados** — majoritariamente sim: linguagem
   (com ressalva), framework, testes, lint, build e comando de cobertura saem detectados
   diretamente do `package.json`/scripts. **Format** é uma exceção real: o perfil não define um
   script de format, então esse campo específico de `<comandos_projeto>` fica **ausente** e vira
   pergunta — o que é o comportamento correto da skill (não inventar), mas contradiz uma leitura
   apressada de "tudo em `<comandos_projeto>` sai detectado". Vale registrar essa nuance no
   relatório em vez de generalizar.
2. **`<ui_projeto>` sai como web com UI** — confirmado: `Tem interface de usuário final? sim`,
   `Tipo: web`.
3. **Threshold e convenções de commit/PR viram pergunta** — confirmado nos dois casos, com base
   textual explícita do próprio `configurar-sdd/SKILL.md` (seção 4, "Temas que normalmente não são
   detectáveis"): "Threshold de cobertura, quando o projeto não define um (sugira um valor e peça
   confirmação)" e "Convenção de commit/branch/PR, quando não houver `commitlint`, hook ou
   template".
4. **Nenhum campo exige conhecimento de Flutter** — confirmado. Não há em `configurar-sdd/SKILL.md`
   nem em `configurar-sdd/references/CONFIG_TEMPLATE.md` nenhuma pergunta, exemplo obrigatório ou
   ramificação que pressuponha Flutter/Dart/widgets. Os exemplos dados no template
   (`[ex.: TypeScript]`, `[ex.: React 18 + Vite]`, `[ex.: Vitest + Testing Library]`) são
   ilustrativos e coerentes com qualquer stack, inclusive esta.

**Como as skills seguintes se comportariam:**

- `criar-prd`: as perguntas de plataforma seguiriam o ramo `web` do próprio texto: *"navegadores e
  tamanhos de tela alvo? precisa funcionar sem conexão? há requisito de SEO ou de carregamento
  inicial?"* (`criar-prd/SKILL.md`, seção 2). Nenhuma pergunta de mobile (permissão de sistema,
  deep link) seria feita, porque está sob o ramo `mobile`, não `web`.
- `criar-techspec`: mapeamento por camada seguiria a ordem inferida
  `lib → hooks → components → pages` declarada no config.
- `criar-tasks`: sequenciamento seguiria a mesma ordem; toda tarefa que cria uma tela nova em
  `src/pages` **não** teria subtarefa de teste visual obrigatória, porque `<ui_projeto>` não
  declarou ferramenta de validação visual (ficou como pergunta, e se o usuário responder "nenhuma",
  o campo fica "não aplicável" e a condicional de `criar-tasks` — *"SE `<ui_projeto>` INDICAR
  FERRAMENTA DE VALIDAÇÃO VISUAL..."* — simplesmente não dispara).
- `executar-qa`: a seção 4 (Validação visual) usaria exatamente o ramo web —
  *"**Produto web**: automação de navegador (ex.: Playwright) é a ferramenta correta — capture
  screenshot/snapshot de cada tela nos temas e resoluções suportados"* (`executar-qa/SKILL.md`,
  seção 4) — **desde que** o usuário, ao responder a pergunta sobre ferramenta de validação visual,
  tenha indicado alguma (ex.: Playwright). Se o usuário disser que não há nenhuma, a seção inteira
  é pulada mesmo sendo web, o que é coerente (a skill valida o que existe, não o que "deveria"
  existir).

**Achado (Step 4):** nenhum bloqueio, nenhuma instrução sem sentido e nenhuma dependência de
Flutter. O único ponto que merece nota é que **nem todo campo de `<comandos_projeto>` sai
detectado** — `format` é um contraexemplo real dentro do próprio perfil dado — o que é o
comportamento *correto* da skill (perguntar em vez de inventar), mas é preciso dizer isso
explicitamente em vez de simplificar para "tudo detectado".

---

## Step 5 — Simulação a seco: API Python sem UI

Perfil fictício:

```
pyproject.toml: fastapi, sqlalchemy, pytest, pytest-cov (fail_under = 85)
Makefile: lint (ruff), test, coverage
estrutura: app/api, app/services, app/repositories, app/models
CONTRIBUTING.md exige Conventional Commits
```

**Seção 1 — regras existentes.** `CONTRIBUTING.md` existe e exige Conventional Commits. Isso
alimenta diretamente `<colaboracao_projeto>` → "Convenção de commit: Conventional Commits —
detectada em: CONTRIBUTING.md" — **sem precisar de pergunta**, ao contrário do perfil React.

**Seção 2/3 — exploração e classificação:**

| Campo | Classificação | Evidência |
|---|---|---|
| Linguagem | **detectado** | `pyproject.toml` (Python) |
| Framework principal | **detectado** | `fastapi` |
| ORM/acesso a dados | **detectado** | `sqlalchemy` |
| Padrão arquitetural | **detectado** | `app/api, app/services, app/repositories, app/models` — camadas nomeadas explicitamente, mapeamento direto para um padrão em camadas (repository/service/API), evidência mais direta que no caso React |
| Ordem de dependência entre camadas | **detectado** | `models` (entidades) → `repositories` (acesso a dados) → `services` (regra de negócio, consome repositories) → `api` (rotas/entrada, consome services) |
| Framework de teste | **detectado** | `pytest`, `pytest-cov` |
| Lint | **detectado** | `Makefile`: alvo `lint` → `ruff` |
| **Format** | **ausente → pergunta** | `Makefile` só lista `lint`, `test`, `coverage`; nenhum alvo `format`/`ruff format` — mesma lacuna observada no perfil React |
| Testes | **detectado** | `Makefile`: alvo `test` |
| Cobertura (comando) | **detectado** | `Makefile`: alvo `coverage` |
| **Cobertura (threshold)** | **detectado: 85%** | `pyproject.toml`: `pytest-cov ... fail_under = 85` — evidência direta e literal, sem necessidade de pergunta nem de sugestão |
| Gate agregado | **ausente → default do template** | nenhum alvo `make ci`/`make all` mencionado |
| Convenção de commit | **detectado: Conventional Commits** | `CONTRIBUTING.md` |
| Convenção de branch/PR | **ausente → pergunta** | não mencionado no perfil |
| `<ui_projeto>` — tem UI? | **detectado: não** | estrutura é só `api/services/repositories/models`; nenhuma dependência de template engine, frontend, ou pasta de views/components no `pyproject.toml`/estrutura |
| Tipo | **"não aplicável"** | opções do template são `web/mobile/desktop/CLI/não aplicável`; API pura sem UI cai em "não aplicável" |
| Ferramenta de validação visual | **"não aplicável"** | decorre diretamente de não ter UI |
| Acessibilidade aplicável | **não** | decorre de não ter UI |
| i18n aplicável | **não** | decorre de não ter UI |

**Confirmando os quatro pontos pedidos no Step 5:**

1. **Threshold sai detectado como 85** — confirmado, com a citação mais forte de toda a
   simulação: `pytest-cov (fail_under = 85)` é um valor literal no manifesto, não uma inferência.
2. **`<ui_projeto>` sai como "sem UI"** — confirmado (`Tem interface de usuário final? não`).
3. **Convenção de commit detectada do `CONTRIBUTING.md`** — confirmado, e é justamente o exemplo
   que a skill cita como fonte primária desse campo (seção 1 do fluxo).
4. **Camadas viram a ordem de sequenciamento de `criar-tasks`** — confirmado. O texto de
   `criar-tasks/SKILL.md`, seção "Ordem de sequenciamento", diz: *"Use a ordem de dependência entre
   camadas declarada em `<stack_projeto>` do config. O princípio, independente de nomenclatura:
   1. Regra de negócio pura ... 2. Acesso a dados e integrações ... 3. Apresentação/entrada ..."* —
   isso mapeia diretamente para `models/repositories` (grupos 1-2) → `services` (grupo 1,
   regra de negócio) → `api` (grupo 3-4, entrada/registro de rotas) → E2E → documentação.

### Verificação linha a linha: acessibilidade, i18n e validação visual desligando

Fui ao texto de cada uma das quatro skills citadas no brief e confirmei o trecho condicional
exato.

**`criar-prd`** — o ramo de perguntas de plataforma troca completamente para o caso sem UI:

> `criar-prd/SKILL.md`, seção 2: *"- **sem UI (API, serviço, biblioteca)**: contrato público,
> versionamento, compatibilidade retroativa, limites de uso"*

Nenhuma pergunta de acessibilidade, tema claro/escuro ou i18n seria feita por essa seção para o
perfil Python. **Achado durante esta verificação** (não estava listado no Step 5, mas apareceu ao
ler o arquivo linha a linha): antes desta sessão, a linha "Design e experiência" da "Checklist de
perguntas de esclarecimento" (`criar-prd/SKILL.md`) **não** era condicionada a `<ui_projeto>`:

```
- **Design e experiência**: diretrizes de UI/UX, tema claro e escuro, acessibilidade
```

Isso significa que, lida ao pé da letra, a checklist instruiria a IA a perguntar sobre "tema claro
e escuro" mesmo para uma API sem nenhuma interface — uma instrução sem sentido para o perfil deste
Step 5. O mesmo problema existia na seção "Experiência do usuário" de
`criar-prd/references/TEMPLATE.md`: ao contrário das seções "Plataforma alvo" e "Referências de
design" (que já tratavam explicitamente o caso sem UI), "Experiência do usuário" pedia
incondicionalmente "personas", "fluxos", "UI/UX incluindo tema claro e escuro" e "estados de
carregamento/vazio/erro de cada tela" — nenhuma ressalva para quando não há tela nenhuma.
**Corrigido nesta sessão** (arquivo incluído neste commit):

- `criar-prd/SKILL.md`: a linha virou `- **Design e experiência** (quando `<ui_projeto>` indicar
  interface de usuário final): diretrizes de UI/UX, tema claro e escuro, acessibilidade`.
- `criar-prd/references/TEMPLATE.md`: a seção "Experiência do usuário" ganhou a instrução de
  abertura `[Preencha esta seção apenas se `<ui_projeto>` indicar que o projeto tem interface de
  usuário final. Se não tiver, escreva "não aplicável — ver 'Plataforma alvo' para os consumidores
  e contratos" e siga para a próxima seção. ...]`.

Com a correção, o fluxo para o perfil Python passa a pular essa pergunta e essa seção de forma
explícita, em vez de depender do bom senso do executor para perceber que "tema claro e escuro" não
se aplica a uma API.

**`criar-tasks`** — a única condicional explícita de UI em todo o arquivo é sobre teste visual, não
sobre acessibilidade/i18n especificamente:

> `criar-tasks/SKILL.md`: *"SE `<ui_projeto>` INDICAR FERRAMENTA DE VALIDAÇÃO VISUAL, TODA TAREFA
> QUE CRIA UMA TELA NOVA PRECISA DA SUBTAREFA DE TESTE VISUAL **ANTES** DA SUBTAREFA QUE IMPLEMENTA
> A TELA"*

Para o perfil sem UI essa condição nunca dispara, então nenhuma subtarefa de teste visual é gerada
— correto. Mas não existe, em `criar-tasks`, nenhuma menção literal a "acessibilidade" nem a
"i18n" (confirmado por busca no arquivo inteiro). Isso não produz instrução errada para o perfil
Python (nada é gerado sobre acessibilidade porque nada nunca foi gerado), mas é uma lacuna real:
para um projeto **com** UI, `criar-tasks` não tem um gatilho explícito próprio que garanta
subtarefa de teste de acessibilidade — esse conteúdo só chega às tarefas indiretamente, via "Extrair
**todos** os casos de teste da seção 'Abordagem de testes' ... da <techspec>", ou seja, depende de
`criar-techspec` ter gerado casos de teste de acessibilidade primeiro. Achado registrado abaixo.

**`executar-qa`** — o desligamento de validação visual e de acessibilidade tem trecho condicional
explícito e literal:

> `executar-qa/SKILL.md`, seção 4: *"**Produto sem interface** (API, serviço, CLI): não há
> validação visual — pule esta etapa."*
>
> `executar-qa/SKILL.md`, seção 7: *"### 7. Verificação de Acessibilidade (quando `<ui_projeto>`
> indicar que é aplicável)"* ... *"Se `<ui_projeto>` não indicar acessibilidade aplicável, pule
> esta etapa."*

Ambos citam `<ui_projeto>` diretamente e ambos desligam corretamente para o perfil sem UI.
**i18n, porém, não tem nenhuma verificação própria em `executar-qa`** — o arquivo inteiro não
menciona "i18n" nenhuma vez (confirmado por busca). A única aparição de i18n em qualquer skill do
fluxo de execução é uma linha de checklist em `executar-review` (ver achado abaixo). Ou seja, o
"desligamento" de i18n em `executar-qa` para o perfil Python é vacuamente verdadeiro — não há nada
para desligar porque nunca havia sido ligado.

**`executar-bugfix`** — o desligamento de validação visual (que cobriria bugs visuais e, por
extensão, bugs de acessibilidade reportados pelo QA) está em:

> `executar-bugfix/SKILL.md`, seção 4: *"### 4. Validação de bugs visuais (Obrigatório para bugs
> de UI, quando `<ui_projeto>` indicar validação visual)"* ... *"Se `<ui_projeto>` não indicar
> ferramenta de validação visual, pule esta etapa."*

Para o perfil sem UI essa seção inteira é pulada. Não há, à parte disso, nenhuma menção literal a
"acessibilidade" nem a "i18n" em `executar-bugfix` (confirmado por busca) — mas isso é consistente
e não é um defeito: como `executar-qa` já não gera bugs de acessibilidade para um projeto sem UI
(seção 7 pulada), não há bug desse tipo em `bugs.md` para `executar-bugfix` processar. O mecanismo
de correção de bugs em si é genérico (diagnosticar causa raiz por camada, corrigir, testar) e não
precisaria de um branch dedicado a "bug de acessibilidade" — ele trataria um bug desse tipo como
qualquer outro bug de UI, coberto pela seção 4 quando ela se aplica.

**Achado (Step 5, i18n):** o campo `<ui_projeto>` → `i18n aplicável` existe no
`configurar-sdd/references/CONFIG_TEMPLATE.md` (linha 40) e é mencionado no fluxo de exploração de
`configurar-sdd/SKILL.md` ("configuração de i18n"), mas **nenhuma das quatro skills citadas no
Step 5 (`criar-prd`, `criar-tasks`, `executar-qa`, `executar-bugfix`) tem um trecho condicional
próprio que leia esse campo**. A única leitura de i18n em toda a parte de execução do fluxo é uma
linha combinada em `executar-review/SKILL.md`: *"- [ ] Acessibilidade e i18n verificadas, quando
`<ui_projeto>` indicar que são aplicáveis"* (seção 8, dentro do checklist de qualidade — não é uma
etapa numerada com passo a passo como a de acessibilidade em `executar-qa`). Isso não chega a ser
um "achado grave" — não há nenhum caso em que o bundle produza uma instrução de i18n sem sentido
para um projeto sem UI, porque i18n nunca é mencionado ativamente nessas quatro skills, com ou sem
UI. Mas é uma lacuna de cobertura real: se o time configurar `i18n aplicável: sim` para um projeto
com UI, não há nenhum passo dedicado em `criar-prd` (perguntas), `criar-tasks` (subtarefa
dedicada) ou `executar-qa` (verificação passo a passo, ao contrário de acessibilidade que tem seção
própria) que efetivamente cobre isso — só a linha de checklist combinada em `executar-review`.
**Optei por não expandir o escopo desta tarefa de verificação para adicionar novas seções de i18n
em quatro skills diferentes** (isso mudaria o comportamento e o conteúdo das skills além de uma
"correção pontual", e o Task 13 brief define `Files: Modify: nenhum` como padrão, reservando
correção só para o que estiver quebrado). Estou registrando a lacuna aqui para que uma tarefa
futura decida se vale a pena adicionar um branch de i18n explícito nessas skills.

**Achado (Step 5, geral):** nenhum bloqueio e nenhuma instrução visivelmente sem sentido restante
para o perfil Python após a correção da seção "Design e experiência"/"Experiência do usuário" em
`criar-prd` (ver acima). O fluxo é plenamente utilizável para uma API sem interface.

---

## Step 6 — Checklist de paridade com a versão Flutter original

Cada regra abaixo é citada com arquivo e trecho exato do bundle atual.

- [x] **Perguntas de esclarecimento obrigatórias antes do PRD**
  `criar-prd/SKILL.md`: `<critical>NÃO GERAR O PRD SEM ANTES FAZER PERGUNTAS DE ESCLARECIMENTO
  (USE A SUA FERRAMENTA NATIVA PARA PERGUNTAR AO USUÁRIO)</critical>` (aparece no topo e no
  rodapé do arquivo).

- [x] **Não desviar do template**
  `criar-prd/SKILL.md`: `<critical>EM HIPÓTESE ALGUMA DESVIAR DO <template> PRD</critical>`.
  `criar-techspec/SKILL.md`: `<critical>EM HIPÓTESE ALGUMA DESVIE DO PADRÃO DO <template></critical>`.

- [x] **PRD não contém implementação**
  `criar-prd/SKILL.md`: `<critical>NÃO INCLUA IMPLEMENTAÇÃO NO PRD — o COMO vive na
  TechSpec</critical>` e, nos Objetivos/Princípios: *"Foque no O QUÊ e no POR QUÊ, não no COMO"* /
  *"O PRD define resultados e restrições, **não implementação**"*.

- [x] **Explorar o projeto antes de perguntar (TechSpec)**
  `criar-techspec/SKILL.md`: `<critical>EXPLORE O PROJETO PRIMEIRO, ANTES DE FAZER PERGUNTAS DE
  ESCLARECIMENTO</critical>` (repetido no topo e no rodapé).

- [x] **Contrato de API confirmado no código, não suposto**
  `criar-techspec/SKILL.md`: `<critical>CONSULTE O CÓDIGO REAL DO SERVIÇO CONSUMIDO PARA DESCOBRIR
  O CONTRATO DA API ANTES DE ESPECIFICAR QUALQUER CLIENTE. A VERDADE ESTÁ NO CÓDIGO, NÃO NA
  SUPOSIÇÃO. Respeite as áreas read-only de `<limites_projeto>`: consultar sim, editar
  não</critical>`.

- [x] **Lista de tarefas aprovada antes de gerar arquivos**
  `criar-tasks/SKILL.md`: `<critical>**ANTES DE GERAR QUALQUER ARQUIVO, MOSTRE A LISTA DE TAREFAS
  DE ALTO NÍVEL PARA APROVAÇÃO**</critical>`.

- [x] **TDD: subtarefa de teste antes da subtarefa de implementação**
  `criar-tasks/SKILL.md`: `<critical>TDD É OBRIGATÓRIO: EM TODA TAREFA A SUBTAREFA DO TESTE VEM
  **ANTES** DA SUBTAREFA DA IMPLEMENTAÇÃO</critical>`.
  `executar-task/SKILL.md`: `<critical>**TDD É OBRIGATÓRIO**: teste falhando primeiro → confirmar
  o motivo da falha → código mínimo → refactor. Nenhum código de produção sem teste falhando
  antes. Exceções: ajuste de UI, código gerado, config</critical>`.

- [x] **Teste visual antes da tela (quando aplicável)**
  `criar-tasks/SKILL.md`: `<critical>SE `<ui_projeto>` INDICAR FERRAMENTA DE VALIDAÇÃO VISUAL, TODA
  TAREFA QUE CRIA UMA TELA NOVA PRECISA DA SUBTAREFA DE TESTE VISUAL **ANTES** DA SUBTAREFA QUE
  IMPLEMENTA A TELA</critical>`.
  `executar-task/SKILL.md`: `<critical>Se `<ui_projeto>` indicar ferramenta de validação visual,
  toda tela nova exige teste visual escrito **antes** da implementação da tela</critical>`.

- [x] **Documentação como última tarefa**
  `criar-tasks/SKILL.md`: `<critical>A ÚLTIMA TAREFA DA LISTA É SEMPRE A DOCUMENTAÇÃO, CONFORME
  `<documentacao_projeto>` DO CONFIG. SE O CONFIG INDICAR REGISTRO EXTERNO OBRIGATÓRIO, A TAREFA
  COBRE OS DOIS DESTINOS</critical>`, e na seção "Ordem de sequenciamento", item 6:
  *"Documentação — conforme `<documentacao_projeto>`"*.

- [x] **Causa raiz, nunca sintoma**
  `executar-bugfix/SKILL.md`: `<critical>Corrija a CAUSA RAIZ, nunca o sintoma. Se a correção é um
  bloco de tratamento de erro engolindo a exceção ou uma verificação defensiva na camada de
  apresentação, provavelmente está na camada errada</critical>`.

- [x] **Teste de regressão falhando antes da correção**
  `executar-bugfix/SKILL.md`: `<critical>TDD também vale para bugfix: escreva PRIMEIRO o teste de
  regressão que **falha reproduzindo o bug**, confirme que ele falha pelo motivo certo, e só então
  corrija</critical>`.

- [x] **Não regenerar artefato visual sem aprovação**
  `executar-bugfix/SKILL.md`: `<critical>NUNCA regenere artefatos de teste visual para "resolver"
  uma falha sem aprovação explícita do usuário — isso apaga a evidência da regressão</critical>`.
  `executar-qa/SKILL.md`, seção 4: *"**Nunca** regenere ou aceite artefato de teste visual como
  'correção' de uma falha sem aprovação explícita do usuário — isso apaga a evidência da
  regressão."*

- [x] **Nunca baixar o threshold de cobertura**
  `executar-task/SKILL.md`: *"Se a cobertura reprovar, **escreva mais teste** — nunca baixe o
  threshold"*.
  `executar-qa/SKILL.md`: `<critical>NUNCA baixe o threshold de cobertura para "passar". Falta de
  cobertura é achado do QA</critical>`.
  `executar-review/SKILL.md`: `<critical>NUNCA aprove uma redução do threshold de cobertura como
  forma de "passar" o gate</critical>`.
  `_shared/QUALITY_BASELINE.md`: *"O threshold de cobertura de `<comandos_projeto>` é tratado como
  absoluto: se faltar cobertura, escreve-se mais teste; **nunca** se baixa o threshold"*.

- [x] **Pedir confirmação antes de escrever em ambiente compartilhado**
  `executar-qa/SKILL.md`: `<critical>Antes de rodar qualquer teste que ESCREVA dados em ambiente
  compartilhado, PEÇA CONFIRMAÇÃO explícita ao usuário</critical>` (repetida na seção 5, E2E).
  `executar-task/SKILL.md`, seção 6: *"**Peça confirmação antes** de qualquer execução que escreva
  dados em ambiente compartilhado"*.
  `executar-bugfix/SKILL.md`, seção 5: *"pedindo **confirmação antes** de qualquer execução que
  escreva dados em ambiente compartilhado"*.

- [x] **Área read-only nunca alterada**
  `executar-task/SKILL.md`: `<critical>Nunca altere área listada como read-only em
  `<limites_projeto>`</critical>`.
  `criar-tasks/SKILL.md`: *"Nenhuma tarefa pode alterar área listada em `<limites_projeto>` como
  read-only; se a dependência não existir lá, a tarefa vira 'confirmar contrato com o time
  responsável' e é sinalizada como bloqueio"*.
  `executar-review/SKILL.md`: `<critical>Qualquer alteração em área read-only de
  `<limites_projeto>` REPROVA o review</critical>`.
  `executar-bugfix/SKILL.md`: *"Se a causa raiz estiver em área listada como read-only em
  `<limites_projeto>`, **não corrija**: documente o achado e sinalize ao time responsável"*.

- [x] **Gate verde obrigatório para fechar tarefa, bugfix, QA e review**
  `executar-task/SKILL.md`: `<critical>A tarefa NÃO está completa com gate vermelho: os comandos de
  `<comandos_projeto>` (lint, format, testes, cobertura) precisam passar, e a cobertura precisa
  atingir o threshold declarado</critical>`.
  `executar-bugfix/SKILL.md`: `<critical>O bugfix NÃO está completo enquanto os comandos de
  `<comandos_projeto>` não passarem, incluindo o threshold de cobertura</critical>`.
  `executar-qa/SKILL.md`: `<critical>O QA está REPROVADO se o gate de `<comandos_projeto>` falhar —
  lint, format e o threshold de cobertura são bloqueantes</critical>`.
  `executar-review/SKILL.md`: `<critical>O REVIEW NÃO ESTÁ COMPLETO ATÉ QUE O GATE DE
  `<comandos_projeto>` PASSE INTEIRO</critical>`.

- [x] **Qualidade dos testes avaliada além da cobertura**
  `executar-review/SKILL.md`, seção 7: *"Cobertura alta com teste ruim não vale nada. Confira a
  implementação contra `../_shared/QUALITY_BASELINE.md`, seção 'Qualidade dos testes'..."*.
  `executar-task/SKILL.md`: `<critical>Antes de escrever ou refatorar qualquer teste, releia a
  seção "Qualidade dos testes" de `../_shared/QUALITY_BASELINE.md`</critical>`.
  `_shared/QUALITY_BASELINE.md`, seção "Qualidade dos testes": lista de 6 verificações (testes de
  comportamento observável, caminho de falha pareado, sem asserção fraca, nome descritivo, mock na
  fronteira certa, threshold absoluto).

**Resultado do Step 6: 17 de 17 regras têm equivalente genérico ou condicional localizado e
citado no bundle atual.** Nenhuma regra foi perdida na generalização. A generalização, em quase
todos os casos, foi literal (a regra ficou igual, só passou a apontar para `<comandos_projeto>` /
`<limites_projeto>` / `<ui_projeto>` do config em vez de um comando ou caminho fixo do projeto
Flutter original).

---

## Achados consolidados

1. **(Corrigido nesta sessão) `criar-prd` perguntava/pedia UI/UX incondicionalmente.** A linha
   "Design e experiência" da checklist de esclarecimento e a seção "Experiência do usuário" do
   template de PRD não estavam condicionadas a `<ui_projeto>`, ao contrário de "Plataforma alvo" e
   "Referências de design" no mesmo arquivo, que já tratavam o caso sem UI. Isso produziria uma
   pergunta/instrução sem sentido ("tema claro e escuro") para um projeto como a API Python do
   Step 5. Corrigido em `criar-prd/SKILL.md` e `criar-prd/references/TEMPLATE.md` (incluído neste
   commit).

2. **(Não corrigido — registrado para decisão futura) i18n é um campo do config sem consumidor
   dedicado no fluxo.** `<ui_projeto>` → `i18n aplicável` existe no `CONFIG_TEMPLATE.md`, mas
   nenhuma das quatro skills verificadas no Step 5 (`criar-prd`, `criar-tasks`, `executar-qa`,
   `executar-bugfix`) tem uma etapa própria que o leia — só existe uma linha combinada
   ("Acessibilidade e i18n") no checklist de `executar-review`. Não gera instrução incorreta (o
   campo simplesmente não é usado, com ou sem UI), mas é uma lacuna de cobertura real que uma
   iteração futura deveria fechar, adicionando passos dedicados de i18n onde acessibilidade já os
   tem.

3. **(Observação, sem impacto) cobertura do sweep do Step 1.** O comando literal do Step 1 não
   inclui `configurar-sdd/references/`. Confirmei manualmente que esse caminho está limpo, mas o
   comando do brief, se copiado ao pé da letra em execuções futuras, deixaria esse arquivo de fora.

4. **Nenhum ponto do fluxo travaria ou exigiria conhecimento de Flutter** nos dois perfis
   simulados, depois da correção do item 1. Os dois perfis (React web e API Python sem UI)
   completam o fluxo `configurar-sdd → criar-prd → criar-techspec → criar-tasks → executar-task →
   executar-qa → executar-bugfix → executar-review` de ponta a ponta sem nenhuma instrução
   inaplicável remanescente.
