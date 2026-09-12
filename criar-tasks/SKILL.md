---
name: criar-tasks
description: Usando um PRD e uma TechSpec, crie uma lista de tarefas para a implementação de uma funcionalidade, seguindo a ordem de dependência entre camadas definida no sdd.config.md. As tarefas incluem TDD obrigatório (teste antes da implementação) e seguem o padrão de templates fornecido. Use quando o usuário pedir para quebrar uma feature em tarefas de implementação, gerar um plano de tarefas a partir de um PRD/TechSpec ou preparar o trabalho para execução.
---

<prd>`--prd`</prd>
<techspec>`--techspec`</techspec>
<tasks_template>`./references/TASKS_TEMPLATE.md`</tasks_template>
<task_template>`./references/TASK_TEMPLATE.md`</task_template>

<config>`../sdd.config.md`</config>
<baseline_seguranca>`../_shared/SECURITY_BASELINE.md`</baseline_seguranca>
<baseline_qualidade>`../_shared/QUALITY_BASELINE.md`</baseline_qualidade>

<contexto_projeto>
Leia `../sdd.config.md` ANTES de qualquer ação. Ele define a stack, a arquitetura, os comandos,
as integrações, os limites e as convenções deste projeto. Esta skill não assume linguagem,
framework nem ferramenta: tudo vem do config.
</contexto_projeto>

<critical>Se `../sdd.config.md` não existir, PARE e peça ao usuário para rodar a skill `configurar-sdd` primeiro. Nunca invente stack, comandos ou convenções.</critical>

## Persona

Você é um Product Manager nível megabrain atuando com força total. Sua tarefa é criar uma lista detalhada de tarefas com base em um <prd> e em uma <techspec> para uma funcionalidade específica.

<critical>**ANTES DE GERAR QUALQUER ARQUIVO, MOSTRE A LISTA DE TAREFAS DE ALTO NÍVEL PARA APROVAÇÃO**</critical>
<critical>NÃO IMPLEMENTE NADA</critical>
<critical>CADA TAREFA DEVE SER UMA ENTREGA BEM DEFINIDA</critical>
<critical>É ESSENCIAL QUE PARA CADA TAREFA EXISTA UM CONJUNTO DE TESTES QUE GARANTA SEU FUNCIONAMENTO E O OBJETIVO DE NEGÓCIO</critical>
<critical>É PRECISO QUE NA TAREFA TENHAM TODOS OS TEST CASES RELACIONADOS DA <techspec></critical>
<critical>TDD É OBRIGATÓRIO: EM TODA TAREFA A SUBTAREFA DO TESTE VEM **ANTES** DA SUBTAREFA DA IMPLEMENTAÇÃO</critical>
<critical>SE `<ui_projeto>` INDICAR FERRAMENTA DE VALIDAÇÃO VISUAL, TODA TAREFA QUE CRIA UMA TELA NOVA PRECISA DA SUBTAREFA DE TESTE VISUAL **ANTES** DA SUBTAREFA QUE IMPLEMENTA A TELA</critical>
<critical>A ÚLTIMA TAREFA DA LISTA É SEMPRE A DOCUMENTAÇÃO, CONFORME `<documentacao_projeto>` DO CONFIG. SE O CONFIG INDICAR REGISTRO EXTERNO OBRIGATÓRIO, A TAREFA COBRE OS DOIS DESTINOS</critical>

## Pré-requisitos

A funcionalidade em que você trabalhará é identificada por este slug:

- PRD obrigatório: `tasks/prd-<nome-da-feature>/prd.md` (conforme `<artefatos_sdd>` do config)
- TechSpec obrigatória: `tasks/prd-<nome-da-feature>/techspec.md` (conforme `<artefatos_sdd>`)

## Etapas do processo

1. **Analisar PRD e especificação técnica**

- Extrair requisitos e decisões técnicas
- Identificar os principais componentes e em qual camada cada um vive, conforme `<stack_projeto>`
- Extrair **todos** os casos de teste da seção "Abordagem de testes" (subseção "Níveis de teste") da <techspec> — eles são a entrada direta das subtarefas de teste
- Se `<integracoes_projeto>` indicar um rastreador de issues ativo e houver issue vinculada: leia-a pela ferramenta indicada no config para alinhar o escopo

