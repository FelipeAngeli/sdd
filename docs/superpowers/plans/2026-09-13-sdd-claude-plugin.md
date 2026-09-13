# Distribuição do SDD via plugin do Claude Code — Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Tornar o bundle mestre instalável como plugin do Claude Code (`claude plugin marketplace add` + `claude plugin install`), substituindo o passo manual de "baixar do GitHub e copiar pasta" por um comando, sem alterar nenhuma das 7 skills de fluxo nem `.sdd/`/`qa/`.

**Architecture:** Dois arquivos de manifesto (`plugin.json`, `marketplace.json`) na raiz do bundle tornam o próprio repositório um marketplace de um plugin só (`"source": "./"`). O plugin expõe exatamente **uma** skill nova, `skills/instalar-sdd/`, na única pasta que o Claude Code descobre automaticamente para skills de plugin — ela copia as pastas do fluxo de `$CLAUDE_PLUGIN_ROOT` para o projeto do usuário (reproduzindo o passo 1 do README) e entrega para `configurar-sdd` (passo 2). Nada além desses três arquivos novos e uma seção do README muda.

**Tech Stack:** JSON (manifestos de plugin), Markdown com frontmatter YAML (`SKILL.md`, no mesmo formato das demais skills do bundle), `claude` CLI (`plugin marketplace`, `plugin install`, `plugin details`, `plugin tag`) para validação real do manifesto e da descoberta de componentes, `git`.

**Spec:** `docs/superpowers/specs/2026-09-13-sdd-claude-plugin-design.md`

## Global Constraints

Estas regras valem para **todas** as tarefas. Cada tarefa as inclui implicitamente.

1. **Idioma:** todo o conteúdo é escrito em português (PT-BR). Termos técnicos consagrados (plugin, marketplace, commit, scope) permanecem em inglês.

2. **Nenhuma das 7 skills de fluxo, `.sdd/*` ou `qa/*` pode ser modificada.** Ao fim deste plano, `git diff` nessas pastas precisa estar vazio.

3. **Cópia sempre nomeada, nunca o diretório inteiro.** Qualquer comando de cópia dentro da skill `instalar-sdd` nomeia cada pasta — nunca `cp -a $CLAUDE_PLUGIN_ROOT/. destino/` nem `cp -r $CLAUDE_PLUGIN_ROOT/* destino/`. A skill `instalar-sdd` e o `.claude-plugin/` nunca são copiados para o destino — eles são infraestrutura do plugin, não parte do bundle portátil.

4. **Nomes:** `"sdd"` é o nome do plugin **e** do marketplace (repo de um plugin só). Kebab-case, casa com `^[a-z][a-z0-9]*(-[a-z0-9]+)*$`.

5. **Versão:** `plugin.json`, o único plugin listado em `marketplace.json` e `VERSION` sobem juntos, sempre para o mesmo valor. `claude plugin tag --dry-run` é o jeito de confirmar que os dois manifestos concordam antes de qualquer commit de versão.

6. **Commits:** Conventional Commits, mensagem em português, um commit por task (a Task 3, de validação pura, não gera commit).

7. **`license: "UNLICENSED"`** em `plugin.json` — uso interno, repo privado, sem revisão jurídica formal.

8. **Autor em todo manifesto:** `{"name": "Felipe Angeli", "email": "felipe.trebien@keltech.app"}`.

---

### Task 1: Manifestos do plugin

**Files:**
- Create: `.claude-plugin/plugin.json`
- Create: `.claude-plugin/marketplace.json`

**Interfaces:**
- Consumes: nada (primeira tarefa)
- Produces: os dois manifestos que `claude plugin marketplace add`/`install` leem. Task 3 valida os dois via CLI; Task 5 sobe a versão dos dois.

- [ ] **Step 1: Criar `.claude-plugin/plugin.json`**

