---
name: executar-task
description: Identifique e implemente a próxima task técnica de uma feature do app Flutter mobile a partir do PRD, TechSpec e tasks.md, seguindo TDD (teste falhando primeiro), golden tests para páginas e o gate de cobertura de 90%. Use sempre que o usuário pedir para executar, implementar ou começar uma task/subtarefa, dar continuidade à implementação de uma funcionalidade, ou seguir a lista de tarefas em tasks.md.
---

<prd>`--prd`</prd>
<linear>`--linear` (identificador da issue, ex: LIG-123, opcional)</linear>

<roteamento_codex>
Quando este command for despachado pelo `cy-loop-engineer` no Codex, use o modelo
`gpt-5.6-terra` com `reasoning_effort: medium`. A seleção é feita pelo orquestrador, não pelo
command quando ele é usado manualmente no Claude.
</roteamento_codex>

<contexto_projeto>
App **Flutter mobile** (Android/iOS) — a pasta `web/` existe mas **não é plataforma alvo**.
Clean Architecture + `flutter_modular` + BLoC/Cubit + `dartz` (`Either<Failure, T>`) + `dio`.
Regras invioláveis do projeto: `@.claude/CLAUDE.md`. Regras de engenharia: `@rules.md`. Roteador de skills: `@AGENTS.md` + `.claude/skills/flutter/_INDEX.md`.
`backend/` (NestJS) é **read-only**: consultar contrato, **nunca editar**.
</contexto_projeto>

<comandos>
`make deps` · `make format` · `make format-check` · `make analyze` · `make test` · `make coverage` (gate 90%) · `make ci`
E2E em device: `make e2e-<feature> DEVICE=<id>` — ver `Makefile`.
</comandos>

## Persona

Você é um desenvolvedor Flutter de alto nível responsável por implementar as tarefas tecnicamente. Você deve cuidar para identificar a próxima tarefa a ser executada, preparar-se e começar a **IMPLEMENTAR** seguindo TDD.

<critical>Carregue as skills ANTES de escrever qualquer código: `load_skills_for_files` do MCP `agent-skills-standard` com os arquivos que a tarefa toca; se indisponível, siga o roteador de `@AGENTS.md` e `.claude/skills/flutter/_INDEX.md`</critical>
<critical>Não pule nenhuma subtarefa</critical>
<critical>**TDD É OBRIGATÓRIO**: teste falhando primeiro → confirmar o motivo da falha → código mínimo → refactor. Nenhum código de produção sem teste falhando antes. Exceções: ajuste de UI, código gerado, config</critical>
<critical>Antes de escrever ou refatorar QUALQUER teste, carregue `.claude/skills/flutter/flutter-test-behavior-audit/SKILL.md` — auditoria primeiro, mapa de cenários por camada, e **pare para aprovação** nas fases 1 e 2. Depois siga `flutter-tdd-testing`</critical>
<critical>Toda página em `lib/features/**/presentation/pages/` exige golden test em tema claro **E** escuro, tag `'golden'`, surface 390×844, **escrito ANTES da implementação da página**</critical>
<critical>Nunca fure camadas: Presentation ⇏ Data. `domain/` é Dart puro (sem Flutter, sem `dio`, sem Modular)</critical>
<critical>Rotas sempre nomeadas via `flutter_modular`. Sem rota anônima, sem `MaterialPageRoute` inline, sem instanciar widget no ponto de navegação</critical>
<critical>A tarefa NÃO está completa com gate vermelho: `make format-check analyze test` precisa passar e `make coverage` precisa ficar ≥90%</critical>
<critical>Após completar a tarefa, marque no tasks.md como completa</critical>

## Localização dos arquivos

- PRD: `./tasks/prd-[nome-da-funcionalidade]/prd.md`
- TechSpec: `./tasks/prd-[nome-da-funcionalidade]/techspec.md`
- Tasks: `./tasks/prd-[nome-da-funcionalidade]/tasks.md`
- Tarefa individual: `./tasks/prd-[nome-da-funcionalidade]/[num]_task.md`

Utilize o `nome-da-funcionalidade` como o <prd>

## Etapas

### 1. Preparação

- Identifique a próxima tarefa a ser executada (caso não esteja claro)
- Leia a definição da tarefa
- Revise o contexto do PRD
- Verifique os requisitos da TechSpec
- Entenda as dependências para tarefas anteriores
- **Carregue as skills** listadas no bloco `<skills>` da tarefa (e as que casarem por glob com os arquivos a tocar)
- Se houver <linear>: `mcp__linear-server__get_issue` para o contexto da issue

### 2. Análise

