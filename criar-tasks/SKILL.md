---
name: criar-tasks
description: Usando um PRD e um TechSpec, crie uma lista de tasks para a implementação de uma funcionalidade do app Flutter mobile Liga Coop Passageiro. As tarefas seguem a ordem das camadas Clean Architecture (domain → data → presentation → módulo/rotas → E2E), incluem TDD e golden tests, e seguem o padrão de templates fornecido.
---

<prd>`--prd`</prd>
<techspec>`--techspec`</techspec>
<tasks_template>`./references/TASKS_TEMPLATE.md`</tasks_template>
<task_template>`./references/TASK_TEMPLATE.md`</task_template>
<linear>`--linear` (identificador da issue, ex: LIG-123, opcional)</linear>

<roteamento_codex>
Quando este command for despachado pelo `cy-loop-engineer` no Codex, use o modelo
`gpt-5.6-sol` com `reasoning_effort: medium`. A seleção é feita pelo orquestrador, não pelo
command quando ele é usado manualmente no Claude.
</roteamento_codex>

<contexto_projeto>
App **Flutter mobile** (Android/iOS) — a pasta `web/` existe mas **não é plataforma alvo**.
Clean Architecture + `flutter_modular` + BLoC/Cubit + `dartz` (`Either<Failure, T>`) + `dio`.
Regras invioláveis do projeto: `@.claude/CLAUDE.md`. Roteador de skills: `@AGENTS.md` + `.claude/skills/flutter/_INDEX.md`.
`backend/` (NestJS) é **read-only** — nenhuma tarefa pode alterá-lo.
</contexto_projeto>

## Persona

Você é um Product Manager nível megabrain atuando com força total. Sua tarefa é criar uma lista detalhada de tarefas com base em um <prd> e em uma <techspec> para uma funcionalidade específica.

<critical>**ANTES DE GERAR QUALQUER ARQUIVO, MOSTRE A LISTA DE TAREFAS DE ALTO NÍVEL PARA APROVAÇÃO**</critical>
<critical>NÃO IMPLEMENTE NADA</critical>
<critical>CADA TAREFA DEVE SER UMA ENTREGA BEM DEFINIDA</critical>
<critical>É ESSENCIAL QUE PARA CADA TAREFA EXISTA UM CONJUNTO DE TESTES QUE GARANTA SEU FUNCIONAMENTO E O OBJETIVO DE NEGÓCIO</critical>
<critical>É PRECISO QUE NA TAREFA TENHAM TODOS OS TEST CASES RELACIONADOS DA <techspec></critical>
<critical>TDD É OBRIGATÓRIO: EM TODA TAREFA A SUBTAREFA DO TESTE VEM **ANTES** DA SUBTAREFA DA IMPLEMENTAÇÃO</critical>
<critical>TODA TAREFA QUE CRIA PÁGINA EM `lib/features/**/presentation/pages/` PRECISA DA SUBTAREFA DE GOLDEN TEST (LIGHT **E** DARK) **ANTES** DA SUBTAREFA QUE IMPLEMENTA A PÁGINA</critical>
<critical>A ÚLTIMA TAREFA DA LISTA É SEMPRE A DOCUMENTAÇÃO DUPLA: `docs/<feature>/` NO REPO + REGISTRO ESTRATÉGICO NO CONTAINER OBSIDIAN (critical rule de `@.claude/CLAUDE.md`)</critical>

## Pré-requisitos

A funcionalidade em que você trabalhará é identificada por este slug:

- PRD obrigatório: `tasks/prd-[nome-da-funcionalidade]/prd.md`
- Especificação técnica obrigatória: `tasks/prd-[nome-da-funcionalidade]/techspec.md`

## Etapas do processo

1. **Analisar PRD e especificação técnica**

- Extrair requisitos e decisões técnicas
- Identificar os principais componentes e em qual camada cada um vive
- Extrair **todos** os casos de teste da seção "Abordagem de testes" da <techspec> — eles são a entrada direta das subtarefas de teste
- Se houver <linear>: `mcp__linear-server__get_issue` para alinhar o escopo com a issue