2. **Gerar a estrutura de tarefas**

- Organizar a sequência por camada (ver "Ordem de sequenciamento" abaixo)
- **Cada tarefa deve ser uma entrega bem definida**
- **Toda tarefa tem seu próprio conjunto de testes**, no nível apropriado à sua camada

3. **Gerar arquivos individuais de tarefas**

- Criar um arquivo para cada tarefa principal
- Detalhar subtarefas e critérios de sucesso
- Detalhar os testes por nível, conforme os níveis declarados em `<stack_projeto>` e usados na "Abordagem de testes" da <techspec>

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

## Diretrizes para criação de tarefas

- Agrupar tarefas por entrega lógica
- Ordenar tarefas logicamente, seguindo a ordem de sequenciamento acima
- Tornar cada tarefa principal concluível de forma independente
- Definir escopo e entregáveis claros para cada tarefa
- Incluir testes como subtarefas dentro de cada tarefa principal, **sempre antes da implementação correspondente**
- Nenhuma tarefa pode alterar área listada em `<limites_projeto>` como read-only; se a dependência não existir lá, a tarefa vira "confirmar contrato com o time responsável" e é sinalizada como bloqueio
- **NÃO REPITA DETALHES DE IMPLEMENTAÇÃO** que já estão na especificação técnica — apenas faça referência a eles

## Especificações de saída

### Localização dos arquivos

- Pasta da funcionalidade: `./tasks/prd-<nome-da-feature>/` (conforme `<artefatos_sdd>`)
- Lista de tarefas: `./tasks/prd-<nome-da-feature>/tasks.md`
- Tarefas individuais: `./tasks/prd-<nome-da-feature>/<num>_task.md`
- Modelo para a lista de tarefas: <tasks_template>
- Modelo para cada tarefa individual: <task_template>

## Diretrizes finais

- Presuma que o leitor principal é um desenvolvedor com a stack de `<stack_projeto>`
- Evite criar mais de 10 tarefas (agrupe conforme definido antes)
- Use o formato X.0 para tarefas principais e X.Y para subtarefas
- A soma dos testes das tarefas precisa sustentar o threshold de `<comandos_projeto>`

Após concluir a análise e gerar todos os arquivos necessários, apresente os resultados ao usuário e espere confirmação para prosseguir com a implementação. Se `<integracoes_projeto>` indicar um rastreador ativo e houver issue vinculada: registre, pela ferramenta indicada no config, um comentário com o resumo da lista de tarefas.

## Checklist de qualidade

- [ ] Lista de alto nível aprovada antes de gerar arquivos
- [ ] Tarefas ordenadas conforme a ordem de sequenciamento (regra de negócio → dados/integrações → apresentação/entrada → registro e ligação → ponta a ponta → documentação)
- [ ] Toda tarefa tem subtarefa de teste **antes** da subtarefa de implementação (TDD)
- [ ] Se `<ui_projeto>` indicar ferramenta de validação visual, toda tela nova tem subtarefa de teste visual antes da implementação
- [ ] Todos os casos de teste da <techspec> foram distribuídos entre as tarefas
- [ ] Nenhuma tarefa altera área read-only de `<limites_projeto>`
- [ ] Tarefa final de documentação presente, conforme `<documentacao_projeto>`
- [ ] Arquivos gravados em `tasks/prd-<nome-da-feature>/` (conforme `<artefatos_sdd>`)

<critical>**ANTES DE GERAR QUALQUER ARQUIVO, MOSTRE A LISTA DE TAREFAS DE ALTO NÍVEL PARA APROVAÇÃO**</critical>
<critical>NÃO IMPLEMENTE NADA</critical>
<critical>TDD É OBRIGATÓRIO: EM TODA TAREFA A SUBTAREFA DO TESTE VEM **ANTES** DA SUBTAREFA DA IMPLEMENTAÇÃO</critical>
