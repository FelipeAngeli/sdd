# SDD Global Template — Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Transformar o bundle SDD (hoje hard-coded para um app Flutter específico) num template portátil que se adapta sozinho a qualquer projeto, via uma skill de configuração que explora o repositório, pergunta o que falta e grava um único arquivo de configuração.

**Architecture:** Três camadas. As 8 skills (`SKILL.md`) contêm apenas processo, sem nenhuma menção a stack. As baselines em `_shared/` contêm checklists universais de segurança e qualidade. O arquivo `sdd.config.md`, na raiz do bundle, contém os únicos dados específicos do projeto e é gerado pela skill `configurar-sdd`. Cada skill referencia o config como `../sdd.config.md`, caminho que funciona tanto com o bundle na raiz do projeto quanto copiado para `.claude/skills/`.

**Tech Stack:** Markdown (convenção `SKILL.md` com frontmatter YAML do Claude Code), `grep`/shell para verificação, `git` para versionamento do bundle.

**Spec:** `docs/superpowers/specs/2026-09-12-sdd-global-template-design.md`

## Global Constraints

Estas regras valem para **todas** as tarefas. Cada tarefa as inclui implicitamente.

1. **Idioma:** todo o conteúdo é escrito em português (PT-BR). Termos técnicos consagrados (commit, branch, lint, coverage) permanecem em inglês.
2. **Nenhum arquivo pode assumir stack.** Nenhuma linguagem, framework, ferramenta de teste, comando ou caminho de pasta específico pode aparecer como fato. Só pode aparecer como *exemplo explicitamente marcado* (linha contendo `ex.:`, `ex:` ou `por exemplo`) ou dentro do `CONFIG_TEMPLATE.md`, que é um esqueleto de valores.
3. **Caminho do config:** sempre `../sdd.config.md` (relativo à pasta da skill). Nunca `.claude/sdd.config.md`, nunca caminho absoluto.
4. **Caminho das baselines:** sempre `../_shared/SECURITY_BASELINE.md` e `../_shared/QUALITY_BASELINE.md`.
5. **Bloco de cabeçalho padrão** — todas as 7 skills generalizadas recebem este bloco logo após o frontmatter, substituindo o `<contexto_projeto>` atual (texto exato, copie verbatim):

```md
<config>`../sdd.config.md`</config>
<baseline_seguranca>`../_shared/SECURITY_BASELINE.md`</baseline_seguranca>
<baseline_qualidade>`../_shared/QUALITY_BASELINE.md`</baseline_qualidade>

<contexto_projeto>
Leia `../sdd.config.md` ANTES de qualquer ação. Ele define a stack, a arquitetura, os comandos,
as integrações, os limites e as convenções deste projeto. Esta skill não assume linguagem,
framework nem ferramenta: tudo vem do config.
</contexto_projeto>

<critical>Se `../sdd.config.md` não existir, PARE e peça ao usuário para rodar a skill `configurar-sdd` primeiro. Nunca invente stack, comandos ou convenções.</critical>
```

6. **Padrão de condicional de integração** — toda menção a rastreador de issues, ferramenta de design ou base de conhecimento externa é escrita como condicional, nunca como fato (texto exato do padrão):

```md
Se `<integracoes_projeto>` do config indicar um rastreador de issues ativo (ex.: Linear, Jira,
GitHub Issues), [ação]. Caso contrário, pule esta etapa.
```

7. **Padrão de comando** — nunca cite um comando literal. Use: `o comando definido em <comandos_projeto>` (`Lint`, `Format`, `Testes`, `Cobertura`, `E2E`, `Build`, `Gate agregado`).
8. **Padrão de cobertura** — nunca cite um percentual. Use: `o threshold definido em <comandos_projeto>`. A regra "nunca baixe o threshold para passar" permanece, pois é universal.
9. **Termos proibidos** (sweep de verificação, roda na raiz do bundle):

```bash
grep -rniE 'flutter|\bdart\b|pubspec|\bbloc\b|cubit|modular|dartz|mocktail|nestjs|golden|talkback|voiceover|cy-loop|gpt-5|LIG-[0-9]|tool/coverage|make (ci|coverage|analyze|test|format)' \
  criar-* executar-* configurar-sdd/SKILL.md _shared/ README.md 2>/dev/null \
  | grep -viE 'ex\.?:|por exemplo|exemplo:'
```

Saída esperada em todas as tarefas: **vazia**. (`configurar-sdd/references/CONFIG_TEMPLATE.md` fica fora do sweep de propósito: ele contém valores de exemplo.)

10. **Commits:** Conventional Commits, em português no corpo. Um commit por tarefa.
11. **O bundle original não é reescrito por uso.** `configurar-sdd` grava o config e, opcionalmente, copia as skills para `.claude/skills/`; nunca edita o texto das skills.

---

### Task 1: Inicializar versionamento e criar a baseline de segurança

**Files:**
- Create: `.git/` (via `git init`)
- Create: `_shared/SECURITY_BASELINE.md`
- Create: `.gitignore`

**Interfaces:**
- Consumes: nada (primeira tarefa)
- Produces: `_shared/SECURITY_BASELINE.md` — referenciado por `criar-techspec`, `executar-task`, `executar-bugfix`, `executar-qa` e `executar-review` como `../_shared/SECURITY_BASELINE.md`. As seções que as skills citam pelo nome são: "Segredos e credenciais", "Dependências vulneráveis", "Injeção e validação de entrada", "Autenticação e autorização", "Dados sensíveis", "Configuração segura", "Extensões do projeto".

- [ ] **Step 1: Inicializar o repositório git**

```bash
cd /Users/felipe/Desktop/sdd
git init
```

- [ ] **Step 2: Criar o `.gitignore`**

```
.DS_Store
sdd.config.md
```

> `sdd.config.md` é ignorado no bundle **mestre** porque é gerado por projeto. Dentro de um projeto que usa o SDD, ele deve ser versionado normalmente — isso é explicado no README (Task 12).

- [ ] **Step 3: Escrever `_shared/SECURITY_BASELINE.md`**

```md
# Baseline de segurança (universal)

Checklist obrigatório, válido para qualquer linguagem, framework ou plataforma. É carregado
pelas skills de TechSpec, execução de task, bugfix, QA e code review.

Regras específicas deste projeto ficam em `<seguranca_extensoes>` do `sdd.config.md` e **somam**
a este arquivo — nunca o substituem. Este arquivo não deve ser editado por projeto.

## Segredos e credenciais

- [ ] Nenhuma chave, token, senha, certificado ou connection string hardcoded no código-fonte
- [ ] Nenhum segredo versionado (inclusive em arquivos de exemplo, fixtures, testes e logs de CI)
- [ ] Segredos vêm de variável de ambiente ou cofre; o repositório contém apenas o arquivo de
      exemplo sem valores reais
- [ ] Nenhum segredo impresso em log, mensagem de erro ou resposta de API

## Dependências vulneráveis

- [ ] O scanner de vulnerabilidades da stack foi executado (o comando correto está em
      `<comandos_projeto>`; se não houver, use o scanner padrão do gerenciador de pacotes do
      projeto — ex.: `npm audit`, `pip-audit`, `govulncheck`, `cargo audit`, `bundler-audit`)
- [ ] Achados de severidade **alta** ou **crítica** são tratados como bloqueantes
- [ ] Nenhuma dependência nova foi adicionada sem justificativa registrada na TechSpec
- [ ] O lockfile está commitado e coerente com o manifesto

## Injeção e validação de entrada

- [ ] Toda entrada vinda de fora (usuário, API, arquivo, variável de ambiente, deep link) é
      validada antes do uso
- [ ] Consultas a banco são parametrizadas — nenhuma concatenação de string com entrada do usuário
- [ ] Nenhuma entrada do usuário chega a um interpretador de comando do sistema sem sanitização
- [ ] Saída é codificada para o contexto em que é renderizada (evita XSS e equivalentes)
- [ ] Caminhos de arquivo derivados de entrada são normalizados e restritos (evita path traversal)
- [ ] Requisições a URLs derivadas de entrada são restritas a destinos permitidos (evita SSRF)
- [ ] Desserialização de dados não confiáveis usa formato e tipos restritos

## Autenticação e autorização

- [ ] Verificação de permissão acontece no servidor/camada de confiança, nunca só no cliente
- [ ] Toda rota, endpoint ou operação sensível tem verificação explícita de quem pode executá-la
- [ ] Identificadores previsíveis não dão acesso a recurso de outro usuário (referência direta
      insegura a objeto)
- [ ] Sessão/token tem expiração e é invalidado no logout

## Dados sensíveis

- [ ] Dados pessoais, credenciais e tokens nunca aparecem em log em texto claro
- [ ] Dados sensíveis em repouso usam o armazenamento seguro da plataforma
- [ ] Tráfego sensível trafega criptografado
- [ ] Mensagens de erro expostas ao usuário não vazam detalhe interno (stack trace, query, caminho)

## Configuração segura

- [ ] Modo debug/verbose desligado fora de ambiente de desenvolvimento
- [ ] CORS e headers de segurança restritos ao necessário
- [ ] Permissões e escopos seguem o princípio do menor privilégio
- [ ] Nenhum endpoint de diagnóstico/administração exposto publicamente sem autenticação

## Extensões do projeto

- [ ] As regras de `<seguranca_extensoes>` do `sdd.config.md` foram lidas e verificadas
- [ ] Requisitos de conformidade declarados no config (ex.: proteção de dados pessoais, setor
      regulado) foram endereçados
```

- [ ] **Step 4: Verificar que a baseline não assume stack**

```bash
grep -niE 'flutter|\bdart\b|pubspec|\bbloc\b|cubit|modular|dartz|mocktail|nestjs|golden' \
  _shared/SECURITY_BASELINE.md | grep -viE 'ex\.?:|por exemplo'
```

Expected: saída vazia.

- [ ] **Step 5: Verificar que as seções esperadas existem**

```bash
grep -c '^## ' _shared/SECURITY_BASELINE.md
```

Expected: `7`

- [ ] **Step 6: Commit**

```bash
git add .gitignore _shared/SECURITY_BASELINE.md
git commit -m "feat: adiciona baseline universal de seguranca

Checklist agnostico de stack cobrindo segredos, dependencias vulneraveis,
injecao, authn/authz, dados sensiveis e configuracao segura."
```

---

### Task 2: Criar a baseline de qualidade

**Files:**
- Create: `_shared/QUALITY_BASELINE.md`

**Interfaces:**
- Consumes: nada
- Produces: `_shared/QUALITY_BASELINE.md` — referenciado como `../_shared/QUALITY_BASELINE.md`. Seções citadas pelas skills: "Estrutura e complexidade", "Reuso", "Nomenclatura", "Tratamento de erro", "Comentários e código morto", "Performance", "Qualidade dos testes", "Extensões do projeto".

- [ ] **Step 1: Escrever `_shared/QUALITY_BASELINE.md`**

