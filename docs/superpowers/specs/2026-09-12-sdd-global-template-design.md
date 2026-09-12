# SDD — Template global adaptável a qualquer projeto

**Data:** 2026-09-12
**Status:** Aprovado para plano de implementação

## Contexto e problema

O bundle atual em `/Users/felipe/Desktop/sdd/` contém 7 skills do Claude Code (`criar-prd`,
`criar-techspec`, `criar-tasks`, `executar-task`, `executar-bugfix`, `executar-qa`,
`executar-review`) que implementam um fluxo SDD (PRD → TechSpec → Tasks → Execução → Bugfix →
QA → Review). Todo o conteúdo — SKILL.md e os 7 templates em `references/` — está hard-coded
para um único projeto: app Flutter mobile "Liga Coop Passageiro" (Clean Architecture +
`flutter_modular` + BLoC/Cubit + `dartz` + `dio`), com integrações fixas a Linear, Figma e
Obsidian, cobertura mínima de 90%, golden tests, e um roteamento para um orquestrador Codex
específico.

Objetivo: transformar esse bundle num **template portável**. O usuário copia a mesma pasta de
skills para qualquer projeto novo (qualquer linguagem/stack), pede para a IA se adaptar, e o
fluxo SDD passa a funcionar corretamente para aquele projeto — sem precisar reescrever nada à
mão.

## Objetivos

