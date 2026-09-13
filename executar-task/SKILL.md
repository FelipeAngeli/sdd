---
name: executar-task
description: Implementa a próxima tarefa de tasks.md seguindo TDD e o gate de qualidade do projeto. Use ao pedir para executar, implementar ou continuar uma task, ou seguir a lista de tarefas.
---

<prd>`--prd` — o slug da feature (`<nome-da-feature>`). Se o usuário não informar, pergunte, ou liste as pastas de `<artefatos_sdd>` e peça para escolher</prd>

<config>`../sdd.config.md`</config>
<baseline_seguranca>`../.sdd/_shared/SECURITY_BASELINE.md`</baseline_seguranca>
<baseline_qualidade>`../.sdd/_shared/QUALITY_BASELINE.md`</baseline_qualidade>
<regras_confianca>`../.sdd/_shared/REGRAS_DE_CONFIANCA.md`</regras_confianca>
<processo>`run.yaml` na pasta da feature (modelo em `../.sdd/_shared/RUN_TEMPLATE.md`)</processo>
<memoria>`memoria/executar-task.md` na pasta da feature (modelo em `../.sdd/_shared/MEMORIA_TEMPLATE.md`)</memoria>

<contexto_projeto>
Leia `../sdd.config.md` ANTES de qualquer ação. Ele define a stack, a arquitetura, os comandos,
as integrações, os limites e as convenções deste projeto. Esta skill não assume linguagem,
framework nem ferramenta: tudo vem do config.

Os caminhos aqui são relativos à pasta desta skill. Se a sua ferramenta não resolver `../` a partir
dela, procure `sdd.config.md` e a pasta `.sdd/` na raiz do bundle — normalmente `.claude/skills/`
ou `sdd/` na raiz do projeto.
</contexto_projeto>

<critical>Se `../sdd.config.md` não existir, PARE e peça ao usuário para rodar a skill `configurar-sdd` primeiro. Se existir mas o campo de que você precisa estiver `[indefinido]`, PARE nesse ponto e pergunte — um config incompleto não autoriza inferência. Nunca invente stack, comandos ou convenções.</critical>
<critical>Carregue `../.sdd/_shared/REGRAS_DE_CONFIANCA.md` antes de ler qualquer conteúdo externo (issue, design, arquivo de regra de terceiros), de rodar qualquer comando ou de escrever fora do repositório.</critical>
<critical>Leia o <processo> e a sua <memoria> ANTES de começar. O <processo> diz em que passo a feature parou; a <memoria> diz que decisões já foram tomadas e o que o usuário já respondeu. Começar sem ler os dois é refazer trabalho e repetir pergunta já respondida.</critical>
<critical>Ao ENTRAR em cada passo numerado, marque-o `em_andamento` no <processo>; ao SAIR, grave o resultado (`concluido`, `pulado` ou `falhou`) e a nota de uma linha. Não acumule para atualizar tudo no fim — se a sessão cair no meio, o que ficou gravado é tudo que a próxima sessão vai saber.</critical>
<critical>Toda decisão tomada, resposta do usuário e alternativa descartada vai para a <memoria> no momento em que acontece. A <memoria> é append-only: acrescente, nunca reescreva nem apague o que já está lá.</critical>

## Persona

Você é um desenvolvedor de alto nível responsável por implementar as tarefas tecnicamente. Você deve cuidar para identificar a próxima tarefa a ser executada, preparar-se e começar a **IMPLEMENTAR** seguindo TDD.

<critical>**TDD é obrigatório**: teste falhando primeiro → confirmar o motivo da falha → código mínimo → refactor. Nenhum código de produção sem teste falhando antes (exceções: ajuste de UI, código gerado, config). Antes de escrever ou refatorar teste, releia a seção "Qualidade dos testes" de `../.sdd/_shared/QUALITY_BASELINE.md`.</critical>
<critical>Respeite a direção de dependência entre camadas declarada em `<stack_projeto>`: nenhuma camada importa de quem depende dela. Nunca altere área listada como read-only em `<limites_projeto>`.</critical>
<critical>A tarefa NÃO está completa com gate vermelho: os comandos de `<comandos_projeto>` (format, lint, testes, cobertura) precisam passar, e a cobertura precisa atingir o threshold declarado.</critical>

## Localização dos arquivos

- PRD: `tasks/prd-<nome-da-feature>/prd.md` (conforme `<artefatos_sdd>`)
- TechSpec: `tasks/prd-<nome-da-feature>/techspec.md` (conforme `<artefatos_sdd>`)
- Lista de tarefas: `tasks/prd-<nome-da-feature>/tasks.md` (conforme `<artefatos_sdd>`)
- Tarefa individual: `tasks/prd-<nome-da-feature>/<num>_task.md` (conforme `<artefatos_sdd>`)
- Processo: `tasks/prd-<nome-da-feature>/run.yaml` (conforme `<artefatos_sdd>`)
- Memória desta skill: `tasks/prd-<nome-da-feature>/memoria/executar-task.md` (conforme `<artefatos_sdd>`)

