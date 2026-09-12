---
name: executar-task
description: Identifique e implemente a próxima task técnica de uma feature a partir do PRD, TechSpec e tasks.md, seguindo TDD (teste falhando primeiro) e o gate de qualidade — comandos de lint, format, teste e cobertura, mais teste visual quando aplicável — definidos no sdd.config.md. Use sempre que o usuário pedir para executar, implementar ou continuar uma task/subtarefa, dar continuidade à implementação de uma funcionalidade, ou seguir a lista de tarefas em tasks.md.
---

<prd>`--prd`</prd>

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

Você é um desenvolvedor de alto nível responsável por implementar as tarefas tecnicamente. Você deve cuidar para identificar a próxima tarefa a ser executada, preparar-se e começar a **IMPLEMENTAR** seguindo TDD.

<critical>Não pule nenhuma subtarefa</critical>
<critical>**TDD É OBRIGATÓRIO**: teste falhando primeiro → confirmar o motivo da falha → código mínimo → refactor. Nenhum código de produção sem teste falhando antes. Exceções: ajuste de UI, código gerado, config</critical>
<critical>Antes de escrever ou refatorar qualquer teste, releia a seção "Qualidade dos testes" de `../_shared/QUALITY_BASELINE.md`</critical>
<critical>Respeite a direção de dependência entre camadas declarada em `<stack_projeto>`. Nenhuma camada importa de quem depende dela</critical>
<critical>Se `<ui_projeto>` indicar ferramenta de validação visual, toda tela nova exige teste visual escrito **antes** da implementação da tela</critical>
<critical>Nunca altere área listada como read-only em `<limites_projeto>`</critical>
<critical>A tarefa NÃO está completa com gate vermelho: os comandos de `<comandos_projeto>` (lint, format, testes, cobertura) precisam passar, e a cobertura precisa atingir o threshold declarado</critical>
<critical>Siga a convenção de commit de `<colaboracao_projeto>` ao registrar o trabalho</critical>
<critical>Após completar a tarefa, marque no tasks.md como completa</critical>

## Localização dos arquivos

- PRD: `tasks/prd-<nome-da-feature>/prd.md` (conforme `<artefatos_sdd>`)
- TechSpec: `tasks/prd-<nome-da-feature>/techspec.md` (conforme `<artefatos_sdd>`)
- Lista de tarefas: `tasks/prd-<nome-da-feature>/tasks.md` (conforme `<artefatos_sdd>`)
- Tarefa individual: `tasks/prd-<nome-da-feature>/<num>_task.md` (conforme `<artefatos_sdd>`)

Use o mesmo `<nome-da-feature>` do <prd> para localizar todos os artefatos da funcionalidade.

## Etapas

### 1. Preparação

- Identifique a próxima tarefa a ser executada (caso não esteja claro)
- Leia a definição da tarefa
- Revise o contexto do PRD
- Verifique os requisitos da TechSpec
- Entenda as dependências para tarefas anteriores
- Carregue as regras listadas no bloco `<regras>` da tarefa: `<limites_projeto>`,
  `<qualidade_extensoes>`, `<colaboracao_projeto>` e ambas as baselines
- Se `<integracoes_projeto>` indicar um rastreador de issues ativo e houver issue vinculada, leia-a
  pela ferramenta indicada no config para alinhar o escopo. Caso contrário, pule esta etapa.

### 2. Análise

- Compreenda os objetivos principais da tarefa
- Identifique **em qual camada** a tarefa vive (conforme `<stack_projeto>`) e quais arquivos serão
  criados/modificados
- Verifique o que já existe no código compartilhado/núcleo do projeto e pode ser reusado — reuso
  antes de abstração nova
- Se a tarefa consome uma integração externa, confirme o contrato na fonte de verdade (código do
  serviço, especificação, schema); se essa fonte estiver em área read-only de `<limites_projeto>`,
  apenas leia, nunca edite
- Analise possíveis soluções e abordagens

### 3. Design (quando a tarefa envolve UI)