```md
# Baseline de qualidade de código (universal)

Checklist obrigatório, válido para qualquer linguagem ou framework. É carregado pelas skills de
TechSpec, execução de task, bugfix, QA e code review.

Convenções específicas da linguagem ou do time ficam em `<qualidade_extensoes>` do
`sdd.config.md` e **somam** a este arquivo. Este arquivo não deve ser editado por projeto.

## Estrutura e complexidade

- [ ] Funções e métodos têm uma responsabilidade só e cabem na tela
- [ ] Aninhamento profundo foi achatado (retorno antecipado em vez de cascata de condicionais)
- [ ] Arquivo que cresceu demais foi dividido por responsabilidade, não por camada técnica
- [ ] Nenhuma função recebe mais parâmetros do que consegue justificar

## Reuso

- [ ] O que já existe no projeto foi reusado antes de criar algo novo
- [ ] Nenhuma duplicação evitável de lógica; regra de negócio vive em um lugar só
- [ ] Abstração nova só foi criada com uso real comprovado (nada especulativo)

## Nomenclatura

- [ ] Nomes descrevem intenção, não implementação
- [ ] Convenção da linguagem/projeto seguida de forma consistente (ver `<qualidade_extensoes>`
      e a configuração de lint do projeto)
- [ ] Nenhuma abreviação obscura nem nome genérico (`data`, `info`, `handle`, `temp`)

## Tratamento de erro

- [ ] Erros são tratados explicitamente, nunca engolidos em silêncio
- [ ] O tipo de erro carrega informação suficiente para o chamador decidir o que fazer
- [ ] Falhas esperadas (rede, validação, ausência de permissão) têm caminho definido na UI/API
- [ ] Nenhum `catch` genérico que esconde defeito de programação

## Comentários e código morto

- [ ] Comentário explica o porquê, não o que o código já diz
- [ ] Nenhum código comentado, nenhum TODO órfão sem dono ou referência
- [ ] A política de comentários do projeto (`<colaboracao_projeto>`) foi respeitada

## Performance

- [ ] Nenhum trabalho redundante em caminho quente (recálculo, requisição ou leitura repetida)
- [ ] Coleções grandes não são carregadas inteiras quando um recorte resolve
- [ ] Recursos (conexões, arquivos, listeners, timers) são liberados

## Qualidade dos testes

- [ ] Testes verificam comportamento observável, não detalhe de implementação
- [ ] Todo caminho feliz tem o caminho de falha correspondente testado
- [ ] Nenhuma asserção fraca isolada (só "não é nulo", só "não lançou exceção")
- [ ] Nome do teste descreve o comportamento esperado e a condição
- [ ] Mock aplicado na fronteira certa — a unidade sob teste continua sendo exercitada de verdade
- [ ] O threshold de cobertura de `<comandos_projeto>` é tratado como absoluto: se faltar
      cobertura, escreve-se mais teste; **nunca** se baixa o threshold

## Extensões do projeto

- [ ] As convenções de `<qualidade_extensoes>` do `sdd.config.md` foram lidas e verificadas
```

- [ ] **Step 2: Verificar que a baseline não assume stack**

```bash
grep -niE 'flutter|\bdart\b|pubspec|\bbloc\b|cubit|modular|dartz|mocktail|nestjs|golden' \
  _shared/QUALITY_BASELINE.md | grep -viE 'ex\.?:|por exemplo'
```

Expected: saída vazia.

- [ ] **Step 3: Verificar que as seções esperadas existem**

```bash
grep -c '^## ' _shared/QUALITY_BASELINE.md
```

Expected: `8`

- [ ] **Step 4: Commit**

```bash
git add _shared/QUALITY_BASELINE.md
git commit -m "feat: adiciona baseline universal de qualidade de codigo"
```

---

### Task 3: Criar o esqueleto do arquivo de configuração

**Files:**
- Create: `configurar-sdd/references/CONFIG_TEMPLATE.md`

**Interfaces:**
- Consumes: nada
- Produces: o esqueleto que `configurar-sdd` copia e preenche para gerar `sdd.config.md`. Os nomes de tag definidos aqui são contrato para as 7 skills: `<stack_projeto>`, `<comandos_projeto>`, `<integracoes_projeto>`, `<ui_projeto>`, `<limites_projeto>`, `<seguranca_extensoes>`, `<qualidade_extensoes>`, `<colaboracao_projeto>`, `<documentacao_projeto>`.

- [ ] **Step 1: Escrever `configurar-sdd/references/CONFIG_TEMPLATE.md`**

````md
# Configuração do SDD para este projeto

> Gerado pela skill `configurar-sdd`. Pode ser editado à mão a qualquer momento.
> Rodar `configurar-sdd` de novo atualiza seção por seção, preservando edições manuais nas
> seções não afetadas.

<stack_projeto>
Linguagem(ns): [ex.: TypeScript]
Framework principal: [ex.: React 18 + Vite]
Padrão arquitetural: [ex.: feature-first com hooks; ou "sem padrão formal"]
Gerenciador de pacotes: [ex.: pnpm]
Ordem de dependência entre camadas (usada para sequenciar tasks): [ex.: utils → hooks → componentes → páginas → rotas]
Framework de teste: [ex.: Vitest + Testing Library]
Biblioteca de mock/dublê: [ex.: vi.mock nativo do Vitest]
</stack_projeto>

<comandos_projeto>
Lint: [comando]
Format: [comando]
Testes: [comando]
Cobertura: [comando] — threshold: [ex.: 80% por linha; ou "sem gate definido"]
E2E: [comando; ou "não aplicável"]
Build: [comando]
Gate agregado (roda tudo que bloqueia entrega): [comando; ou "não existe — rodar lint, testes e cobertura em sequência"]
Scanner de vulnerabilidade de dependências: [ex.: pnpm audit]
</comandos_projeto>

<integracoes_projeto>
Rastreador de issues: [Linear | Jira | GitHub Issues | nenhum] — como acessar: [ex.: MCP do Linear; ou "não disponível"]
Ferramenta de design: [Figma | nenhuma] — como acessar: [ex.: MCP do Figma]
Base de conhecimento externa: [ex.: Obsidian, Notion; ou nenhuma]
Orquestrador/roteamento de modelo: [descreva se existir; ou "nenhum"]
</integracoes_projeto>

<ui_projeto>
Tem interface de usuário final? [sim | não]
Tipo: [web | mobile | desktop | CLI | não aplicável]
Ferramenta de validação visual: [ex.: Playwright screenshot; ex.: golden test; ou "não aplicável"]
Acessibilidade aplicável: [sim — padrão alvo (ex.: WCAG 2.1 AA) | não]
i18n aplicável: [sim — como as strings são gerenciadas | não]
</ui_projeto>

<limites_projeto>
Áreas read-only ou fora de escopo: [ex.: `vendor/` é gerado, não editar; ou "nenhuma"]
Regras invioláveis do projeto (de CLAUDE.md / AGENTS.md / rules.md / CONTRIBUTING.md): [liste, ou "nenhuma encontrada"]
Arquivos de regra lidos na configuração: [caminhos; ou "nenhum encontrado"]
</limites_projeto>