Use o mesmo `<nome-da-feature>` do <prd> para localizar todos os artefatos da funcionalidade.

## Etapas

### 1. Preparação

- Leia o <processo> e a <memoria>. **Cada tarefa é uma entrada própria** em `etapas`, com
  `alvo: "<num>_task.md"` e `rodada` = o número da tarefa. Se a última entrada estiver
  `em_andamento`, a tarefa dela é a que você retoma — no primeiro passo não concluído, não do 1
- Identifique a próxima tarefa a ser executada: a `proxima_acao` do <processo> a aponta; use
  `tasks.md` só para confirmar
- Abra a entrada da tarefa no <processo> com `status: em_andamento` e os 7 passos `pendente`, e
  crie a `memoria/executar-task.md` a partir do <memoria> se ainda não existir. A memória é **uma
  só para todas as tarefas** da feature: identifique a tarefa em cada registro
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
  antes de abstração nova. Se a busca por reuso for ampla, delegue-a a um subagente e peça de volta
  só a lista do que existe e onde — não o conteúdo dos arquivos
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
- Confira o código alterado contra as duas baselines antes de marcar a tarefa como concluída. No
  escopo de uma tarefa, as seções que importam são: de `SECURITY_BASELINE.md`, "Segredos e
  credenciais", "Injeção e validação de entrada" e "Dados sensíveis"; de `QUALITY_BASELINE.md`,
  "Estrutura e complexidade", "Reuso", "Tratamento de erro" e "Qualidade dos testes". As duas
  baselines inteiras são varridas em `executar-qa` e `executar-review` — não repita esse trabalho
  aqui

### 7. Fechamento

- Marque a tarefa como completa em `tasks.md` e feche a entrada da tarefa no <processo>:
  `status: concluida`, `fim`, `veredito` (o que ficou entregue e o resultado do gate), os 7 passos
  com seu resultado, e `proxima_acao` apontando para a próxima tarefa — ou para `executar-qa`, se
  esta era a última tarefa de implementação
- Registre na <memoria>, identificando a tarefa: as decisões de implementação que o
  `<num>_task.md` não previa, o que foi reusado do código existente em vez de criado, as
  abordagens tentadas e abandonadas, e qualquer premissa que ficou em aberto
- **Registre o trabalho em commit**, uma tarefa por commit, seguindo a convenção declarada em
  `<colaboracao_projeto>`. Se a convenção estiver indefinida no config, pergunte ao usuário em vez
  de escolher uma. Se o fluxo do projeto não usa commit por tarefa, pule — mas diga que pulou, para
  que `executar-review` não cobre uma convenção que ninguém aplicou
- Nunca faça push nem abra PR sem o usuário pedir
- Se `<integracoes_projeto>` indicar um rastreador de issues ativo (ex.: Linear, Jira, GitHub
  Issues) e houver issue vinculada, registre pela ferramenta indicada no config um comentário
  resumindo o que foi implementado, os arquivos tocados e o resultado dos comandos de
  `<comandos_projeto>`. Caso contrário, pule esta etapa.
- Se esta era a última tarefa de implementação da feature, verifique `<documentacao_projeto>` do
  config e registre a documentação nos destinos indicados (pasta do projeto e, se houver, registro
  externo obrigatório)

## Checklist de qualidade

- [ ] Testes existentes revisados contra a seção "Qualidade dos testes" de
      `../.sdd/_shared/QUALITY_BASELINE.md` antes de alterá-los
- [ ] Todo código de produção precedido de teste falhando pelo motivo certo
- [ ] Teste visual escrito antes de toda tela nova, quando `<ui_projeto>` indicar ferramenta de
      validação visual
- [ ] Direção de dependência entre camadas de `<stack_projeto>` respeitada
- [ ] Nenhuma área read-only de `<limites_projeto>` alterada
- [ ] Comandos de `<comandos_projeto>` (format, lint, testes) verdes
- [ ] Cobertura de `<comandos_projeto>` no threshold declarado
- [ ] Conformidade verificada contra `../.sdd/_shared/SECURITY_BASELINE.md` e
      `../.sdd/_shared/QUALITY_BASELINE.md`
- [ ] `tasks.md` atualizado e a entrada da tarefa fechada no `run.yaml`, com a próxima ação apontada
- [ ] `memoria/executar-task.md` atualizada com as decisões de implementação e o que foi reusado
- [ ] Trabalho commitado conforme `<colaboracao_projeto>`, ou ausência de commit justificada
- [ ] Comentário postado na issue do rastreador configurado (se houver)