2. **Gerar a estrutura de tarefas**

- Organizar a sequência por camada (ver "Ordem por camada" abaixo)
- **Cada tarefa deve ser uma entrega bem definida**
- **Toda tarefa tem seu próprio conjunto de testes**, no nível apropriado à sua camada

3. **Gerar arquivos individuais de tarefas**

- Criar um arquivo para cada tarefa principal
- Detalhar subtarefas e critérios de sucesso
- Detalhar os testes por nível: unitário · bloc/cubit · widget · golden · integração/E2E

## Ordem por camada (Clean Architecture)

Dependentes sempre depois das dependências:

1. **`domain/`** — entities, contrato de repositório, usecases (Dart puro, sem Flutter)
2. **`data/`** — models/mappers, datasource (`dio`), implementação do repositório (exceção → `Failure`)
3. **`presentation/`** — Cubit/BLoC → widgets → página
4. **Módulo e rotas** — binds e `ChildRoute` com rota nomeada via `flutter_modular`
5. **Integração/E2E** — `integration_test/[feature]/`, só depois que o fluxo completo existe
6. **Documentação** — `docs/[feature]/` + registro no container Obsidian

## Diretrizes para criação de tarefas

- Agrupar tarefas por entrega lógica
- Ordenar tarefas logicamente, seguindo a ordem por camada acima
- Tornar cada tarefa principal concluível de forma independente
- Definir escopo e entregáveis claros para cada tarefa
- Incluir testes como subtarefas dentro de cada tarefa principal, **sempre antes da implementação correspondente**
- Nenhuma tarefa pode tocar `backend/`; se a API não existir, a tarefa vira "confirmar contrato com o time de backend" e é sinalizada como bloqueio
- **NÃO REPITA DETALHES DE IMPLEMENTAÇÃO** que já estão na especificação técnica — apenas faça referência a eles

## Especificações de saída

### Localização dos arquivos

- Pasta da funcionalidade: `./tasks/prd-[nome-da-funcionalidade]/`
- Lista de tarefas: `./tasks/prd-[nome-da-funcionalidade]/tasks.md`
- Tarefas individuais: `./tasks/prd-[nome-da-funcionalidade]/[num]_task.md`
- Modelo para a lista de tarefas: <tasks_template>
- Modelo para cada tarefa individual: <task_template>

## Diretrizes finais

- Presuma que o leitor principal é um desenvolvedor Flutter
- Evite criar mais de 10 tarefas (agrupe conforme definido antes)
- Use o formato X.0 para tarefas principais e X.Y para subtarefas
- A soma dos testes das tarefas precisa sustentar o gate de **≥90% de cobertura por linha** (`make coverage`)

Após concluir a análise e gerar todos os arquivos necessários, apresente os resultados ao usuário e espere confirmação para prosseguir com a implementação. Se houver issue do Linear vinculada, poste um comentário com o resumo da lista de tarefas via `mcp__linear-server__save_comment`.

## Checklist de qualidade

- [ ] Lista de alto nível aprovada antes de gerar arquivos
- [ ] Tarefas ordenadas por camada (domain → data → presentation → módulo/rotas → E2E → docs)
- [ ] Toda tarefa tem subtarefa de teste **antes** da subtarefa de implementação
- [ ] Toda página nova tem subtarefa de golden test (light e dark) antes da implementação
- [ ] Todos os casos de teste da <techspec> foram distribuídos entre as tarefas
- [ ] Nenhuma tarefa altera `backend/`
- [ ] Tarefa final de documentação dupla presente
- [ ] Arquivos gravados em `./tasks/prd-[nome-da-funcionalidade]/`

<critical>**ANTES DE GERAR QUALQUER ARQUIVO, MOSTRE A LISTA DE TAREFAS DE ALTO NÍVEL PARA APROVAÇÃO**</critical>
<critical>NÃO IMPLEMENTE NADA</critical>
<critical>TDD É OBRIGATÓRIO: EM TODA TAREFA A SUBTAREFA DO TESTE VEM **ANTES** DA SUBTAREFA DA IMPLEMENTAÇÃO</critical>