<seguranca_extensoes>
[Regras de segurança específicas deste projeto, que somam ao `_shared/SECURITY_BASELINE.md`.
Ex.: "token de sessão só em armazenamento seguro do sistema", "dados de usuário sujeitos a
proteção de dados pessoais". Se não houver, escreva "nenhuma além da baseline".]
</seguranca_extensoes>

<qualidade_extensoes>
[Convenções de estilo/nomenclatura específicas, que somam ao `_shared/QUALITY_BASELINE.md`.
Ex.: regras relevantes da configuração de lint detectada. Se não houver, escreva
"nenhuma além da baseline".]
</qualidade_extensoes>

<colaboracao_projeto>
Convenção de commit: [ex.: Conventional Commits; ou "livre"] — detectada em: [ex.: commitlint.config.js; ou "perguntado ao usuário"]
Convenção de branch: [ex.: feature/<slug>; ou "sem convenção formal"]
Processo de PR: [ex.: template obrigatório, 1 aprovação, squash merge; ou "sem processo formal"]
Política de comentários no código: [ex.: comentar só onde a intenção não é óbvia]
</colaboracao_projeto>

<documentacao_projeto>
Onde documentar cada feature: [ex.: `docs/<feature>/`; ou "não aplicável"]
Registro externo obrigatório: [ex.: página na base de conhecimento com link de volta; ou "não"]
Idioma da documentação: [ex.: português]
</documentacao_projeto>

<artefatos_sdd>
Pasta raiz dos artefatos por feature: [padrão: `tasks/prd-<nome-da-feature>/`]
Arquivos por feature: prd.md · techspec.md · tasks.md · <num>_task.md · bugs.md · bugfixes.md · qa.md · codereview.md · evidences/
</artefatos_sdd>
````

- [ ] **Step 2: Verificar que todas as tags de contrato existem**

```bash
for t in stack_projeto comandos_projeto integracoes_projeto ui_projeto limites_projeto \
         seguranca_extensoes qualidade_extensoes colaboracao_projeto documentacao_projeto \
         artefatos_sdd; do
  grep -q "<$t>" configurar-sdd/references/CONFIG_TEMPLATE.md && echo "OK $t" || echo "FALTA $t"
done
```

Expected: dez linhas `OK`, nenhuma `FALTA`.

- [ ] **Step 3: Commit**

```bash
git add configurar-sdd/references/CONFIG_TEMPLATE.md
git commit -m "feat: adiciona esqueleto do sdd.config.md"
```

---

### Task 4: Criar a skill `configurar-sdd`

**Files:**
- Create: `configurar-sdd/SKILL.md`

**Interfaces:**
- Consumes: `configurar-sdd/references/CONFIG_TEMPLATE.md` (Task 3)
- Produces: a skill que gera `sdd.config.md` na raiz do bundle. Todas as 7 skills generalizadas apontam o usuário para ela pelo nome exato `configurar-sdd`.

- [ ] **Step 1: Escrever o frontmatter e os parâmetros**

```md
---
name: configurar-sdd
description: Adapte este bundle SDD ao projeto atual. Explora o repositório (manifestos, estrutura, testes, CI, arquivos de regra), infere stack, arquitetura, comandos e convenções, pergunta apenas o que não dá para detectar, e grava o arquivo sdd.config.md que todas as outras skills do SDD leem. Use sempre que o usuário pedir para configurar, adaptar, inicializar ou ajustar o SDD a um projeto novo, ou quando outra skill do SDD parar por falta do sdd.config.md.
---

<template_config>`./references/CONFIG_TEMPLATE.md`</template_config>
<saida>`../sdd.config.md`</saida>
<caminho_bundle>a pasta que contém esta skill e as demais skills do SDD</caminho_bundle>
```

- [ ] **Step 2: Escrever a persona e as regras críticas**

```md
## Persona

Você é um engenheiro que chega em um projeto desconhecido e precisa entendê-lo bem o suficiente
para configurar um fluxo de desenvolvimento orientado a especificação. Você investiga antes de
perguntar, e pergunta antes de supor.

<critical>EXPLORE O PROJETO ANTES DE PERGUNTAR QUALQUER COISA. Perguntar o que está escrito no repositório é o erro que esta skill existe para evitar.</critical>
<critical>NUNCA invente um valor de configuração. Se não detectou e não perguntou, o campo fica explicitamente marcado como indefinido.</critical>
<critical>NÃO edite o texto das outras skills do SDD. A adaptação acontece inteira no `sdd.config.md`.</critical>
<critical>Se `../sdd.config.md` já existir, ATUALIZE seção por seção preservando o que o usuário editou à mão. Nunca sobrescreva o arquivo inteiro sem mostrar o diff e obter aprovação.</critical>
<critical>NÃO implemente nenhuma feature nem altere código do projeto. Esta skill só produz configuração.</critical>
```

- [ ] **Step 3: Escrever o fluxo de trabalho**

```md
## Fluxo de trabalho

### 1. Ler as regras existentes do projeto (obrigatório)

Procure e leia, na raiz e um nível abaixo: `CLAUDE.md`, `.claude/CLAUDE.md`, `AGENTS.md`,
`rules.md`, `.cursorrules`, `CONTRIBUTING.md`, `CODEOWNERS`.

O que sair daqui alimenta `<limites_projeto>` (regras invioláveis, áreas read-only) e pode
alimentar `<seguranca_extensoes>` e `<colaboracao_projeto>`. Registre quais arquivos foram
encontrados — a ausência deles também é informação.

### 2. Explorar o projeto (obrigatório, antes de perguntar)

Investigue, nesta ordem:

- **Manifesto de pacote** — identifique linguagem, framework e dependências principais
- **Scripts e automação** — scripts do manifesto, `Makefile`, `justfile`, workflows de CI:
  daqui saem os comandos reais de lint, format, teste, cobertura, build e o threshold já
  configurado
- **Estrutura de pastas** — infira o padrão arquitetural e a ordem de dependência entre camadas
- **Testes existentes** — framework de teste, biblioteca de mock, convenção de nomenclatura,
  níveis de teste que já existem
- **Configuração de lint/format** — convenções de estilo e nomenclatura em uso
- **Convenção de commit/PR** — `commitlint`, hooks de git, template de PR, histórico recente de
  mensagens de commit
- **Indícios de UI** — presença de componentes de interface, testes de snapshot/screenshot,
  configuração de i18n

Use busca paralela e leia só o necessário: o objetivo é classificar, não auditar.

### 3. Classificar confiança (obrigatório)

Para cada campo do <template_config>, marque internamente:

- **detectado** — há evidência direta no repositório (cite o arquivo)
- **incerto** — há indício, mas ambíguo
- **ausente** — não há evidência

Só os campos **incerto** e **ausente** viram pergunta.

### 4. Perguntar apenas os gaps (obrigatório)

Use a ferramenta nativa de perguntas, agrupando por tema e preferindo múltipla escolha. Para
cada pergunta, mostre o que você já detectou, para o usuário só corrigir se estiver errado.

Temas que normalmente não são detectáveis e quase sempre viram pergunta:

- Rastreador de issues em uso e como acessá-lo
- Ferramenta de design em uso e como acessá-la
- Base de conhecimento externa para documentação estratégica
- Requisitos de conformidade ou segurança específicos do domínio
- Threshold de cobertura, quando o projeto não define um (sugira um valor e peça confirmação)
- Convenção de commit/branch/PR, quando não houver `commitlint`, hook ou template
- Padrão arquitetural, quando a estrutura for ambígua

<critical>NÃO PERGUNTE o que você já detectou com confiança. Apresente como fato a confirmar, em bloco, no fim.</critical>

### 5. Gravar o `sdd.config.md` (obrigatório)

- Copie a estrutura do <template_config> e preencha cada campo
- Campo sem resposta fica como `[indefinido — rodar configurar-sdd novamente quando souber]`
- Salve em <saida> (raiz do bundle, ao lado das pastas de skill)
- Se o arquivo já existia, mostre o que muda antes de gravar

### 6. Disponibilizar as skills para chamada por linguagem natural (opcional)

Se o projeto usa Claude Code e as skills ainda não estão em `.claude/skills/`, ofereça copiar o
bundle para lá, para que as chamadas seguintes aconteçam sem repetir o caminho. O
`sdd.config.md` acompanha a cópia, na raiz dela.

Se o usuário recusar, ou a ferramenta não for o Claude Code, nada quebra: todas as skills
continuam funcionando quando o usuário aponta o caminho do arquivo e pede para segui-lo.

### 7. Relatar (obrigatório)

- O que foi **detectado** (com o arquivo que serviu de evidência) versus o que foi **perguntado**
- Campos que ficaram indefinidos
- Caminho do `sdd.config.md` gerado
- Se as skills foram copiadas para `.claude/skills/`
- Como acionar a primeira skill do fluxo (`criar-prd`) nos dois modos
```

- [ ] **Step 4: Escrever o checklist de qualidade**

```md
## Checklist de qualidade

- [ ] Arquivos de regra do projeto procurados e lidos (ou ausência registrada)
- [ ] Manifesto, scripts, CI, estrutura, testes e lint explorados antes de qualquer pergunta
- [ ] Cada campo classificado como detectado / incerto / ausente
- [ ] Perguntas feitas só para campos incertos ou ausentes
- [ ] Nenhum campo preenchido por suposição
- [ ] `sdd.config.md` gravado na raiz do bundle
- [ ] Config preexistente atualizado por seção, com diff aprovado
- [ ] Relatório final entregue com detectado vs. perguntado e caminho do arquivo

<critical>EXPLORE ANTES DE PERGUNTAR</critical>
<critical>NUNCA invente valor de configuração</critical>
<critical>NÃO edite as outras skills — a adaptação vive no config</critical>
```

- [ ] **Step 5: Verificar que a skill não assume stack**

```bash
grep -niE 'flutter|\bdart\b|pubspec|\bbloc\b|cubit|modular|dartz|mocktail|nestjs|golden|make (ci|coverage)' \
  configurar-sdd/SKILL.md | grep -viE 'ex\.?:|por exemplo'
```

Expected: saída vazia.

- [ ] **Step 6: Verificar o frontmatter e as referências de caminho**

```bash
head -4 configurar-sdd/SKILL.md
grep -c '\.\./sdd\.config\.md' configurar-sdd/SKILL.md
```

Expected: frontmatter com `name: configurar-sdd`; contagem ≥ 1.

- [ ] **Step 7: Commit**

```bash
git add configurar-sdd/SKILL.md
git commit -m "feat: adiciona skill configurar-sdd

Explora o projeto, classifica confianca por campo, pergunta so os gaps
e grava o sdd.config.md que as demais skills leem."
```

---

### Task 5: Generalizar `criar-prd`

**Files:**
- Modify: `criar-prd/SKILL.md`
- Modify: `criar-prd/references/TEMPLATE.md`

**Interfaces:**
- Consumes: `../sdd.config.md` (tags `<ui_projeto>`, `<integracoes_projeto>`, `<artefatos_sdd>`, `<documentacao_projeto>`)
- Produces: `tasks/prd-<nome-da-feature>/prd.md` — consumido por `criar-techspec` (Task 6), `criar-tasks` (Task 7) e pelas skills de execução

- [ ] **Step 1: Substituir o cabeçalho do `SKILL.md`**

Remova as linhas `<figma>`, `<linear>`, o bloco `<roteamento_codex>` inteiro e o bloco
`<contexto_projeto>` inteiro. Mantenha `<prompt_base>` e `<template>`. Insira, no lugar, o
**Bloco de cabeçalho padrão** (Global Constraint 5).

Atualize a `description` do frontmatter para:

```md
description: Crie um PRD para uma nova funcionalidade do projeto atual. O PRD segue um formato padrão e é detalhado o suficiente para que a implementação possa começar a partir dele. Adapta-se à stack, ao tipo de produto e às integrações definidas no sdd.config.md, e aceita referências de design e de issue como contexto quando o projeto as tiver. Use quando o usuário pedir para criar um PRD, especificar uma feature nova ou levantar requisitos.
```

- [ ] **Step 2: Tornar condicionais as etapas de integração**

Na etapa "Coletar referências de entrada", substitua os blocos fixos de Linear e Figma por:

```md
**Rastreador de issues** — se `<integracoes_projeto>` indicar um rastreador ativo (ex.: Linear,
Jira, GitHub Issues) e o usuário tiver informado um identificador de issue:

- Leia título, descrição, labels, estimativa e estado pela ferramenta indicada no config
- Leia os comentários para capturar decisões já discutidas
- Trate a issue como **contexto**, não como fonte única: ela raramente tem requisitos completos

**Ferramenta de design** — se `<integracoes_projeto>` indicar uma ferramenta de design ativa
(ex.: Figma) e houver referência de design para a feature:

- Mapeie as telas/frames envolvidos e entenda visualmente cada fluxo
- Registre os identificadores das telas na seção "Referências de design" do <template> — a
  TechSpec vai reusá-los

Se o config indicar "nenhum"/"nenhuma", pule a etapa correspondente sem avisar erro.

<critical>NUNCA escreva no rastreador de issues além de comentários. Não crie, não mova e não feche issues.</critical>
```

- [ ] **Step 3: Tornar condicionais as perguntas de plataforma**

Na etapa "Esclarecer", substitua a linha de comportamento mobile-específico por:

```md
- Comportamento específico da plataforma, conforme `<ui_projeto>`:
  - **mobile**: funciona offline? exige permissão do sistema? entra por deep link ou
    notificação? muda entre ambientes/variantes de build?
  - **web**: navegadores e tamanhos de tela alvo? precisa funcionar sem conexão? há requisito
    de SEO ou de carregamento inicial?
  - **desktop/CLI**: sistemas operacionais alvo? comportamento offline? instalação/atualização?
  - **sem UI (API, serviço, biblioteca)**: contrato público, versionamento, compatibilidade
    retroativa, limites de uso
```

- [ ] **Step 4: Substituir caminhos e checklist do `SKILL.md`**

- Troque `./tasks/prd-[nome-da-feature]/` pela referência a `<artefatos_sdd>` do config, mantendo
  `tasks/prd-<nome-da-feature>/prd.md` como padrão citado
- No checklist de qualidade, troque "Considerações mobile (permissões, offline, deep link, tema
  claro/escuro)" por "Considerações específicas da plataforma de `<ui_projeto>` endereçadas"
- Troque "Comentário postado na issue do Linear (se houver)" por "Comentário postado na issue do
  rastreador configurado (se houver)"
- Remova a menção a `mcp__linear-server__save_comment` e descreva a ação em prosa

- [ ] **Step 5: Generalizar o `references/TEMPLATE.md`**

Substitua a seção "Plataformas alvo" por:

```md
## Plataforma alvo

[Preencha conforme `<ui_projeto>` do `sdd.config.md`. Cubra apenas o que se aplica ao tipo
deste projeto:

- Plataformas/ambientes alvo e diferenças de comportamento entre eles
- Permissões, capacidades ou limites do ambiente, e o que acontece quando faltam
- Comportamento sem conectividade, se aplicável
- Formas de entrada além do fluxo principal (link direto, notificação, chamada externa)

Se o projeto não tem interface de usuário final, descreva aqui os consumidores do que está
sendo construído e os contratos que eles dependem.]
```

Substitua "Referências de design (Figma)" por:

```md
## Referências de design

[Preencha apenas se `<integracoes_projeto>` indicar uma ferramenta de design ativa. Uma linha
por tela:

| Tela | Referência | Observações |
| --- | --- | --- |
| [Nome da tela] | `[identificador ou link]` | [estados cobertos: vazio, carregando, erro] |

- Se o projeto tem temas (ex.: claro e escuro), sinalize se o design cobriu todos
- Se não houver design, declare "sem referência de design" e trate como risco]
```

Na seção "Experiência do usuário", troque os requisitos de acessibilidade mobile por:

```md
- Requisitos de acessibilidade, se `<ui_projeto>` indicar acessibilidade aplicável: rótulos para
  leitor de tela, contraste, alvo de interação, suporte a aumento de escala de fonte, navegação
  por teclado quando aplicável
```

Na seção "Restrições técnicas de alto nível", troque a linha do backend por:

```md
- Serviços, APIs ou sistemas existentes que a funcionalidade consome, e quais deles estão fora
  do escopo de alteração conforme `<limites_projeto>`
```

Na seção "Rastreabilidade", troque por:

```md
[- Issue no rastreador configurado: `[identificador]` ou "sem issue vinculada"
- Referência de design: `[link]` ou "sem design"
- Documentação da feature a ser produzida: conforme `<documentacao_projeto>`]
```

- [ ] **Step 6: Rodar o sweep de termos proibidos**

```bash
grep -rniE 'flutter|\bdart\b|pubspec|\bbloc\b|cubit|modular|dartz|mocktail|nestjs|golden|talkback|voiceover|cy-loop|gpt-5|LIG-[0-9]|tool/coverage|make (ci|coverage|analyze)' \
  criar-prd/ | grep -viE 'ex\.?:|por exemplo|exemplo:'
```

Expected: saída vazia.

- [ ] **Step 7: Verificar que o bloco padrão entrou**

```bash
grep -c '\.\./sdd\.config\.md' criar-prd/SKILL.md
```

Expected: ≥ 2 (a tag `<config>` e a regra crítica).

- [ ] **Step 8: Commit**

```bash
git add criar-prd/
git commit -m "refactor: generaliza criar-prd para qualquer stack

Plataforma, design e rastreador de issues passam a ser condicionais
lidos do sdd.config.md."
```

---

### Task 6: Generalizar `criar-techspec`

**Files:**
- Modify: `criar-techspec/SKILL.md`
- Modify: `criar-techspec/references/TEMPLATE.md`

**Interfaces:**
- Consumes: `tasks/prd-<feature>/prd.md` (Task 5), `../sdd.config.md`, `../_shared/SECURITY_BASELINE.md`, `../_shared/QUALITY_BASELINE.md`
- Produces: `tasks/prd-<feature>/techspec.md` com a seção "Abordagem de testes", que é a entrada direta de `criar-tasks` (Task 7)

- [ ] **Step 1: Substituir o cabeçalho do `SKILL.md`**

Remova `<linear>`, o bloco `<roteamento_codex>`, o bloco `<contexto_projeto>` e o bloco
`<comandos>`. Mantenha `<prd>` e `<template>`. Insira o **Bloco de cabeçalho padrão** (Global
Constraint 5).

Atualize a `description`:

```md
description: Crie um TechSpec que traduz o PRD em decisões de arquitetura para a stack deste projeto, mapeia os contratos das APIs consumidas, as referências de design quando houver e a estratégia de testes com o threshold de cobertura definido no sdd.config.md. Use quando o usuário pedir para criar uma especificação técnica, detalhar como uma feature será implementada ou definir a arquitetura de uma funcionalidade.
```

- [ ] **Step 2: Generalizar as regras críticas**

Substitua as regras críticas específicas de Flutter por:

```md
<critical>EXPLORE O PROJETO PRIMEIRO, ANTES DE FAZER PERGUNTAS DE ESCLARECIMENTO</critical>
<critical>NÃO GERE A ESPECIFICAÇÃO SEM ANTES FAZER PERGUNTAS DE ESCLARECIMENTO (use sua ferramenta nativa de perguntas)</critical>
<critical>CONSULTE O CÓDIGO REAL DO SERVIÇO CONSUMIDO PARA DESCOBRIR O CONTRATO DA API ANTES DE ESPECIFICAR QUALQUER CLIENTE. A VERDADE ESTÁ NO CÓDIGO, NÃO NA SUPOSIÇÃO. Respeite as áreas read-only de `<limites_projeto>`: consultar sim, editar não</critical>
<critical>EM HIPÓTESE ALGUMA DESVIE DO PADRÃO DO <template></critical>
<critical>EM HIPÓTESE ALGUMA IMPLEMENTE O CÓDIGO; O OBJETIVO É PRODUZIR A ESPECIFICAÇÃO</critical>
<critical>A SEÇÃO "Abordagem de testes" PRECISA SER POPULADA COM O MÁXIMO DE CASOS DE TESTE POSSÍVEL PARA ATINGIR O THRESHOLD DE `<comandos_projeto>`, COBRINDO TODOS OS NÍVEIS DE TESTE QUE A STACK SUPORTA</critical>
<critical>Carregue `../_shared/SECURITY_BASELINE.md` e `../_shared/QUALITY_BASELINE.md` antes de escrever as seções de testes e de segurança</critical>
```

- [ ] **Step 3: Generalizar a etapa de análise exploratória**

Substitua a etapa 2 ("Análise profunda exploratória do projeto") por:

```md
### 2. Análise profunda exploratória do projeto (obrigatório)

Explore o projeto seguindo a estrutura e o padrão arquitetural declarados em `<stack_projeto>`:

- A área da feature, camada por camada, na ordem de dependência declarada no config
- O código compartilhado/núcleo do projeto — o que já existe e pode ser reusado: componentes,
  tema/estilos, logging, tratamento de erro, cliente HTTP, armazenamento
- Registro de dependências e rotas/pontos de entrada existentes
- Testes existentes: padrões vigentes, helpers reutilizáveis, fixtures
- Manifesto de pacote — o que já é dependência antes de propor qualquer pacote novo

Mapeie: quem chama quem, tratamento de erro, persistência, concorrência, navegação, testes.
```

- [ ] **Step 4: Generalizar a descoberta de contrato de API e de design**

Na etapa 3, substitua as referências a `backend/` e NestJS por:

```md
### 3. Descobrir o contrato real das APIs consumidas (obrigatório quando a feature consome APIs)

Se o contrato tem uma fonte de verdade acessível (código do serviço, especificação OpenAPI,
schema), consulte-a antes de especificar qualquer cliente:

- Localize as rotas da funcionalidade e leia os tipos de request/response e as validações
- Leia as regras de autenticação/autorização de cada rota
- Registre método, rota, payload, códigos de status e formato de erro no <template>

<critical>Se a fonte de verdade estiver em uma área listada em `<limites_projeto>` como read-only, apenas leia. Nunca edite, formate, rode ou builde nada nessa área.</critical>
```

Na etapa 4, substitua o bloco de Figma por uma condicional equivalente baseada em
`<integracoes_projeto>`, mantendo a regra de reusar token existente antes de criar token novo.

- [ ] **Step 5: Generalizar a tabela de camadas do `references/TEMPLATE.md`**

Substitua a seção "Mapeamento Clean Architecture" por:

```md
### Mapeamento por camada

<critical>
**OBRIGATÓRIO.** Toda funcionalidade precisa declarar em qual camada cada arquivo vive, usando
as camadas e a ordem de dependência declaradas em `<stack_projeto>` do `sdd.config.md`. As
regras de direção de dependência do projeto não podem ser violadas: nenhuma camada pode importar
de uma camada que dependa dela.
</critical>

| Camada | Arquivo | Responsabilidade | Novo ou modificado |
| --- | --- | --- | --- |
| [camada do config] | `[caminho real no projeto]` | [responsabilidade] | novo/modificado |

[Uma linha por arquivo. Se o projeto não tem camadas formais, agrupe por responsabilidade
(entrada, regra de negócio, acesso a dados, apresentação) e declare isso explicitamente.]
```

Substitua "Navegação e injeção de dependência" e "Reuso de `lib/core/`" por versões genéricas
que citam o padrão de injeção/registro e o código compartilhado detectados no config.

- [ ] **Step 6: Generalizar as seções de interfaces, estado e testes do template**

- "Principais interfaces": troque o exemplo em Dart por instrução de escrever os contratos **na
  linguagem do projeto**, com limite de 20 linhas por exemplo
- "Estados do BLoC/Cubit" → "Máquina de estados", mantendo a exigência de cobrir carregando,
  sucesso, vazio e erro, sem citar biblioteca
- "Abordagem de testes": troque as subseções fixas (unitário/BLoC/widget/golden/E2E) por:

```md
### Níveis de teste

[Uma subseção por nível de teste suportado pela stack, conforme `<stack_projeto>`. Para cada
nível, liste os cenários concretos — caminho feliz **e** caminho de falha.

Níveis comuns, use os que se aplicam:

- **Unitário** — regra de negócio isolada, sem dependência externa
- **Integração entre componentes** — a unidade com suas colaboradoras reais
- **Contrato/API** — request e response conferidos contra a fonte de verdade
- **Interface** — cada estado renderiza o que deve, incluindo vazio, carregando e erro
- **Visual** — apenas se `<ui_projeto>` indicar ferramenta de validação visual
- **Ponta a ponta** — apenas os fluxos principais do usuário

Para cada cenário de falha, especifique o erro concreto esperado, não uma exceção genérica.]

### Meta de cobertura

[Threshold do projeto: ver `<comandos_projeto>`. Declare a meta por área e como verificar.
Se o projeto não tem gate de cobertura, declare "sem gate" e liste mesmo assim os cenários
obrigatórios.]
```

- Substitua "Design e tokens (Figma)", "Documentação da feature" e "Conformidade com regras e
  skills" por versões condicionais que leem `<integracoes_projeto>`, `<documentacao_projeto>` e
  `<limites_projeto>`
- Na seção "Segurança", substitua o conteúdo por: "Verifique contra
  `../_shared/SECURITY_BASELINE.md` e registre aqui o que é específico desta feature, além das
  regras de `<seguranca_extensoes>`"

- [ ] **Step 7: Rodar o sweep de termos proibidos**

```bash
grep -rniE 'flutter|\bdart\b|pubspec|\bbloc\b|cubit|modular|dartz|dio\b|mocktail|nestjs|golden|talkback|voiceover|cy-loop|gpt-5|LIG-[0-9]|tool/coverage|make (ci|coverage|analyze)' \
  criar-techspec/ | grep -viE 'ex\.?:|por exemplo|exemplo:'
```

Expected: saída vazia.

- [ ] **Step 8: Verificar que as baselines são referenciadas**

```bash
grep -c '_shared/SECURITY_BASELINE.md\|_shared/QUALITY_BASELINE.md' criar-techspec/SKILL.md
```

Expected: ≥ 1.

- [ ] **Step 9: Commit**

```bash
git add criar-techspec/
git commit -m "refactor: generaliza criar-techspec para qualquer stack

Camadas, contratos de API, niveis de teste e cobertura passam a vir
do sdd.config.md; seguranca referencia a baseline compartilhada."
```

