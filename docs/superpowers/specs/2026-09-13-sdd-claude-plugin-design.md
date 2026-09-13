# SDD — Distribuição via plugin do Claude Code

**Data:** 2026-09-13
**Status:** Aprovado para plano de implementação

## Contexto e problema

A instalação do SDD hoje é manual: baixar o repo do GitHub, copiar as pastas certas (nomeando
cada uma, para não arrastar `.git`/`.gitignore` do bundle mestre) e pedir para a IA ler
`configurar-sdd/SKILL.md`. Isso funciona para qualquer ferramenta de IA, mas tem fricção para
compartilhar internamente: cada colega repete o clone + cópia + prompt na mão.

O Claude Code tem um sistema nativo de plugins/marketplace — confirmado neste mesmo ambiente,
onde os plugins `superpowers` e `figma` já chegam instalados por esse mecanismo
(`~/.claude/plugins/installed_plugins.json`, `known_marketplaces.json`). Um repo git vira
"marketplace" com um `.claude-plugin/marketplace.json` na raiz; cada plugin tem
`.claude-plugin/plugin.json`. Instalação vira `claude plugin marketplace add <repo>` +
`claude plugin install <plugin>`, sem clone manual.

**Achado técnico que molda o desenho:** a doc oficial do Claude Code (`skills/plugin-structure`
do plugin `plugin-dev`) documenta que **skills só são descobertas automaticamente dentro de uma
pasta `skills/` na raiz do plugin** — ao contrário de `commands`/`agents`/`hooks`/`mcpServers`,
não existe campo no manifesto para apontar um caminho customizado para skills. As 7 skills do
fluxo do SDD hoje ficam soltas na raiz do bundle, e `.sdd/*`/`qa/*` ficam aninhadas, todas com
caminhos relativos fixos (`../sdd.config.md`, `../../sdd.config.md`) que dependem dessa
profundidade exata. Mover tudo para dentro de `skills/` mudaria a profundidade de todo mundo e
exigiria reescrever esses caminhos em cada skill.

Decisão: **não tocar em nenhuma skill existente.** O plugin expõe exatamente **uma** skill nova
(`skills/instalar-sdd/`), que automatiza o passo manual de copiar + entregar para
`configurar-sdd` que hoje um humano faz na mão. Todo o resto do bundle (as 7 skills de fluxo,
`.sdd/`, `qa/`) continua existindo do mesmo jeito, apenas referenciado por essa skill via
`${CLAUDE_PLUGIN_ROOT}`.

## Objetivos

1. Adicionar `.claude-plugin/plugin.json` e `.claude-plugin/marketplace.json` na raiz do bundle
   mestre, tornando o próprio repo instalável via
   `claude plugin marketplace add FelipeAngeli/sdd` + `claude plugin install sdd`.
2. Adicionar uma skill nova, `skills/instalar-sdd/SKILL.md`, que reproduz o passo 1 do README
   (copiar as pastas certas para o projeto do usuário, perguntando destino — raiz ou
   `.claude/skills/` — e se quer `qa/` já nesta cópia) e em seguida entrega para o passo 2
   (ler `configurar-sdd/SKILL.md` no caminho recém-copiado e seguir as instruções).
3. Nova seção no `README.md` descrevendo esse caminho para usuários de Claude Code, deixando
   explícito que o repo é **privado** e que colegas precisam ser adicionados como collaborators
   antes de rodar `claude plugin marketplace add`.
4. `plugin.json` com `license: "UNLICENSED"` (uso interno, sem revisão jurídica formal), autor
   Felipe Angeli / felipe.trebien@keltech.app, `repository`/`homepage` apontando para o repo
   privado, `version` espelhando o `VERSION` do bundle no momento do release.
5. O método manual de cópia continua documentado como está, sem alteração — é o caminho
   universal para quem não usa Claude Code.

## Não-objetivos

- Não mover as 7 skills de fluxo nem `.sdd/`/`qa/` para dentro de `skills/` — nenhuma reescrita
  de caminho relativo nas skills existentes.
- Não implementar o modelo "compartilhado, sem cópia por projeto" (skills servidas ao vivo do
  cache do plugin) — descartado por complexidade e por eliminar a customização de skill por
  projeto que o `atualizar-sdd` já trata hoje.
- Não mudar a visibilidade/dono do repo — continua privado, sob `FelipeAngeli`.
- Não criar publicação automatizada (CI) para nenhum marketplace público — é um marketplace de
  um repo só, para distribuição interna.
- Não alterar o comportamento de diff do `atualizar-sdd` — a skill nova é puramente aditiva.

## Arquitetura da solução