1. Um único comando/pedido em linguagem natural ("adapta esse SDD pra esse projeto", "configura
   o SDD aqui") aciona uma skill dedicada que investiga o projeto novo e o configura. O
   acionamento funciona tanto apontando o caminho da pasta explicitamente para qualquer IA
   quanto por descoberta automática no Claude Code (ver "Invocação em dois modos" abaixo).
2. A skill de adaptação **explora primeiro** (manifestos, estrutura, testes, CI, regras
   existentes) e só pergunta ao usuário o que não dá para inferir.
3. As 7 skills existentes deixam de ter qualquer menção fixa a Flutter/Dart/Linear/Figma/Obsidian
   e passam a se comportar corretamente para qualquer stack, a partir de uma configuração gerada
   pela adaptação.
4. Segurança e qualidade de código mantêm uma base universal e obrigatória (incluindo cuidados
   explícitos com vulnerabilidades), estendida com regras específicas de cada projeto.
5. Acessibilidade, i18n e validação visual (hoje golden tests) ficam condicionais ao tipo de
   projeto (tem UI ou não, e que tipo).
6. Convenções de colaboração do projeto (GitHub, padrão de commit, branch/PR, comentários no
   código) também entram na configuração, para que qualquer skill que precise segui-las saiba
   como.
7. Rodar a adaptação de novo, mais tarde, atualiza a configuração sem tocar nas skills.

### Invocação em dois modos

O bundle precisa funcionar em qualquer lugar em que o usuário o coloque (raiz do projeto, dentro
de `.claude/skills/`, ou qualquer outro caminho) e com qualquer assistente de IA, não só Claude
Code. Por isso, todo `SKILL.md` continua sendo um arquivo markdown simples e autocontido —
qualquer IA consegue segui-lo se for instruída a "ler e seguir as instruções do arquivo em
`<caminho>`". Dois modos de uso coexistem, sem exigir escolha do usuário:

- **Caminho explícito (universal):** o usuário aponta o caminho da pasta (ex.: `sdd/` na raiz do
  projeto) e pede para a IA seguir as instruções do arquivo de configuração ou de uma skill
  específica. Funciona em qualquer ferramenta.
- **Descoberta automática (Claude Code):** se o Claude Code está em uso, `configurar-sdd`
  verifica se as skills já estão presentes em `.claude/skills/`; se não estiverem (por exemplo,
  o usuário só colocou a pasta `sdd/` na raiz), oferece copiá-las para lá como parte da própria
  configuração, para que as chamadas seguintes aconteçam por linguagem natural, sem precisar
  repetir o caminho toda vez.

## Não-objetivos

- Não vamos criar um sistema de plugin/marketplace do Claude Code para o bundle — ele continua
  sendo uma pasta simples que se copia para `.claude/skills/` do projeto novo.
- Não vamos migrar o conteúdo para inglês (decisão do usuário: manter português).
- Não vamos automatizar a detecção de quais MCPs estão conectados (Linear/Figma) — a skill de
  adaptação pergunta isso, pois não há como inspecionar isso de forma confiável a partir do
  repositório.

## Arquitetura da solução

Três camadas com responsabilidades separadas:

| Camada | O quê | Onde vive | Quem escreve |
| --- | --- | --- | --- |
| **Skills (processo)** | As 8 `SKILL.md` (7 atuais generalizadas + `configurar-sdd` nova) | `.claude/skills/<nome>/` no projeto novo (copiadas do bundle) | Mantenedor do template (você) |
| **Baselines compartilhadas** | Checklists universais fixos de segurança e qualidade | `.claude/skills/_shared/` no projeto novo (copiada do bundle) | Mantenedor do template (você) |
| **Configuração do projeto** | Stack, comandos, integrações, limites, extensões — os únicos dados realmente específicos de cada projeto | `sdd.config.md` **na raiz do bundle** (irmão de `_shared/` e das pastas de skill), onde quer que o bundle esteja | Gerado/atualizado pela skill `configurar-sdd` |

> **Convenção de caminho:** o bundle inteiro se move como uma unidade — seja colocado na raiz
> do projeto (`sdd/`) ou copiado para `.claude/skills/`. `sdd.config.md` sempre fica na raiz
> desse bundle, ao lado das pastas de skill e de `_shared/`. Cada skill referencia o config como
> `../sdd.config.md` (um nível acima da própria pasta) — a referência funciona igual nos dois
> locais possíveis, sem precisar saber onde o bundle está de fato.

As skills e as baselines são o "template" que se copia igual para qualquer projeto. O arquivo de
configuração é o único artefato que muda de projeto para projeto, e é ele que faz a mesma pasta
de skills "funcionar" tanto num app Flutter quanto num projeto React/JavaScript.

### Inventário de arquivos

**Novos:**

- `configurar-sdd/SKILL.md`
- `configurar-sdd/references/CONFIG_TEMPLATE.md` (esqueleto vazio do `sdd.config.md`, com
  comentários explicando cada campo)
- `_shared/SECURITY_BASELINE.md`
- `_shared/QUALITY_BASELINE.md`
- `README.md` na raiz do bundle — instruções de uso para humanos: onde colocar a pasta, como
  acionar a configuração nos dois modos de invocação, e o que cada skill faz

**Generalizados (reescrita do conteúdo específico de Flutter, mantendo a estrutura e o
propósito de cada arquivo):**

- `criar-prd/SKILL.md` + `criar-prd/references/TEMPLATE.md`
- `criar-techspec/SKILL.md` + `criar-techspec/references/TEMPLATE.md`
- `criar-tasks/SKILL.md` + `criar-tasks/references/TASKS_TEMPLATE.md` + `TASK_TEMPLATE.md`
- `executar-task/SKILL.md`
- `executar-bugfix/SKILL.md` + `executar-bugfix/references/TEMPLATE.md`
- `executar-qa/SKILL.md` + `executar-qa/references/TEMPLATE.md`
- `executar-review/SKILL.md` + `executar-review/references/TEMPLATE.md`

## Esquema de `sdd.config.md`

Arquivo único, no mesmo estilo de tags já usado nas skills atuais (`<contexto_projeto>`,
`<comandos>`), para ficar consistente com o que as skills já sabem ler.

```md
# Configuração do SDD para este projeto

<stack_projeto>
Linguagem(ns): [ex: JavaScript/TypeScript]
Framework principal: [ex: React 18 + Vite]
Padrão arquitetural detectado: [ex: feature-first com hooks + context, sem Clean Architecture formal]
Gerenciador de pacotes: [ex: pnpm]
Camadas/ordem de dependência (para sequenciar tasks): [ex: lib/utils → hooks → components → pages → rotas]
</stack_projeto>

<comandos_projeto>
Lint: [comando detectado ou definido]
Format: [comando]
Testes: [comando]
Cobertura: [comando] — threshold: [%, ou "sem gate definido"]
E2E (se houver): [comando]
Build: [comando]
Gate agregado (equivalente a "make ci"): [comando, se existir]
</comandos_projeto>

<integracoes_projeto>
Rastreador de issues: [Linear / Jira / GitHub Issues / nenhum]
Ferramenta de design: [Figma / nenhuma]
Base de conhecimento externa para documentação estratégica: [Obsidian / Notion / nenhuma]
Outras integrações relevantes: [...]
</integracoes_projeto>

<ui_projeto>
Tem interface de usuário final? [sim/não]
Tipo: [web / mobile / desktop / não aplicável]
Ferramenta de validação visual: [ex: Playwright + screenshot / golden test / não aplicável]
Acessibilidade aplicável: [sim/não — e qual padrão, ex: WCAG 2.1 AA]
i18n aplicável: [sim/não]
</ui_projeto>

<limites_projeto>
Áreas read-only ou fora de escopo (equivalente ao `backend/` read-only do projeto original): [...]
Regras invioláveis descobertas em CLAUDE.md/AGENTS.md/rules.md do projeto: [...]
</limites_projeto>

<seguranca_extensoes>
[Regras de segurança específicas deste projeto, além do _shared/SECURITY_BASELINE.md —
ex: "tokens sempre em Keychain/Keystore", "compliance LGPD para dados de usuário", etc.]
</seguranca_extensoes>

<qualidade_extensoes>
[Convenções de nomenclatura/estilo específicas da linguagem/projeto, além do
_shared/QUALITY_BASELINE.md — ex: convenções de naming do ESLint config detectado]
</qualidade_extensoes>

<colaboracao_projeto>
Convenção de commit: [ex: Conventional Commits / livre / detectado via commitlint]
Convenção de branch: [ex: feature/<slug>, ou "sem convenção formal"]
Processo de PR: [ex: exige template preenchido, exige review de X pessoas, squash merge]
Política de comentários no código: [ex: só onde a intenção não é óbvia / exigir comentário em toda função pública / nenhuma regra]
</colaboracao_projeto>

<documentacao_projeto>
Onde a documentação de cada feature deve ser salva: [ex: docs/<feature>/, ou "não aplicável"]
Documentação dupla exigida (registro externo)? [sim → onde / não]
</documentacao_projeto>

<artefatos_sdd>
Pasta raiz dos artefatos por feature: [padrão: tasks/prd-<nome-da-feature>/]
Arquivos por feature: prd.md · techspec.md · tasks.md · <num>_task.md · bugs.md · bugfixes.md · qa.md · codereview.md · evidences/
</artefatos_sdd>
```

## Skill nova: `configurar-sdd`

**Frontmatter `description`** cobre frases de gatilho como: "adapta esse SDD pra esse
projeto", "configura o SDD aqui", "entende esse projeto e adapta as skills", "setup do SDD
neste repositório" — para ativar por intenção, sem exigir um comando especial.

**Fluxo:**

1. **Ler regras existentes do projeto (obrigatório):** procurar e ler `CLAUDE.md`, `AGENTS.md`,
   `rules.md`, `.cursorrules`, `CONTRIBUTING.md` ou equivalente, em qualquer nível razoável do
   repositório. Essas regras alimentam `<limites_projeto>` e `<seguranca_extensoes>`.
2. **Explorar o projeto (obrigatório, antes de perguntar):**
   - Manifestos de pacote (`package.json`, `pubspec.yaml`, `go.mod`, `requirements.txt`,
     `pyproject.toml`, `Cargo.toml`, `pom.xml`, `*.csproj`, etc.) → linguagem, framework,
     dependências.
   - Estrutura de pastas e arquivos existentes → padrão arquitetural provável.
   - Scripts em `package.json`/`Makefile`/CI (`.github/workflows/`, `.gitlab-ci.yml`) → comandos
     reais de lint/test/coverage/build e threshold já configurado.
   - Testes existentes → framework de teste e convenções de nomenclatura em uso.
   - Presença de pastas tipo `web/`, `mobile/`, `apps/*` → indício de tipo de UI.
3. **Inferir e classificar confiança:** para cada campo do config, marcar como "detectado com
   confiança" ou "incerto".
4. **Perguntar só os gaps (obrigatório, uma pergunta de cada vez ou agrupada por tema, preferindo
   múltipla escolha):**
   - Confirmar/corrigir o padrão arquitetural quando incerto ou ausente.
   - Rastreador de issues em uso (Linear/Jira/GitHub Issues/nenhum) e ferramenta de design
     (Figma/nenhuma) — não é detectável a partir do repositório.
   - Threshold de cobertura, se não detectado, sugerindo um valor razoável (ex.: 80%) para
     confirmação.
   - Requisitos de compliance/segurança específicos (dados pessoais, setor regulado, etc.).
   - Onde a documentação de feature deve viver e se existe um registro externo obrigatório.
   - Convenções de colaboração não detectáveis com confiança: padrão de commit, convenção de
     branch/PR, política de comentários (tentar detectar via `commitlint`, template de PR,
     `.editorconfig`/lint de comentários antes de perguntar).
5. **Gravar `sdd.config.md` na raiz do bundle** (ao lado das pastas de skill, não dentro de
   `.claude/`) seguindo o esquema acima. Se o arquivo já existir, atualiza por seção em vez de
   sobrescrever tudo (permite reconfiguração incremental).
6. **Disponibilizar as skills para descoberta automática (Claude Code):** se o projeto usa Claude
   Code e as skills ainda não estão em `.claude/skills/`, oferecer copiá-las para lá (mantendo o
   bundle original intacto onde estiver). Se o usuário preferir não copiar, ou a ferramenta não
   for Claude Code, seguir funcionando por referência explícita de caminho — nenhuma etapa fica
   bloqueada por isso.
7. **Relatar** ao usuário um resumo do que foi detectado vs. perguntado, o caminho do arquivo de
   configuração gerado, e se as skills foram (ou não) copiadas para `.claude/skills/`.

<critical>Nunca prossiga para gerar o config sem ter tentado explorar o projeto primeiro — perguntar
antes de explorar é o erro que este design existe para evitar.</critical>

## Baselines compartilhadas (`_shared/`)

### `SECURITY_BASELINE.md`

Checklist fixo, agnóstico de stack, referenciado por `criar-techspec`, `executar-task`,
`executar-bugfix`, `executar-qa` e `executar-review`:

- Nenhum segredo/credencial hardcoded no código ou versionado (chaves, tokens, senhas,
  connection strings).
- Dependências com vulnerabilidades conhecidas: rodar o scanner do stack detectado (`npm audit`,
  `pip-audit`, `govulncheck`, `cargo audit`, `bundler-audit`, etc.) e tratar achados de
  severidade alta/crítica como bloqueantes.
- Classes de injeção cobertas (SQL/NoSQL, comando de shell, XSS, SSRF, path traversal):
  entrada validada/sanitizada, saída codificada corretamente para o contexto.
- AuthN/authZ aplicados na camada correta (nunca só no client); nenhuma rota/endpoint sensível
  sem verificação de permissão.
- Dados sensíveis: nunca logados em claro (PII, tokens, senhas), armazenamento adequado
  (secure storage/keychain conforme a plataforma), transporte sempre criptografado.
- Configuração segura por padrão: CORS restrito, headers de segurança, modo debug/verbose
  desligado em produção, princípio do menor privilégio.
- Toda extensão específica de projeto (ex.: LGPD, storage de token, deep link) entra via
  `<seguranca_extensoes>` do config, nunca reescrevendo este arquivo.

### `QUALITY_BASELINE.md`

Checklist fixo de qualidade de código, agnóstico de stack:

- Complexidade: funções/métodos curtos e com responsabilidade única; extrair quando crescem.
- DRY: sem duplicação evitável; reuso do que já existe no projeto antes de criar algo novo.
- Naming: nomes claros e descritivos, seguindo a convenção da linguagem/projeto (detectada via
  linter/config existente, complementada por `<qualidade_extensoes>`).
- Tratamento de erro: erros tipados/tratados explicitamente, nunca engolidos silenciosamente.
- Comentários: só onde necessário; sem código morto nem TODO órfão.
- Performance: sem trabalho redundante óbvio em hot paths; recursos liberados corretamente.
- Testes: cobrem comportamento (não implementação), caminho feliz **e** falha, sem asserção
  fraca isolada (`isNotNull`, `expect(true)`).
- Gate de cobertura, quando existir, é absoluto — nunca se reduz o threshold para "passar".

## Generalização das skills e templates existentes

Padrão aplicado a todas as 7 skills:

1. Adicionar `<config>`../sdd.config.md`</config>` no topo, junto dos outros parâmetros (caminho
   relativo — um nível acima da própria pasta da skill, onde quer que o bundle esteja).
2. Trocar todo `<contexto_projeto>` fixo por instrução para ler o config e agir de acordo.
3. Se `sdd.config.md` não existir: avisar o usuário e sugerir rodar `configurar-sdd` antes de
   continuar (guarda leve, não é um mini-setup embutido).
4. Trocar blocos fixos de Linear/Figma/Obsidian por condicionais lidos de
   `<integracoes_projeto>`.
5. Trocar comandos fixos (`make ci`, `flutter analyze`, etc.) pelos comandos de
   `<comandos_projeto>`.
6. Referenciar `_shared/SECURITY_BASELINE.md` e `_shared/QUALITY_BASELINE.md` em vez de
   checklists de segurança/qualidade fixos em Dart.

Ajustes específicos por arquivo:

- **`criar-prd` + TEMPLATE:** "Plataformas alvo" (fixo em mobile) vira seção condicional por tipo
  de projeto lida em `<ui_projeto>` (mobile: permissões/offline/deep link; web: navegadores/
  responsividade; API: versionamento/rate limit — quando não há UI, a seção é omitida).
  "Referências de design (Figma)" só aparece se `<integracoes_projeto>` tiver Figma.
  "Rastreabilidade" cita o rastreador de issues configurado, não Linear fixo.
- **`criar-techspec` + TEMPLATE:** a tabela "Mapeamento Clean Architecture" com paths Dart
  literais vira uma tabela de camadas genérica, preenchida a partir de `<stack_projeto>`. A
  seção "Abordagem de testes" deixa de ser fixa em unitário/BLoC/widget/golden/E2E e usa os
  níveis de teste reais do stack (ex.: unitário/componente/integração/E2E para um projeto
  React). "Meta de cobertura" usa o threshold do config. "Design e tokens (Figma)" e
  "Documentação da feature" ficam condicionais.
- **`criar-tasks` + templates:** "Ordem por camada (Clean Architecture)" vira "ordem por
  dependência de camada, conforme `<stack_projeto>`" — sempre dependências antes de
  dependentes, mas sem impor nomes de camada Flutter. TDD (teste antes da implementação)
  continua universal e obrigatório. Golden test antes da página só entra se `<ui_projeto>`
  indicar teste visual aplicável.
- **`executar-task`:** troca comandos e regras de camada fixas pelas do config; mantém TDD e a
  auditoria de testes antes de escrever/alterar testes como princípio universal.
- **`executar-bugfix` + TEMPLATE:** mantém como universais "causa raiz, nunca sintoma" e "teste
  de regressão falhando primeiro"; a seção de validação visual fica condicional a
  `<ui_projeto>`; "se a causa raiz está fora do escopo (read-only)" generaliza o antigo
  `backend/` usando `<limites_projeto>`.
- **`executar-qa` + TEMPLATE:** "não use automação de browser, é mobile" vira condicional —
  browser automation (Playwright/Cypress) é a ferramenta correta quando `<ui_projeto>` = web;
  device/golden quando mobile; QA de contrato/API quando não há UI. Acessibilidade só entra
  quando aplicável.
- **`executar-review` + TEMPLATE:** troca as regras de camada Flutter fixas (Presentation ⇏
  Data, DI por construtor Modular, `mocktail`) pelas equivalentes lidas do config/stack
  detectado; troca "documentação dupla no Obsidian" por condicional de
  `<documentacao_projeto>`.

## Fluxo de uso ponta a ponta (exemplo do usuário)

1. Usuário inicia um projeto novo em React + JavaScript.
2. Coloca a pasta `sdd/` (este bundle, como está) na raiz do projeto novo — não precisa
   movê-la para dentro de `.claude/skills/` antes.
3. Aponta a IA escolhida para o caminho da pasta e pede algo como "configura o SDD nesse
   projeto" (funciona igual em Claude Code, Codex ou qualquer outra ferramenta que consiga ler
   arquivos e seguir instruções).