---

### Task 7: Generalizar `criar-tasks`

**Files:**
- Modify: `criar-tasks/SKILL.md`
- Modify: `criar-tasks/references/TASKS_TEMPLATE.md`
- Modify: `criar-tasks/references/TASK_TEMPLATE.md`

**Interfaces:**
- Consumes: `prd.md` (Task 5) e `techspec.md` (Task 6), `../sdd.config.md`
- Produces: `tasks/prd-<feature>/tasks.md` e `tasks/prd-<feature>/<num>_task.md` — consumidos por `executar-task` (Task 8), `executar-qa` (Task 10) e `executar-review` (Task 11)

- [ ] **Step 1: Substituir o cabeçalho do `SKILL.md`**

Remova `<linear>`, `<roteamento_codex>` e `<contexto_projeto>`. Mantenha `<prd>`, `<techspec>`,
`<tasks_template>` e `<task_template>`. Insira o **Bloco de cabeçalho padrão**.

- [ ] **Step 2: Generalizar as regras críticas**

Mantenha, sem alteração de sentido: mostrar a lista de alto nível para aprovação antes de gerar
arquivos; não implementar nada; cada tarefa é uma entrega bem definida; toda tarefa tem testes;
todos os casos de teste da TechSpec distribuídos; **TDD obrigatório — a subtarefa de teste vem
antes da subtarefa de implementação**.

Substitua as duas regras específicas de Flutter por:

```md
<critical>SE `<ui_projeto>` INDICAR FERRAMENTA DE VALIDAÇÃO VISUAL, TODA TAREFA QUE CRIA UMA TELA NOVA PRECISA DA SUBTAREFA DE TESTE VISUAL **ANTES** DA SUBTAREFA QUE IMPLEMENTA A TELA</critical>
<critical>A ÚLTIMA TAREFA DA LISTA É SEMPRE A DOCUMENTAÇÃO, CONFORME `<documentacao_projeto>` DO CONFIG. SE O CONFIG INDICAR REGISTRO EXTERNO OBRIGATÓRIO, A TAREFA COBRE OS DOIS DESTINOS</critical>
```

- [ ] **Step 3: Generalizar a ordem por camada**

Substitua a seção "Ordem por camada (Clean Architecture)" por:

```md
## Ordem de sequenciamento

Dependentes sempre depois das dependências. Use a ordem de dependência entre camadas declarada
em `<stack_projeto>` do config. O princípio, independente de nomenclatura:

1. **Regra de negócio pura** — o que não depende de nada externo
2. **Acesso a dados e integrações** — o que fala com o mundo de fora
3. **Apresentação/entrada** — o que expõe a funcionalidade ao consumidor
4. **Registro e ligação** — rotas, injeção de dependência, configuração
5. **Ponta a ponta** — só depois que o fluxo completo existe
6. **Documentação** — conforme `<documentacao_projeto>`

Se o projeto não tem camadas formais, agrupe por dependência real entre os arquivos.
```

- [ ] **Step 4: Generalizar as diretrizes e o checklist**

- Troque "Nenhuma tarefa pode tocar `backend/`" por "Nenhuma tarefa pode alterar área listada em
  `<limites_projeto>` como read-only; se a dependência não existir lá, a tarefa vira 'confirmar
  contrato com o time responsável' e é sinalizada como bloqueio"
- Troque "Presuma que o leitor principal é um desenvolvedor Flutter" por "Presuma que o leitor
  principal é um desenvolvedor com a stack de `<stack_projeto>`"
- Troque a linha do gate de 90% por "A soma dos testes das tarefas precisa sustentar o threshold
  de `<comandos_projeto>`"
- Ajuste o checklist de qualidade na mesma linha

- [ ] **Step 5: Reescrever `references/TASKS_TEMPLATE.md`**

Substitua o arquivo inteiro por:

```md
# Resumo das tarefas de implementação de [Funcionalidade]

> Ordem de sequenciamento: conforme a ordem de dependência entre camadas de `<stack_projeto>`.
> Gate de conclusão da feature: comandos de `<comandos_projeto>` verdes, cobertura no threshold
> declarado.

## Tarefas

- [ ] 1.0 Título da tarefa `[camada]`
- [ ] 2.0 Título da tarefa `[camada]`
- [ ] 3.0 Título da tarefa `[camada]`
- [ ] N.0 Documentação da feature (conforme `<documentacao_projeto>`)
```

- [ ] **Step 6: Reescrever `references/TASK_TEMPLATE.md`**

Substitua o arquivo inteiro por:

````md
# Tarefa X.0: [Título da tarefa]

## Visão geral

[Descrição breve da tarefa]

**Camada:** [uma das camadas declaradas em `<stack_projeto>`]

<regras>
### Regras e convenções aplicáveis

[Liste as regras do projeto que esta tarefa precisa respeitar: as de `<limites_projeto>`, as
convenções de `<qualidade_extensoes>` e de `<colaboracao_projeto>`, e as baselines
`../../_shared/SECURITY_BASELINE.md` e `../../_shared/QUALITY_BASELINE.md`. Carregue tudo antes
de implementar.]
</regras>

<requirements>
[Lista de requisitos obrigatórios (usar os RFs do PRD quando aplicável)]
</requirements>

## Subtarefas

<critical>A subtarefa de teste vem SEMPRE antes da subtarefa de implementação correspondente (TDD). Se `<ui_projeto>` indicar validação visual, o teste visual vem antes da tela.</critical>

- [ ] X.1 [Teste falhando que descreve o comportamento esperado]
- [ ] X.2 [Implementação mínima para o teste passar]
- [ ] X.3 [Refactor / próximo comportamento]

## Detalhes de implementação

[Seções pertinentes da especificação técnica. **NÃO REPITA A IMPLEMENTAÇÃO, APENAS REFERENCIE
`techspec.md`**]

## Critérios de sucesso

- [Resultados mensuráveis]
- [Requisitos de qualidade]
- [ ] Comandos de `<comandos_projeto>` (format, lint, testes) passam
- [ ] Cobertura não derruba o threshold declarado em `<comandos_projeto>`

## Testes da tarefa

<critical>Antes de escrever ou refatorar qualquer teste, leia `../../_shared/QUALITY_BASELINE.md`, seção "Qualidade dos testes". Use a biblioteca de mock declarada em `<stack_projeto>` e nenhuma outra. O nome do teste descreve o comportamento esperado e a condição.</critical>

[Uma subseção por nível de teste aplicável, conforme os níveis declarados em `<stack_projeto>`
e a estratégia definida na TechSpec. Use apenas os níveis que existem neste projeto.]

### [Nível de teste — ex.: unitário]

- [ ] ...
  <!-- Caminho feliz e caminho de falha. Asserção no valor concreto e no tipo de erro concreto. -->

### [Nível de teste — ex.: integração entre componentes]

- [ ] ...
  <!-- Mock só na fronteira externa; a unidade sob teste continua sendo exercitada de verdade. -->

### [Nível de teste — interface, se `<ui_projeto>` indicar UI]

- [ ] ...
  <!-- Cada estado renderiza o esperado: conteúdo, vazio, erro no lugar certo, carregando. -->

### [Validação visual — só se `<ui_projeto>` declarar ferramenta de validação visual]

- [ ] [Um item por variação exigida pelo projeto, ex.: cada tema suportado]
  <!-- Escrito ANTES da implementação da tela. Regenerar artefato visual só com aprovação
       explícita do usuário. -->

### [Ponta a ponta, se aplicável]

- [ ] ...
  <!-- Só os fluxos principais. Declare aqui se o teste ESCREVE dados em ambiente compartilhado
       — nesse caso, exige confirmação do usuário antes de rodar. -->

## Arquivos relevantes

**Implementação**

- [`caminho real no projeto, conforme a estrutura de <stack_projeto>`]

**Testes**

- [`caminho real dos testes`]

**Registro e ligação** (rotas, injeção de dependência, configuração)

- [`caminho real, se a tarefa tocar nesses pontos`]
````

- [ ] **Step 7: Rodar o sweep de termos proibidos**

```bash
grep -rniE 'flutter|\bdart\b|pubspec|\bbloc\b|cubit|modular|dartz|mocktail|nestjs|golden|cy-loop|gpt-5|LIG-[0-9]|make (ci|coverage|analyze)' \
  criar-tasks/ | grep -viE 'ex\.?:|por exemplo|exemplo:'
```

Expected: saída vazia.

- [ ] **Step 8: Verificar que TDD continua obrigatório**

```bash
grep -ci 'tdd' criar-tasks/SKILL.md
```

Expected: ≥ 2.

- [ ] **Step 9: Commit**

```bash
git add criar-tasks/
git commit -m "refactor: generaliza criar-tasks para qualquer stack

Sequenciamento por dependencia declarada no config; teste visual e
documentacao externa viram condicionais. TDD permanece obrigatorio."
```

---

### Task 8: Generalizar `executar-task`

**Files:**
- Modify: `executar-task/SKILL.md`

**Interfaces:**
- Consumes: `prd.md`, `techspec.md`, `tasks.md`, `<num>_task.md` (Tasks 5-7), `../sdd.config.md`, ambas as baselines
- Produces: código implementado e `tasks.md` atualizado — verificado por `executar-qa` (Task 10) e `executar-review` (Task 11)

- [ ] **Step 1: Substituir o cabeçalho**

Remova `<linear>`, `<roteamento_codex>`, `<contexto_projeto>` e `<comandos>`. Mantenha `<prd>`.
Insira o **Bloco de cabeçalho padrão**.

- [ ] **Step 2: Generalizar as regras críticas**

Mantenha universais: não pular subtarefa; TDD obrigatório (teste falhando primeiro → confirmar o
motivo da falha → código mínimo → refactor); iniciar a implementação logo após preparação e
análise; marcar a tarefa como completa ao final.

Substitua as regras de camada, rota e golden por:

```md
<critical>Respeite a direção de dependência entre camadas declarada em `<stack_projeto>`. Nenhuma camada importa de quem depende dela</critical>
<critical>Se `<ui_projeto>` indicar ferramenta de validação visual, toda tela nova exige teste visual escrito **antes** da implementação da tela</critical>
<critical>Nunca altere área listada como read-only em `<limites_projeto>`</critical>
<critical>A tarefa NÃO está completa com gate vermelho: os comandos de `<comandos_projeto>` (lint, format, testes, cobertura) precisam passar, e a cobertura precisa atingir o threshold declarado</critical>
<critical>Siga a convenção de commit de `<colaboracao_projeto>` ao registrar o trabalho</critical>
```

- [ ] **Step 3: Generalizar as etapas de ciclo TDD e verificação**

Na etapa "Implementação em ciclos TDD", substitua as regras específicas (mocktail, `bloc_test`,
`DioException`, `pumpWidget`, helper de golden) por:

```md
Regras que valem em todo ciclo:

- Use a biblioteca de mock declarada em `<stack_projeto>` e nenhuma outra
- Nome de teste descreve o comportamento esperado e a condição
- Asserção específica: verifique o valor concreto e o tipo de erro concreto, não apenas que
  "não é nulo" ou que "não lançou"
- Falha de rede/integração usa o erro real da biblioteca do projeto, com o status e o tipo
  corretos — nunca um erro genérico
- Mock na fronteira certa: o teste de um componente dubla suas dependências diretas, não ele mesmo
- Injeção por construtor/parâmetro no teste; sem bootstrap do contêiner de injeção em teste unitário
- Teste visual, quando aplicável, usa o helper e a superfície declarados em `<ui_projeto>`
```

Na etapa "Verificação", substitua o bloco de comandos `make` por:

```md
Rode, nesta ordem, os comandos declarados em `<comandos_projeto>`: format, lint, testes e
cobertura. Nenhum pode falhar.

- Se a cobertura reprovar, **escreva mais teste** — nunca baixe o threshold
- Se o projeto tem E2E e a tarefa o afeta, rode o comando de E2E. **Peça confirmação antes** de
  qualquer execução que escreva dados em ambiente compartilhado
```

- [ ] **Step 4: Generalizar o fechamento e o checklist**

- Troque a etapa de comentário no Linear por condicional de `<integracoes_projeto>`
- Troque a menção à documentação dupla por `<documentacao_projeto>`
- Ajuste o checklist de qualidade removendo golden/Modular/Flutter e citando as regras do config

- [ ] **Step 5: Rodar o sweep de termos proibidos**

```bash
grep -rniE 'flutter|\bdart\b|pubspec|\bbloc\b|cubit|modular|dartz|dio\b|mocktail|golden|cy-loop|gpt-5|LIG-[0-9]|make (ci|coverage|analyze|format|test)' \
  executar-task/ | grep -viE 'ex\.?:|por exemplo|exemplo:'
```

Expected: saída vazia.

- [ ] **Step 6: Verificar que o ciclo TDD continua explícito**

```bash
grep -ciE 'vermelho|falhando primeiro|teste.*antes' executar-task/SKILL.md
```

Expected: ≥ 2.

- [ ] **Step 7: Commit**

```bash
git add executar-task/
git commit -m "refactor: generaliza executar-task para qualquer stack

Comandos, camadas, mocks e teste visual passam a vir do config.
Ciclo TDD e gate verde permanecem obrigatorios."
```

---

### Task 9: Generalizar `executar-bugfix`

**Files:**
- Modify: `executar-bugfix/SKILL.md`
- Modify: `executar-bugfix/references/TEMPLATE.md`

**Interfaces:**
- Consumes: `bugs.md` (produzido por `executar-qa`, Task 10), `prd.md`, `techspec.md`, `../sdd.config.md`, ambas as baselines
- Produces: `bugfixes.md` e `bugs.md` atualizado com status de cada defeito

- [ ] **Step 1: Substituir o cabeçalho**

Remova `<linear>`, `<device>`, `<contexto_projeto>` e `<comandos>`. Mantenha `<prd>` e
`<template>`. Insira o **Bloco de cabeçalho padrão**.

- [ ] **Step 2: Generalizar as regras críticas**

Mantenha universais, sem alteração de sentido: TDD vale para bugfix (teste de regressão que
falha reproduzindo o bug primeiro); corrigir a causa raiz, nunca o sintoma.

Substitua as específicas por:

```md
<critical>Se `<ui_projeto>` indicar ferramenta de validação visual, bug visual se valida por teste visual mais verificação no ambiente real do produto — não por suposição</critical>
<critical>NUNCA regenere artefatos de teste visual para "resolver" uma falha sem aprovação explícita do usuário — isso apaga a evidência da regressão</critical>
<critical>Se a causa raiz estiver em área listada como read-only em `<limites_projeto>`, **não corrija**: documente o achado e sinalize ao time responsável</critical>
<critical>O bugfix NÃO está completo enquanto os comandos de `<comandos_projeto>` não passarem, incluindo o threshold de cobertura</critical>
```

- [ ] **Step 3: Generalizar o diagnóstico e a validação visual**

Na etapa "Diagnóstico por bug", substitua a lista de camadas Flutter por "identifique a camada da
causa raiz entre as camadas declaradas em `<stack_projeto>`".

Substitua a etapa "Validação de bugs visuais" por uma versão condicional que: usa a ferramenta de
`<ui_projeto>`, verifica os estados vazio/carregando/erro e os temas que o projeto tiver, captura
evidência na pasta de evidências de `<artefatos_sdd>`, e só regenera artefato visual após
aprovação.

- [ ] **Step 4: Generalizar as regras de escrita do teste de regressão**

Substitua o bloco "Regras de escrita" por:

```md
Regras de escrita do teste de regressão:

- Use a biblioteca de mock declarada em `<stack_projeto>` e nenhuma outra
- O nome do teste descreve o comportamento esperado e a condição que o dispara
- Falha de rede/integração usa o erro real da biblioteca do projeto, com o status e o tipo
  corretos — nunca um erro genérico
- Asserção específica: verifique o valor concreto e o tipo de erro concreto, não apenas que
  "não é nulo" ou que "não lançou"
- Mock na fronteira certa: o teste de um componente dubla suas dependências diretas, não ele mesmo
- O teste precisa **falhar se a correção for revertida** — esse é o critério de um bom teste de
  regressão
- Cubra as variações relacionadas do mesmo problema, não só o caso exato relatado
```

- [ ] **Step 5: Generalizar a verificação e o fechamento**

- Substitua o bloco de comandos `make` por: "Rode os comandos declarados em
  `<comandos_projeto>`: format, lint, testes e cobertura. Nenhum pode falhar, e a cobertura
  precisa atingir o threshold declarado."
- Substitua a linha de E2E por: "Se o bug tinha cobertura ponta a ponta, rode também o comando de
  E2E de `<comandos_projeto>`, pedindo **confirmação antes** de qualquer execução que escreva
  dados em ambiente compartilhado"
- Troque as menções ao Linear por condicional de `<integracoes_projeto>`, conforme o padrão da
  Global Constraint 6
- Ajuste o checklist de qualidade: troque "Nenhuma alteração em `backend/`" por "Nenhuma
  alteração em área read-only de `<limites_projeto>`"; troque as linhas de golden/`make` pelas
  equivalentes do config

- [ ] **Step 6: Reescrever `references/TEMPLATE.md`**

Substitua o arquivo inteiro por:

```md
# Relatório de Bugfix - [Nome da Funcionalidade]

## Resumo
- Data: [data]
- Branch: [branch]
- Total de Bugs: [X]
- Bugs Corrigidos: [Y]
- Bugs Não Corrigidos: [Z] (ver "Bloqueios")
- Testes de Regressão Criados: [N]
- Issue vinculada: [identificador no rastreador configurado ou "sem issue"]

## Detalhes por Bug
| ID | Severidade | Status | Camada da causa raiz | Correção | Testes de regressão criados |
|----|------------|--------|----------------------|----------|------------------------------|
| BUG-01 | Alta | Corrigido | [camada de `<stack_projeto>`] | [descrição] | [arquivos e nomes dos testes] |

## Verificação
[Uma linha por comando de `<comandos_projeto>` que se aplica.]

| Etapa | Comando | Resultado |
|-------|---------|-----------|
| Format | [comando do config] | OK / NOK |
| Lint | [comando do config] | [N] issues (esperado: 0) |
| Testes | [comando do config] | TODOS PASSANDO / [N] falhando |
| Cobertura | [comando do config] | [resultado] — threshold [X]% OK / FAIL |
| Validação visual | [comando do config, se aplicável] | TODOS PASSANDO / [N] falhando / N/A |
| Ponta a ponta | [comando do config, se aplicável] | PASSOU / FALHOU / NÃO EXECUTADO |

> Artefatos de validação visual regenerados nesta rodada? [não / sim — aprovado pelo usuário em [contexto]]

## Evidências visuais
[Preencher apenas se `<ui_projeto>` indicar validação visual.]

| BUG | Antes | Depois | Variações verificadas |
|-----|-------|--------|-----------------------|
| BUG-0X | [evidência] | [evidência] | [ex.: temas, tamanhos de tela] |

## Bloqueios
[Bugs cuja causa raiz está em área read-only de `<limites_projeto>`, ou que dependem de decisão
do usuário. Descreva o achado e o que é necessário para destravar.]

## Conclusão
[Parecer final]
```

- [ ] **Step 7: Rodar o sweep de termos proibidos**

```bash
grep -rniE 'flutter|\bdart\b|\bbloc\b|cubit|modular|dartz|dio\b|mocktail|golden|cy-loop|LIG-[0-9]|make (ci|coverage|analyze|test)' \
  executar-bugfix/ | grep -viE 'ex\.?:|por exemplo|exemplo:'
```

Expected: saída vazia.

- [ ] **Step 8: Verificar que a regra de causa raiz sobreviveu**

```bash
grep -ci 'causa raiz' executar-bugfix/SKILL.md
```

Expected: ≥ 3.

- [ ] **Step 9: Commit**

```bash
git add executar-bugfix/
git commit -m "refactor: generaliza executar-bugfix para qualquer stack

Camada da causa raiz, validacao visual e comandos vem do config.
Teste de regressao falhando primeiro permanece obrigatorio."
```

---

### Task 10: Generalizar `executar-qa`

**Files:**
- Modify: `executar-qa/SKILL.md`
- Modify: `executar-qa/references/TEMPLATE.md`

**Interfaces:**
- Consumes: `prd.md`, `techspec.md`, `tasks.md`, `../sdd.config.md`, ambas as baselines
- Produces: `qa.md` (relatório) e `bugs.md` (entrada de `executar-bugfix`, Task 9)

- [ ] **Step 1: Substituir o cabeçalho**

Remova `<linear>`, `<device>`, `<roteamento_codex>`, `<contexto_projeto>` e `<comandos>`.
Mantenha `<prd>` e `<template>`. Insira o **Bloco de cabeçalho padrão**.

- [ ] **Step 2: Generalizar as regras críticas**

```md
<critical>O QA só está APROVADO quando TODOS os requisitos do PRD forem verificados e estiverem funcionando</critical>
<critical>O QA está REPROVADO se o gate de `<comandos_projeto>` falhar — lint, format e o threshold de cobertura são bloqueantes</critical>
<critical>NUNCA baixe o threshold de cobertura para "passar". Falta de cobertura é achado do QA</critical>
<critical>Use o método de validação declarado em `<ui_projeto>`: automação de navegador quando o produto é web, execução em dispositivo/emulador quando é mobile, verificação de contrato e comportamento quando não há interface</critical>
<critical>Antes de rodar qualquer teste que ESCREVA dados em ambiente compartilhado, PEÇA CONFIRMAÇÃO explícita ao usuário</critical>
<critical>Verifique a implementação contra `../_shared/SECURITY_BASELINE.md` — achado de segurança é bug, com severidade proporcional ao risco</critical>
```

- [ ] **Step 3: Generalizar as etapas de execução**

- "Gate automatizado": substitua `make ci` pelos comandos de `<comandos_projeto>`, registrando o
  resultado de cada um e a linha final de cobertura
- "Golden tests": vira "Validação visual", condicional a `<ui_projeto>` — ausência de teste
  visual para tela nova continua sendo achado quando o projeto usa essa ferramenta