```json
{
  "name": "sdd",
  "version": "1.3.0",
  "description": "Fluxo de desenvolvimento orientado a especificação (PRD -> TechSpec -> Tasks -> Execução -> QA -> Bugfix -> Review) como skills portáteis para Claude Code.",
  "author": { "name": "Felipe Angeli", "email": "felipe.trebien@keltech.app" },
  "homepage": "https://github.com/FelipeAngeli/sdd",
  "repository": "https://github.com/FelipeAngeli/sdd",
  "license": "UNLICENSED",
  "keywords": ["sdd", "spec-driven-development", "prd", "techspec", "qa", "workflow", "skills"]
}
```

- [ ] **Step 2: Criar `.claude-plugin/marketplace.json`**

```json
{
  "name": "sdd",
  "description": "Marketplace interno do bundle SDD",
  "owner": { "name": "Felipe Angeli", "email": "felipe.trebien@keltech.app" },
  "plugins": [
    {
      "name": "sdd",
      "description": "Fluxo de desenvolvimento orientado a especificação como skills portáteis.",
      "version": "1.3.0",
      "source": "./",
      "author": { "name": "Felipe Angeli", "email": "felipe.trebien@keltech.app" }
    }
  ]
}
```

- [ ] **Step 3: Validar sintaxe JSON dos dois arquivos**

```bash
python3 -m json.tool .claude-plugin/plugin.json > /dev/null && echo "plugin.json OK"
python3 -m json.tool .claude-plugin/marketplace.json > /dev/null && echo "marketplace.json OK"
```

Expected: as duas linhas `OK` impressas, sem erro de parsing.

- [ ] **Step 4: Verificar que o nome casa com o padrão kebab-case exigido pelo Claude Code**

```bash
grep -o '"name": *"[^"]*"' .claude-plugin/plugin.json .claude-plugin/marketplace.json
echo "sdd" | grep -qE '^[a-z][a-z0-9]*(-[a-z0-9]+)*$' && echo "nome valido"
```

Expected: as duas ocorrências mostram `"name": "sdd"`, e a linha `nome valido` aparece.

- [ ] **Step 5: Commit**

```bash
git add .claude-plugin/plugin.json .claude-plugin/marketplace.json
git commit -m "feat: adiciona manifestos de plugin do Claude Code ao bundle

Torna o proprio repo um marketplace de um plugin so (source: ./),
sem alterar nenhuma skill existente. E o primeiro passo para instalar
o SDD via claude plugin install em vez de clone manual."
```

---

### Task 2: Skill nova `instalar-sdd`

**Files:**
- Create: `skills/instalar-sdd/SKILL.md`

**Interfaces:**
- Consumes: `$CLAUDE_PLUGIN_ROOT` (variável de ambiente que o Claude Code injeta ao rodar uma skill de um plugin instalado — aponta para a raiz deste mesmo repositório, onde estão `criar-prd/`, `.sdd/`, `qa/`, `VERSION`).
- Produces: a cópia das pastas do fluxo dentro do projeto do usuário, e a entrega para `<destino>/.sdd/configurar-sdd/SKILL.md`. Task 3 valida que esta skill é descoberta pelo `claude plugin details`.

- [ ] **Step 1: Criar `skills/instalar-sdd/SKILL.md`**