| Camada | O quê | Onde vive |
| --- | --- | --- |
| Manifesto do plugin | nome, versão, descrição, autor, license | `.claude-plugin/plugin.json` |
| Registro de marketplace | 1 plugin listado, `"source": "./"` | `.claude-plugin/marketplace.json` |
| Skill de bootstrap | única skill descoberta automaticamente pelo plugin | `skills/instalar-sdd/SKILL.md` |
| Documentação | seção nova para o caminho Claude Code | `README.md` |

### `plugin.json`

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

### `marketplace.json`

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

### Skill nova: `instalar-sdd`

**Gatilhos de descrição:** "instala o SDD neste projeto", "traz o bundle SDD para cá", "configura
o SDD a partir do plugin".

**Fluxo:**

1. **Perguntar destino** — raiz do projeto (`sdd/`) ou `.claude/skills/`, mesma escolha que o
   README já apresenta para a cópia manual.
2. **Perguntar se quer a camada de QA contínuo** (`qa/`) já nesta cópia — mesma decisão que hoje
   é feita ao escolher quais pastas copiar na mão.
3. **Copiar** de `${CLAUDE_PLUGIN_ROOT}/{criar-prd,criar-techspec,criar-tasks,executar-task,
   executar-qa,executar-bugfix,executar-review,.sdd,VERSION}` (+ `qa/` se pedido) para o destino
   escolhido — pasta por pasta, nomeada, nunca `cp -a`/`cp -r` do diretório inteiro (mesma
   ressalva que o README já faz para a cópia manual, para não arrastar nada indesejado).
4. **Entregar para `configurar-sdd`** — ler `<destino>/.sdd/configurar-sdd/SKILL.md` e seguir
   integralmente as instruções, exatamente como o passo 2 do README manual já descreve.

### Fluxo ponta a ponta para um colega

1. Colega recebe acesso de collaborator no `FelipeAngeli/sdd` (repo privado).
2. `claude plugin marketplace add FelipeAngeli/sdd`
3. `claude plugin install sdd@sdd`
4. Dentro do projeto dele: `/instalar-sdd` → responde destino e QA contínuo → a skill copia os
   arquivos e segue direto para `configurar-sdd` → fluxo de configuração de sempre
   (perguntas, `sdd.config.md`, atalhos).
5. Dali em diante, o projeto do colega tem sua própria cópia versionada — igual a quem instalou
   manualmente hoje. `atualizar-sdd` funciona sem nenhuma diferença.

### Inventário de arquivos

**Novos:**

- `.claude-plugin/plugin.json`
- `.claude-plugin/marketplace.json`
- `skills/instalar-sdd/SKILL.md`

**Modificados:**

- `README.md` — nova seção "Instalar via plugin do Claude Code", ao lado da cópia manual.
- `VERSION` — sobe ao final da implementação, em conjunto com o `version` do `plugin.json` e do
  `marketplace.json`.

**Não tocados:** `criar-prd`, `criar-techspec`, `criar-tasks`, `executar-task`, `executar-qa`,
`executar-bugfix`, `executar-review`, `.sdd/configurar-sdd`, `.sdd/atualizar-sdd`, `qa/*`,
`.sdd/_shared/*`.

## Validação

1. **Dry-run local:** `claude plugin marketplace add /Users/felipe/Desktop/sdd` (caminho local) e
   `claude plugin install sdd@sdd` num projeto de teste descartável; confirmar que `/instalar-sdd`
   aparece, rodar de ponta a ponta, e confirmar que `/configurar-sdd` aparece em seguida.
2. `plugin.json`/`marketplace.json` são JSON válido; `name` casa com o padrão kebab-case exigido
   pelo Claude Code.
3. `git diff` vazio nas 7 skills de fluxo, em `.sdd/*` e em `qa/*`.
4. Nova seção do README revisada para não contradizer as instruções de cópia manual existentes.

## Riscos e premissas

- **Premissa:** colegas têm acesso git configurado (SSH/HTTPS com token) para o repo privado —
  sem isso, `claude plugin marketplace add` falha ao clonar. Precisam virar collaborator antes.
- **Risco:** `plugin.json`/`marketplace.json` e o `VERSION` do bundle podem divergir se alguém
  esquecer de subir os três juntos num release — não há automação para isso neste spec; fica
  registrado como lembrete de processo, não como mecanismo novo.
- **Premissa:** `"UNLICENSED"` no `plugin.json` comunica uso interno de forma suficiente para o
  caso de uso atual — não substitui uma revisão jurídica formal se o bundle for compartilhado
  além da empresa no futuro.
- **Risco:** o nome de plugin `sdd` é genérico e pode colidir com outro plugin que um colega
  já tenha instalado com o mesmo nome — aceitável para uso interno agora; renomear depois é uma
  mudança breaking para quem já instalou.