4. `configurar-sdd` lê `package.json`, detecta React + Vite + Jest/Vitest, detecta ausência de
   Clean Architecture formal (estrutura por feature simples), detecta script de coverage no
   `package.json` sem threshold definido, não encontra `CLAUDE.md`/`rules.md`, não encontra
   `commitlint` nem template de PR.
5. Pergunta: qual rastreador de issues (se algum), se há ferramenta de design, sugere 80% de
   cobertura como padrão e pede confirmação, confirma que é um projeto web com UI, pergunta a
   convenção de commit/branch/PR desejada.
6. Grava `sdd/sdd.config.md` (raiz do bundle, ao lado das pastas de skill). Se estiver rodando
   dentro do Claude Code, oferece copiar as 8 skills (com o `sdd.config.md` já preenchido) para
   `.claude/skills/` para uso por linguagem natural daí em diante — o config continua na raiz
   dessa cópia, `.claude/skills/sdd.config.md`.
7. Usuário chama `criar-prd` — por linguagem natural (se copiado para `.claude/skills/`) ou
   apontando o caminho de novo (`sdd/criar-prd/SKILL.md`) — e a skill já se comporta como um
   projeto React web, sem nenhuma menção a Flutter.

## Validação

Como este é um bundle de prompts/templates (não há código para rodar testes automatizados),
a validação é por inspeção e simulação:

1. **Simulação a seco:** aplicar mentalmente o fluxo de `configurar-sdd` contra 2 projetos
   fictícios de perfis bem diferentes (ex.: uma API backend em Python sem UI, e o projeto React
   citado pelo usuário) e verificar que o config gerado faz sentido nos dois.
2. **Revisão cruzada:** para cada uma das 7 skills generalizadas, conferir que nenhuma menção
   remanescente a Flutter/Dart/BLoC/Modular/golden/Linear/Figma/Obsidian ficou hard-coded fora
   de um bloco condicional (`grep` pelos termos antigos no bundle inteiro deve retornar zero
   fora de `_shared/` e exemplos ilustrativos claramente marcados como exemplo).
3. **Checklist de paridade:** todo critical rule/checklist item que existia na versão Flutter
   tem um equivalente genérico ou condicional na nova versão — nada foi perdido, só
   generalizado.

## Riscos e premissas

- **Premissa:** o usuário mantém este bundle (`/Users/felipe/Desktop/sdd/`) como o template
  "mestre" e sempre copia uma cópia nova para cada projeto — o processo de adaptação nunca
  reescreve o bundle original.
- **Risco:** projetos muito atípicos (ex.: monorepo poliglota) podem exigir mais de uma rodada
  de `configurar-sdd` ou ajuste manual do config — aceitável, o arquivo é editável à mão.
- **Risco:** se o usuário editar `sdd.config.md` à mão e depois rodar `configurar-sdd`
  de novo, a skill precisa mesclar por seção (não sobrescrever tudo) para não perder edições
  manuais — tratado no passo 5 do fluxo de bootstrap.