```md
---
name: instalar-sdd
description: Copia o bundle SDD deste plugin para o projeto atual e entrega para configurar-sdd. Use ao instalar o SDD pela primeira vez num projeto via plugin do Claude Code — gatilhos como "instala o SDD aqui", "traz o bundle SDD para este projeto".
---

<origem>`$CLAUDE_PLUGIN_ROOT` — a raiz deste plugin, resolvida pelo Claude Code em tempo de execução; contém as 7 pastas do fluxo, `.sdd/`, `qa/` e `VERSION`, no mesmo layout do bundle mestre</origem>

## Persona

Você está automatizando o passo manual que o `README.md` do bundle já descreve: copiar as pastas
certas para o projeto do usuário e entregar para `configurar-sdd`. Você não configura nada
sozinho — só copia e entrega.

<critical>Copie SEMPRE nomeando as pastas, uma por uma. Nunca `cp -a "$CLAUDE_PLUGIN_ROOT"/. destino/` nem `cp -r "$CLAUDE_PLUGIN_ROOT"/* destino/` — os dois arrastariam arquivos que não fazem parte do bundle portátil, como `.claude-plugin/` e esta própria skill (`skills/instalar-sdd/`).</critical>
<critical>NÃO configure nada nesta skill: nenhuma pergunta de stack, comando ou convenção. Ao final, entregue para `configurar-sdd` e pare — configurar é responsabilidade dela, não desta skill.</critical>

## Fluxo

### 1. Perguntar o destino

Pergunte ao usuário onde a cópia deve ficar:

- raiz do projeto, numa pasta `sdd/`
- direto em `.claude/skills/`

### 2. Perguntar sobre QA contínuo

Pergunte se o projeto já sabe que quer a camada opcional de QA contínuo (pasta `qa/`). Se o
usuário não souber, diga que dá para ativar depois — `configurar-sdd` pergunta de novo, e
`atualizar-sdd` traz a pasta quando for a hora — e siga sem copiar `qa/` agora.

### 3. Copiar

Com o destino escolhido em `<DESTINO>`, crie a pasta se não existir e copie, pasta por pasta:

```sh
mkdir -p <DESTINO>
cp -R "$CLAUDE_PLUGIN_ROOT/criar-prd" "$CLAUDE_PLUGIN_ROOT/criar-techspec" "$CLAUDE_PLUGIN_ROOT/criar-tasks" \
      "$CLAUDE_PLUGIN_ROOT/executar-task" "$CLAUDE_PLUGIN_ROOT/executar-qa" "$CLAUDE_PLUGIN_ROOT/executar-bugfix" \
      "$CLAUDE_PLUGIN_ROOT/executar-review" "$CLAUDE_PLUGIN_ROOT/.sdd" "$CLAUDE_PLUGIN_ROOT/VERSION" <DESTINO>/
```

Se o usuário pediu QA contínuo na etapa 2, copie também:

```sh
cp -R "$CLAUDE_PLUGIN_ROOT/qa" <DESTINO>/
```

### 4. Entregar para `configurar-sdd`

Leia `<DESTINO>/.sdd/configurar-sdd/SKILL.md` e siga integralmente as instruções dessa skill para
configurar o SDD neste projeto. A skill `instalar-sdd` termina aqui — tudo que vem depois
(explorar o projeto, perguntar gaps, gravar `sdd.config.md`, atalhos) é responsabilidade dela.

## Checklist de qualidade

- [ ] Destino perguntado e confirmado antes de copiar
- [ ] QA contínuo perguntado antes de copiar, não depois
- [ ] Cópia feita pasta por pasta, nomeada — nunca o diretório inteiro
- [ ] `.claude-plugin/` e `skills/instalar-sdd/` (esta própria skill) nunca copiados para o destino
- [ ] Entregue para `configurar-sdd` ao final, sem configurar nada nesta skill
```

- [ ] **Step 2: Verificar frontmatter e ausência de auto-cópia**

```bash
grep -n '^name: instalar-sdd$\|^description:' skills/instalar-sdd/SKILL.md
```

Expected: as duas linhas do frontmatter aparecem.

```bash
grep -n 'CLAUDE_PLUGIN_ROOT/skills\|CLAUDE_PLUGIN_ROOT/\.claude-plugin' skills/instalar-sdd/SKILL.md
```

Expected: saída vazia — a skill nunca referencia copiar a si mesma ou os manifestos do plugin.

- [ ] **Step 3: Commit**

```bash
git add skills/instalar-sdd
git commit -m "feat: adiciona a skill instalar-sdd

Unica skill que o plugin expoe automaticamente (skills/ e a pasta que
o Claude Code descobre sozinho). Copia as pastas do fluxo de
\$CLAUDE_PLUGIN_ROOT para o projeto do usuario e entrega para
configurar-sdd, substituindo o passo manual do README."
```

---

### Task 3: Validação real com o CLI do Claude Code

**Files:** nenhum arquivo do repositório é criado ou modificado — tarefa de verificação pura.

**Interfaces:**
- Consumes: os manifestos (Task 1) e a skill `instalar-sdd` (Task 2).
- Produces: confirmação de que `claude plugin marketplace add`/`claude plugin details` enxergam o plugin e a skill corretamente. Nenhuma tarefa depois desta consome saída sua.

> Esta tarefa registra um marketplace local e instala o plugin num projeto descartável, ambos com `--scope local` — escopo mais contido, que não toca a configuração global (`user`) do Claude Code. O último step remove os dois.

- [ ] **Step 1: Confirmar que nenhuma skill existente foi tocada pelas Tasks 1-2**

```bash
git status --porcelain -- criar-prd criar-techspec criar-tasks executar-task executar-qa executar-bugfix executar-review qa .sdd
```

Expected: saída vazia — as 7 skills de fluxo, `.sdd/*` e `qa/*` não aparecem como modificadas nem
como não rastreadas.

- [ ] **Step 2: Registrar o repo como marketplace local**

```bash
claude plugin marketplace add "$(pwd)" --scope local
claude plugin marketplace list
```

Expected: `sdd` aparece na listagem, apontando para o caminho local do repositório.

- [ ] **Step 3: Instalar num projeto descartável**

```bash
SCRATCH_PROJ="/private/tmp/claude-501/-Users-felipe-Desktop-sdd/d07dac0a-9215-474f-905e-a533f53bfe98/scratchpad/sdd-plugin-validation"
rm -rf "$SCRATCH_PROJ"
mkdir -p "$SCRATCH_PROJ"
cd "$SCRATCH_PROJ" && git init -q
claude plugin install sdd@sdd --scope local -y
claude plugin list --json | grep -o '"sdd@sdd"'
```

Expected: a última linha imprime `"sdd@sdd"`, confirmando o plugin instalado no escopo local desse
projeto.

- [ ] **Step 4: Confirmar que a skill `instalar-sdd` é descoberta**

> **Ordem corrigida em execução (Task 3, ronda 1):** `claude plugin details` exige o plugin
> **instalado**, não só o marketplace registrado — rodar este step antes do Step 3 (ordem
> original do plano) falha com "Plugin not found". Rode de dentro de `$SCRATCH_PROJ`, depois do
> Step 3.

```bash
cd "$SCRATCH_PROJ" && claude plugin details sdd@sdd
```

Expected: a saída lista `instalar-sdd` no inventário de skills do plugin (nenhuma outra skill — as 7 do fluxo e `.sdd/`/`qa/` não são descobertas por não estarem em `skills/`, como o desenho previu).

- [ ] **Step 5: Verificação manual (não automatizável em bash)**

Abra uma sessão interativa do Claude Code dentro de `$SCRATCH_PROJ` e rode `/instalar-sdd`.
Confirme visualmente: a skill pergunta destino e QA contínuo, copia as pastas certas, e entrega
para `configurar-sdd` (que passa a rodar em seguida). Este step é observação humana — o resto da
tarefa já confirmou mecanicamente que o plugin e a skill existem e são descobertos; este é o
equivalente à "simulação a seco" que o resto do bundle usa para validar comportamento de skill.

- [ ] **Step 6: Limpar — remover o plugin, o marketplace local e a pasta descartável**

```bash
cd "$SCRATCH_PROJ" && claude plugin uninstall sdd --scope local -y
cd /Users/felipe/Desktop/sdd
claude plugin marketplace remove sdd --scope local
rm -rf "$SCRATCH_PROJ"
claude plugin marketplace list
```

Expected: `sdd` não aparece mais na listagem de marketplaces, e `$SCRATCH_PROJ` não existe mais.

---

### Task 4: README — nova seção de instalação via plugin

**Files:**
- Modify: `README.md`

**Interfaces:**
- Consumes: nada além do texto já existente no README.
- Produces: a documentação que um colega segue para instalar via plugin — nenhuma outra tarefa consome isto.

- [ ] **Step 1: Inserir a seção nova, logo após "## Como usar em um projeto novo"**

Abra `README.md` e, imediatamente **depois** do bloco numerado 1-4 dessa seção (antes do próximo
`## O fluxo`), insira:

```md
### Instalar via plugin do Claude Code

Se você usa Claude Code, existe um caminho mais curto que os 4 passos acima: o próprio repo é um
marketplace de plugin.

1. Peça para ser adicionado como collaborator no repositório (é privado).
2. `claude plugin marketplace add FelipeAngeli/sdd`
3. `claude plugin install sdd@sdd`
4. Dentro do seu projeto, rode `/instalar-sdd` — ela pergunta o destino da cópia e se você quer a
   camada de QA contínuo, copia as pastas certas e entrega direto para `/configurar-sdd`.

Dali em diante não há diferença nenhuma para quem instalou manualmente: mesma cópia versionada no
seu projeto, mesmo `atualizar-sdd`, mesmo `sdd.config.md`. O que muda é só como o bundle chegou
até aqui.
```

- [ ] **Step 2: Verificar a inserção**

```bash
grep -n '^## Como usar em um projeto novo\|^### Instalar via plugin do Claude Code\|^## O fluxo' README.md
```

Expected: as três linhas aparecem nesta ordem.

- [ ] **Step 3: Commit**

```bash
git add README.md
git commit -m "docs: documenta a instalacao via plugin do Claude Code no README

Caminho alternativo ao passo manual de copiar pastas: marketplace add
+ plugin install + /instalar-sdd. O metodo manual continua documentado
sem alteracao, como caminho universal para quem nao usa Claude Code."
```

---

### Task 5: Subir a versão do bundle

**Files:**
- Modify: `VERSION`
- Modify: `.claude-plugin/plugin.json`
- Modify: `.claude-plugin/marketplace.json`

**Interfaces:**
- Consumes: os manifestos criados na Task 1.
- Produces: os três arquivos na versão `1.4.0`, validados por `claude plugin tag --dry-run` como consistentes entre si.

- [ ] **Step 1: Subir `VERSION`**

Troque o conteúdo de `VERSION` (hoje `1.3.0`) para:

```
1.4.0
```

- [ ] **Step 2: Sincronizar a versão nos dois manifestos**

Em `.claude-plugin/plugin.json`, troque `"version": "1.3.0"` por `"version": "1.4.0"`.
Em `.claude-plugin/marketplace.json`, troque a mesma ocorrência dentro do plugin listado.

- [ ] **Step 3: Verificar que as três versões concordam**

```bash
grep -h '"version"' .claude-plugin/plugin.json .claude-plugin/marketplace.json
cat VERSION
```

Expected: as duas linhas de `"version"` mostram `1.4.0`, e `VERSION` mostra `1.4.0`.

- [ ] **Step 4: Validar com o CLI que plugin.json e marketplace.json concordam**

```bash
claude plugin tag --dry-run
```

Expected: a saída confirma `plugin.json` e a entrada de `marketplace.json` na mesma versão, sem
erro de divergência (nenhuma tag é criada, por causa de `--dry-run`).

- [ ] **Step 5: Commit**

```bash
git add VERSION .claude-plugin/plugin.json .claude-plugin/marketplace.json
git commit -m "chore: sobe o bundle para 1.4.0

Acompanha a distribuicao via plugin do Claude Code (manifestos +
skill instalar-sdd) adicionada neste ciclo."
```