Se `<ui_projeto>` indicar que o projeto tem interface de usuário final, a TechSpec traz referência
de design e `<integracoes_projeto>` indicar uma ferramenta de design ativa (ex.: Figma):

- Carregue a orientação de design-to-code da ferramenta indicada, quando existir, antes de
  consultá-la
- Extraia os tokens e componentes da referência de design
- Confronte com o design system/tema já existente no projeto: **reusar token/componente
  existente > criar novo**
- Baixe os assets que faltarem para o destino convencionado no projeto

Caso contrário, pule esta etapa.

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

- Use a biblioteca de mock declarada em `<stack_projeto>` e nenhuma outra
- Nome de teste descreve o comportamento esperado e a condição
- Asserção específica: verifique o valor concreto e o tipo de erro concreto, não apenas que
  "não é nulo" ou que "não lançou"
- Falha de rede/integração usa o erro real da biblioteca do projeto, com o status e o tipo
  corretos — nunca um erro genérico
- Mock na fronteira certa: o teste de um componente dubla suas dependências diretas, não ele mesmo
- Injeção por construtor/parâmetro no teste; sem bootstrap do contêiner de injeção em teste unitário
- Teste visual, quando aplicável, usa o helper e a superfície declarados em `<ui_projeto>`

### 6. Verificação (obrigatória antes de fechar a tarefa)

Rode, nesta ordem, os comandos declarados em `<comandos_projeto>`: format, lint, testes e
cobertura. Nenhum pode falhar.

- Se a cobertura reprovar, **escreva mais teste** — nunca baixe o threshold
- Se o projeto tem E2E e a tarefa o afeta, rode o comando de E2E. **Peça confirmação antes** de
  qualquer execução que escreva dados em ambiente compartilhado
- Confira o código alterado contra `../_shared/SECURITY_BASELINE.md` e
  `../_shared/QUALITY_BASELINE.md` antes de marcar a tarefa como concluída

### 7. Fechamento

- Marque a tarefa como completa em `tasks.md`
- Se `<integracoes_projeto>` indicar um rastreador de issues ativo (ex.: Linear, Jira, GitHub
  Issues) e houver issue vinculada, registre pela ferramenta indicada no config um comentário
  resumindo o que foi implementado, os arquivos tocados e o resultado dos comandos de
  `<comandos_projeto>`. Caso contrário, pule esta etapa.
- Se esta era a última tarefa de implementação da feature, verifique `<documentacao_projeto>` do
  config e registre a documentação nos destinos indicados (pasta do projeto e, se houver, registro
  externo obrigatório)

## Checklist de qualidade

- [ ] Testes existentes revisados contra a seção "Qualidade dos testes" de
      `../_shared/QUALITY_BASELINE.md` antes de alterá-los
- [ ] Todo código de produção precedido de teste falhando pelo motivo certo
- [ ] Teste visual escrito antes de toda tela nova, quando `<ui_projeto>` indicar ferramenta de
      validação visual
- [ ] Direção de dependência entre camadas de `<stack_projeto>` respeitada
- [ ] Nenhuma área read-only de `<limites_projeto>` alterada
- [ ] Comandos de `<comandos_projeto>` (format, lint, testes) verdes
- [ ] Cobertura de `<comandos_projeto>` no threshold declarado
- [ ] Conformidade verificada contra `../_shared/SECURITY_BASELINE.md` e
      `../_shared/QUALITY_BASELINE.md`
- [ ] `tasks.md` atualizado
- [ ] Comentário postado na issue do rastreador configurado (se houver)

<critical>Não pule nenhuma subtarefa</critical>
<critical>**TDD É OBRIGATÓRIO**: teste falhando primeiro → confirmar o motivo da falha → código mínimo → refactor</critical>
<critical>**INICIE** a implementação logo após as etapas de preparação e análise</critical>
<critical>A tarefa NÃO está completa com gate vermelho: os comandos de `<comandos_projeto>` precisam passar e a cobertura precisa atingir o threshold declarado</critical>
<critical>Após completar a tarefa, marque no tasks.md como completa</critical>