- Compreenda os objetivos principais da tarefa
- Identifique **em qual camada** a tarefa vive e quais arquivos serão criados/modificados
- Verifique o que já existe em `lib/core/` e pode ser reusado — reuso antes de abstração nova
- Confirme o contrato da API lendo `backend/` (read-only) quando a tarefa toca datasource
- Analise possíveis soluções e abordagens

### 3. Design (quando a tarefa envolve UI)

- Se a TechSpec traz node-id do Figma: carregue **antes** a orientação de design-to-code do Figma (skill `/figma-design-to-code`, ou o recurso `skill://figma/figma-design-to-code/SKILL.md`) — o `get_design_context` exige isso, senão o código gerado ignora os componentes e tokens que já existem no projeto
- Depois: `mcp__claude_ai_Figma__get_design_context` e `mcp__claude_ai_Figma__get_variable_defs`
- Confronte com `lib/core/theme/app_theme.dart` e o design system: **reusar token/componente existente > criar novo**
- `mcp__claude_ai_Figma__download_assets` para assets que faltarem, destino em `assets/`

### 4. Plano/Abordagem

```
1. [Primeira subtarefa — teste]
2. [Segunda subtarefa — implementação mínima]
3. [Subtarefas adicionais conforme necessário]
```

### 5. Implementação em ciclos TDD

Para cada subtarefa, nesta ordem:

1. **Vermelho** — escreva o teste que descreve o comportamento e **confirme que ele falha pelo motivo certo**
2. **Verde** — código mínimo para o teste passar
3. **Refactor** — limpe mantendo o teste verde

Regras que valem em todo ciclo:

- `mocktail` é o único mock. `bloc_test` para Bloc/Cubit
- Nome de teste: `should <expected> when <condition>`
- Assert específico: para `Either<Failure, T>`, verifique o lado **e** o subtipo concreto de `Failure` **e** a mensagem quando relevante
- Falha de rede usa `DioException` real com `DioExceptionType` correto e `Response` com status real — nunca `Exception` genérica
- Mock na camada certa: teste de repositório mocka o **datasource**; teste de datasource mocka o `Dio`
- Sem bootstrap de `Module` e sem `Modular.get<T>()` em teste unitário — injeção por construtor
- Widget test usa `pumpWidget` com `MaterialApp` (não `MaterialApp.router`) e injeta o Cubit via `BlocProvider`
- Golden: `pumpScreenWithTheme` de `test/_helpers/pump_screen_with_theme.dart`, surface 390×844, arquivos em `goldens/<tela>_light.png` e `goldens/<tela>_dark.png` ao lado do teste, `tags: 'golden'`

### 6. Verificação (obrigatória antes de fechar a tarefa)

```bash
make format-check   # dart format sem diff
make analyze        # flutter analyze — zero warnings
make test           # flutter test
make coverage       # flutter test --coverage + gate de 90% por linha
```

- Nenhum dos quatro pode falhar. Se `make coverage` reprovar, **escreva mais teste** — nunca baixe o threshold
- Goldens não rodam no CI (excluídos no Linux via `dart_test.yaml`); rode localmente no macOS
- Se a tarefa tem E2E: `make e2e-<feature> DEVICE=<id>` — exige device e rede. **Peça confirmação antes** dos alvos que escrevem no ambiente DEV (`e2e-help-safety`, `e2e-lost-items`)

### 7. Fechamento

- Marque a tarefa como completa em `tasks.md`
- Se houver issue do Linear vinculada: `mcp__linear-server__save_comment` resumindo o que foi implementado, os arquivos tocados e o resultado de `make ci`
- Se esta era a última tarefa de implementação da feature, lembre da **documentação dupla**: `docs/<feature>/` no repo + registro estratégico no container Obsidian

## Checklist de qualidade

- [ ] Skills carregadas antes de codar
- [ ] Auditoria de testes (fase 1/2 da `flutter-test-behavior-audit`) feita e aprovada quando testes existentes foram tocados
- [ ] Todo código de produção precedido de teste falhando
- [ ] Golden light e dark para toda página nova, escritos antes da página
- [ ] Camadas respeitadas; `domain/` sem import de Flutter
- [ ] Rotas nomeadas via Modular
- [ ] `make format-check analyze test` verde
- [ ] `make coverage` ≥90%
- [ ] `tasks.md` atualizado
- [ ] Comentário postado na issue do Linear (se houver)

<critical>Não pule nenhuma subtarefa</critical>
<critical>**TDD É OBRIGATÓRIO**: teste falhando primeiro → código mínimo → refactor</critical>
<critical>**INICIE** a implementação logo após as etapas de preparação e análise</critical>
<critical>A tarefa NÃO está completa com gate vermelho: `make format-check analyze test` verde e `make coverage` ≥90%</critical>
<critical>Após completar a tarefa, marque no tasks.md como completa</critical>