- "Integration tests / E2E em device": vira "Testes ponta a ponta", usando o comando de
  `<comandos_projeto>`; mantém as regras de pedir confirmação antes de escrever em ambiente
  compartilhado e de não reprovar quando o ambiente necessário não existe, registrando a
  pendência
- "Verificação funcional no app": vira "Verificação funcional", descrevendo como exercitar cada
  requisito conforme o tipo de produto
- "Verificação de Acessibilidade": condicional a `<ui_projeto>`, com checklist genérico (rótulo
  descritivo, leitor de tela, contraste, alvo de interação, escala de fonte, ordem de foco,
  navegação por teclado)
- "Verificações Visuais": condicional; comparação com design só se houver ferramenta de design
  no config
- Adicione uma etapa nova de verificação de segurança que roda o scanner de vulnerabilidades de
  `<comandos_projeto>` e confere a baseline

- [ ] **Step 4: Reescrever `references/TEMPLATE.md`**

Substitua o arquivo inteiro por:

```md
# Relatório de QA - [Nome da Funcionalidade]

## Resumo
- Data: [data]
- Branch: [branch]
- Status: APROVADO / APROVADO COM RESSALVAS / REPROVADO
- Total de Requisitos: [X]
- Requisitos Atendidos: [Y]
- Bugs Encontrados: [Z]
- Issue vinculada: [identificador no rastreador configurado ou "sem issue"]

## Gate automatizado
[Uma linha por comando de `<comandos_projeto>`.]

| Etapa | Comando | Resultado |
|-------|---------|-----------|
| Format | [comando do config] | OK / NOK — [arquivos desformatados] |
| Lint | [comando do config] | [N] issues (esperado: 0) |
| Testes | [comando do config] | TODOS PASSANDO / [N] falhando |
| Cobertura | [comando do config] | [resultado] — threshold [X]% OK / FAIL |

> Gate reprovado bloqueia o QA.

## Segurança
| Verificação | Resultado | Observações |
|-------------|-----------|-------------|
| Scanner de dependências vulneráveis | [comando] — [N] achados (alta/crítica: [N]) | [obs] |
| Baseline de segurança (`_shared/SECURITY_BASELINE.md`) | OK / [N] desvios | [obs] |
| Extensões de `<seguranca_extensoes>` | OK / [N] desvios | [obs] |

> Achado de severidade alta ou crítica reprova o QA e vira bug.

## Validação visual
[Preencher apenas se `<ui_projeto>` declarar ferramenta de validação visual.]

| Tela | Variações verificadas | Resultado | Observações |
|------|-----------------------|-----------|-------------|
| [nome da tela] | [ex.: temas suportados] | PASSOU/FALHOU/AUSENTE | [obs] |

> Comando: [comando do config]. Ausência de validação visual para tela nova = bug de severidade Alta.

## Testes ponta a ponta
- Ambiente/dispositivo utilizado: [descrição ou "nenhum disponível"]

| Alvo | Cenário | Resultado | Escreveu em ambiente compartilhado? |
|------|---------|-----------|--------------------------------------|
| [comando do config] | [cenário coberto] | PASSOU/FALHOU/NÃO EXECUTADO | sim/não |

> Se não executado, registre o motivo como pendência explícita — não reprove por isso.
> Alvos que escrevem em ambiente compartilhado exigem confirmação prévia do usuário.

## Requisitos Verificados
| ID | Requisito | Status | Evidência |
|----|-----------|--------|-----------|
| RF-01 | [descrição] | PASSOU/FALHOU | [evidência na pasta de evidências] |

## Fluxos testados
| Fluxo | Estados verificados | Resultado | Observações |
|-------|---------------------|-----------|-------------|
| [fluxo] | carregando / vazio / erro / sem conexão | PASSOU/FALHOU | [obs] |

## Acessibilidade
[Preencher apenas se `<ui_projeto>` indicar acessibilidade aplicável.]

| Verificação | Resultado | Observações |
|-------------|-----------|-------------|
| Rótulo descritivo em elementos interativos | OK / NOK | [obs] |
| Leitor de tela anuncia corretamente | OK / NOK | [obs] |
| Contraste adequado em todos os temas suportados | OK / NOK | [obs] |
| Alvo de interação com tamanho mínimo | OK / NOK | [obs] |
| Layout suporta aumento de escala de fonte | OK / NOK | [obs] |
| Navegação por teclado e ordem de foco | OK / NOK | [obs] |
| Mensagens de erro associadas ao campo correto | OK / NOK | [obs] |

## Verificação visual comparativa
[Preencher apenas se `<integracoes_projeto>` indicar ferramenta de design.]

| Tela | Referência de design | Aderência | Observações |
|------|----------------------|-----------|-------------|
| [tela] | [identificador ou —] | OK / divergente | [obs] |

- Tamanhos de tela verificados: [lista]
- Temas verificados: [lista]

## Bugs Encontrados
| ID | Descrição | Severidade | Evidência |
|----|-----------|------------|-----------|
| BUG-01 | [descrição] | Alta/Média/Baixa | [link] |

## Pendências
[Itens não verificados e o motivo.]

## Conclusão
[Parecer final do QA]
```

- [ ] **Step 5: Rodar o sweep de termos proibidos**

```bash
grep -rniE 'flutter|\bdart\b|\bbloc\b|cubit|modular|dio\b|mocktail|golden|talkback|voiceover|cy-loop|gpt-5|LIG-[0-9]|tool/coverage|make (ci|coverage|test)' \
  executar-qa/ | grep -viE 'ex\.?:|por exemplo|exemplo:'
```

Expected: saída vazia.

- [ ] **Step 6: Verificar que a verificação de segurança entrou**

```bash
grep -c 'SECURITY_BASELINE' executar-qa/SKILL.md
```

Expected: ≥ 1.

- [ ] **Step 7: Commit**

```bash
git add executar-qa/
git commit -m "refactor: generaliza executar-qa para qualquer stack

Metodo de validacao passa a depender do tipo de produto; adiciona
verificacao de vulnerabilidades ao gate de QA."
```

---

### Task 11: Generalizar `executar-review`

**Files:**
- Modify: `executar-review/SKILL.md`
- Modify: `executar-review/references/TEMPLATE.md`

**Interfaces:**
- Consumes: `techspec.md`, `tasks.md`, `../sdd.config.md`, ambas as baselines
- Produces: `codereview.md` com veredito APROVADO / APROVADO COM RESSALVAS / REPROVADO

- [ ] **Step 1: Substituir o cabeçalho**

Remova `<linear>`, `<roteamento_codex>`, `<contexto_projeto>` e `<comandos>`. Mantenha `<prd>` e
`<template>`. Insira o **Bloco de cabeçalho padrão**.

- [ ] **Step 2: Generalizar a conformidade arquitetural**

Substitua o checklist de camadas Flutter por:

```md
### 2. Conformidade arquitetural (obrigatório)

As regras invioláveis de `<limites_projeto>` são bloqueantes. Verifique:

- [ ] **Direção de dependência respeitada** — conforme a ordem de camadas de `<stack_projeto>`;
      nenhuma camada importa de quem depende dela
- [ ] **Camada de regra de negócio isolada** — sem dependência de framework de interface ou de
      biblioteca de acesso a dados, quando o padrão do projeto exigir isso
- [ ] **Erros tratados como parte do contrato**, no formato que o projeto adota
- [ ] **Pontos de entrada/rotas registrados** conforme o padrão do projeto
- [ ] **Injeção de dependência** conforme o padrão do projeto, sem acesso global espalhado
- [ ] **Nenhuma alteração em área read-only de `<limites_projeto>`**
- [ ] **SOLID e coesão** aplicados
```

- [ ] **Step 3: Generalizar padrões de código, testes e qualidade**

- "Conformidade com Padrões de Código": troque as regras Dart por "convenção da linguagem
  conforme `<qualidade_extensoes>` e a configuração de lint do projeto", mantendo os itens
  genéricos (format aplicado, lint sem warning, dependência nova justificada, reuso do código
  compartilhado, logging sem dado sensível)
- "Execução dos Testes": troque os comandos `make` pelos de `<comandos_projeto>`
- "Qualidade dos Testes": referencie `../_shared/QUALITY_BASELINE.md`, seção "Qualidade dos
  testes", em vez de repetir a lista
- "Análise de Qualidade de Código": substitua a tabela por referência a
  `../_shared/QUALITY_BASELINE.md` mais a linha de segurança apontando para
  `../_shared/SECURITY_BASELINE.md`, incluindo explicitamente o resultado do scanner de
  dependências vulneráveis
- "Documentação": condicional a `<documentacao_projeto>`
- Adicione ao checklist: "Convenção de commit/branch/PR de `<colaboracao_projeto>` respeitada"

- [ ] **Step 4: Generalizar os critérios de aprovação**

```md
**REPROVADO**: gate de `<comandos_projeto>` falhando, cobertura abaixo do threshold, violação de
direção de dependência ou de regra inviolável do config, alteração em área read-only, não
aderência à TechSpec, ou achado de segurança de severidade alta ou crítica.
```

- [ ] **Step 5: Reescrever `references/TEMPLATE.md`**

Substitua o arquivo inteiro por:

```md
# Relatório de Code Review - [Nome da Funcionalidade]

## Resumo
- Data: [data]
- Branch: [branch]
- Status: APROVADO / APROVADO COM RESSALVAS / REPROVADO
- Issue vinculada: [identificador no rastreador configurado ou "sem issue"]

## Conformidade Arquitetural (bloqueante)
[Uma linha por regra de direção de dependência de `<stack_projeto>` e por regra inviolável de
`<limites_projeto>`.]

| Regra | Status | Observações |
|-------|--------|-------------|
| Direção de dependência entre camadas respeitada | OK/NOK | [obs] |
| Camada de regra de negócio isolada conforme o padrão do projeto | OK/NOK | [obs] |
| Erros tratados no formato que o projeto adota | OK/NOK | [obs] |
| Pontos de entrada/rotas registrados conforme o padrão | OK/NOK | [obs] |
| Injeção de dependência conforme o padrão | OK/NOK | [obs] |
| Área read-only de `<limites_projeto>` intocada | OK/NOK | [obs] |
| SOLID e coesão | OK/NOK | [obs] |

## Conformidade com as Regras do Projeto
| Regra | Status | Observações |
|-------|--------|-------------|
| [regra de `<limites_projeto>` ou `<qualidade_extensoes>`] | OK/NOK | [obs] |

## Aderência à TechSpec
| Decisão Técnica | Implementado | Observações |
|-----------------|--------------|-------------|
| [decisão] | SIM/NÃO | [obs] |

## Tasks Verificadas
| Task | Status | Observações |
|------|--------|-------------|
| [task] | COMPLETA/INCOMPLETA | [obs] |

## Gate automatizado
| Etapa | Comando | Resultado |
|-------|---------|-----------|
| Format | [comando do config] | OK / NOK |
| Lint | [comando do config] | [N] issues (esperado: 0) |
| Testes | [comando do config] | TODOS PASSANDO / [N] falhando |
| Cobertura | [comando do config] | [resultado] — threshold [X]% OK / FAIL |

## Segurança
| Verificação | Status | Observações |
|-------------|--------|-------------|
| Baseline (`_shared/SECURITY_BASELINE.md`) | OK / [N] desvios | [obs] |
| Scanner de dependências vulneráveis | [N] achados (alta/crítica: [N]) | [obs] |
| Extensões de `<seguranca_extensoes>` | OK / [N] desvios | [obs] |

## Testes
- Total de Testes: [X] · Passando: [Y] · Falhando: [Z]
- **Cobertura: [%]** (threshold de `<comandos_projeto>`: [X]%)
- Validação visual para telas novas: OK / AUSENTE / N/A — [quais]
- Testes ponta a ponta atualizados: SIM / NÃO / N/A

### Qualidade dos testes
[Conforme `_shared/QUALITY_BASELINE.md`, seção "Qualidade dos testes".]

| Verificação | Status | Observações |
|-------------|--------|-------------|
| Testam comportamento, não implementação | OK/NOK | [obs] |
| Sem asserção fraca isolada | OK/NOK | [obs] |
| Caminhos de falha pareados com caminho feliz | OK/NOK | [obs] |
| Erro concreto e específico nos testes de falha | OK/NOK | [obs] |
| Mock na fronteira certa, biblioteca declarada no config | OK/NOK | [obs] |

## Convenções de colaboração
| Item | Status | Observações |
|------|--------|-------------|
| Convenção de commit de `<colaboracao_projeto>` | OK/NOK | [obs] |
| Convenção de branch/PR | OK/NOK | [obs] |
| Política de comentários no código | OK/NOK | [obs] |

## Documentação
[Conforme `<documentacao_projeto>`.]

| Item | Status | Caminho |
|------|--------|---------|
| Documentação da feature no repositório | OK/AUSENTE/N/A | [caminho] |
| Registro externo obrigatório | OK/AUSENTE/N/A | [caminho] |

## Problemas Encontrados
| Severidade | Arquivo | Linha | Descrição | Sugestão |
|------------|---------|-------|-----------|----------|
| Alta/Média/Baixa | [file] | [line] | [desc] | [fix] |

## Pontos Positivos
- [pontos positivos identificados]

## Recomendações
- [recomendações de melhoria]

## Conclusão
[Parecer final do review]
```

- [ ] **Step 6: Rodar o sweep de termos proibidos**

```bash
grep -rniE 'flutter|\bdart\b|pubspec|\bbloc\b|cubit|modular|dartz|dio\b|mocktail|golden|cy-loop|gpt-5|LIG-[0-9]|make (ci|coverage|analyze|format)' \
  executar-review/ | grep -viE 'ex\.?:|por exemplo|exemplo:'
```

Expected: saída vazia.

- [ ] **Step 7: Verificar que as duas baselines são referenciadas**

```bash
grep -c '_shared/' executar-review/SKILL.md
```

Expected: ≥ 2.

- [ ] **Step 8: Commit**

```bash
git add executar-review/
git commit -m "refactor: generaliza executar-review para qualquer stack

Conformidade arquitetural passa a usar as camadas do config; qualidade
e seguranca referenciam as baselines compartilhadas."
```

---

### Task 12: Escrever o README do bundle

**Files:**
- Create: `README.md`

**Interfaces:**
- Consumes: todas as skills e baselines criadas nas Tasks 1-11
- Produces: a porta de entrada humana do bundle

- [ ] **Step 1: Escrever `README.md`**

```md
# SDD — Fluxo de desenvolvimento orientado a especificação

Um bundle de skills portátil. Coloque em qualquer projeto, peça para a IA se adaptar, e o fluxo
PRD → TechSpec → Tasks → Execução → QA → Review → Bugfix passa a funcionar com a stack, as
convenções e as integrações daquele projeto.

## Como usar em um projeto novo

1. Copie a pasta inteira para o projeto. Pode ser na raiz (`sdd/`) ou direto em
   `.claude/skills/` — as duas formas funcionam.
2. Peça para a IA configurar, de um destes dois jeitos:
   - **Qualquer ferramenta de IA:** aponte o caminho e peça, ex.: *"leia `sdd/configurar-sdd/SKILL.md`
     e siga as instruções para configurar o SDD neste projeto"*
   - **Claude Code, com o bundle em `.claude/skills/`:** basta pedir em linguagem natural, ex.:
     *"configura o SDD neste projeto"*
3. A skill `configurar-sdd` lê os arquivos de regra do projeto, explora o repositório, pergunta
   só o que não conseguiu detectar, e grava o `sdd.config.md`.
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

- `configurar-sdd/` — a skill de adaptação e o esqueleto do arquivo de configuração
- `criar-*/` e `executar-*/` — as skills do fluxo, agnósticas de stack
- `_shared/` — baselines universais de segurança e de qualidade, carregadas pelas skills
- `sdd.config.md` — gerado por projeto; **é o único arquivo que muda de projeto para projeto**

## O que é fixo e o que é configurável

**Fixo** (vale para qualquer projeto): o fluxo em si, TDD, corrigir causa raiz em vez de sintoma,
a baseline de segurança e a de qualidade, e a regra de nunca baixar o threshold de cobertura para
fazer o gate passar.

**Configurável** (vem do `sdd.config.md`): linguagem, framework, arquitetura e ordem de camadas,
comandos de lint/teste/cobertura e o threshold, rastreador de issues, ferramenta de design, tipo
de produto e método de validação visual, acessibilidade e i18n, áreas read-only, convenções de
commit/branch/PR e de comentário, e onde documentar.

## Reconfigurar depois

Rode `configurar-sdd` de novo. Ela atualiza o `sdd.config.md` seção por seção, preservando o que
você editou à mão, e mostra o que muda antes de gravar. As skills não são reescritas em nenhum
momento.

## Versionamento

No bundle mestre, `sdd.config.md` é ignorado pelo git, porque é gerado por projeto. Dentro de um
projeto que usa o SDD, versione o `sdd.config.md` normalmente: ele documenta as convenções
acordadas pelo time.
```

- [ ] **Step 2: Verificar que as 8 skills estão listadas**

```bash
for s in configurar-sdd criar-prd criar-techspec criar-tasks executar-task executar-qa executar-bugfix executar-review; do
  grep -q "$s" README.md && echo "OK $s" || echo "FALTA $s"
done
```

Expected: oito linhas `OK`.

- [ ] **Step 3: Commit**

```bash
git add README.md
git commit -m "docs: adiciona README do bundle SDD"
```

---

### Task 13: Verificação final e simulação a seco

**Files:**
- Modify: nenhum (tarefa de verificação; correções pontuais se algo falhar)

**Interfaces:**
- Consumes: todo o bundle (Tasks 1-12)
- Produces: confirmação de que o bundle está pronto para uso

- [ ] **Step 1: Rodar o sweep completo de termos proibidos**

```bash
grep -rniE 'flutter|\bdart\b|pubspec|\bbloc\b|cubit|modular|dartz|\bdio\b|mocktail|nestjs|golden|talkback|voiceover|cy-loop|gpt-5|LIG-[0-9]|Liga Coop|tool/coverage|make (ci|coverage|analyze|test|format)' \
  criar-* executar-* configurar-sdd/SKILL.md _shared/ README.md 2>/dev/null \
  | grep -viE 'ex\.?:|por exemplo|exemplo:'
```

Expected: saída vazia. Se houver hits, corrija o arquivo apontado e rode de novo.

- [ ] **Step 2: Verificar que toda skill referencia o config pelo caminho certo**

```bash
for d in criar-prd criar-techspec criar-tasks executar-task executar-qa executar-bugfix executar-review; do
  c=$(grep -c '\.\./sdd\.config\.md' "$d/SKILL.md")
  echo "$d: $c"
done
```

Expected: todas com contagem ≥ 2, nenhuma com `0`.

- [ ] **Step 3: Verificar que nenhuma skill referencia caminho antigo do config**

```bash
grep -rn '\.claude/sdd\.config\|@\.claude/CLAUDE\.md\|@rules\.md\|@AGENTS\.md' \
  criar-* executar-* configurar-sdd/ _shared/ 2>/dev/null
```

Expected: saída vazia (esses arquivos agora são *procurados* pelo `configurar-sdd`, não
referenciados fixamente pelas skills do fluxo).

- [ ] **Step 4: Simulação a seco — projeto web React**

Leia `configurar-sdd/SKILL.md` e percorra o fluxo mentalmente contra este perfil:

```
package.json: react, vite, vitest, @testing-library/react
scripts: dev, build, lint (eslint), test (vitest), test:coverage (sem threshold)
estrutura: src/components, src/hooks, src/pages, src/lib
sem CLAUDE.md, sem rules.md, sem commitlint, sem template de PR
```

Confirme, por escrito, que: os campos de `<stack_projeto>` e `<comandos_projeto>` saem
detectados; `<ui_projeto>` sai como web com UI; o threshold e as convenções de commit/PR viram
pergunta; e nenhum campo exige conhecimento de Flutter.

- [ ] **Step 5: Simulação a seco — API Python sem UI**

Percorra o mesmo fluxo contra este perfil:

```
pyproject.toml: fastapi, sqlalchemy, pytest, pytest-cov (fail_under = 85)
Makefile: lint (ruff), test, coverage
estrutura: app/api, app/services, app/repositories, app/models
CONTRIBUTING.md exige Conventional Commits
```

Confirme, por escrito, que: o threshold sai detectado como 85; `<ui_projeto>` sai como "sem UI",
desligando acessibilidade, i18n e validação visual nas skills seguintes; a convenção de commit
sai detectada do `CONTRIBUTING.md`; e as camadas viram a ordem de sequenciamento de `criar-tasks`.

- [ ] **Step 6: Checklist de paridade com a versão original**

Confirme que cada regra abaixo, que existia na versão Flutter, tem equivalente genérico ou
condicional na nova versão. Marque uma a uma:

- [ ] Perguntas de esclarecimento obrigatórias antes do PRD
- [ ] Não desviar do template
- [ ] PRD não contém implementação
- [ ] Explorar o projeto antes de perguntar (TechSpec)
- [ ] Contrato de API confirmado no código, não suposto
- [ ] Lista de tarefas aprovada antes de gerar arquivos
- [ ] TDD: subtarefa de teste antes da subtarefa de implementação
- [ ] Teste visual antes da tela (quando aplicável)
- [ ] Documentação como última tarefa
- [ ] Causa raiz, nunca sintoma
- [ ] Teste de regressão falhando antes da correção
- [ ] Não regenerar artefato visual sem aprovação
- [ ] Nunca baixar o threshold de cobertura
- [ ] Pedir confirmação antes de escrever em ambiente compartilhado
- [ ] Área read-only nunca alterada
- [ ] Gate verde obrigatório para fechar tarefa, bugfix, QA e review
- [ ] Qualidade dos testes avaliada além da cobertura

- [ ] **Step 7: Commit final**

```bash
git add -A
git commit -m "chore: verificacao final do bundle SDD generalizado

Sweep de termos especificos de stack limpo, caminhos de config
consistentes e paridade de regras com a versao original confirmada."
```
