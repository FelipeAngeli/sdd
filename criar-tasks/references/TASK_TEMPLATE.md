# Tarefa X.0: [Título da tarefa]

## Visão geral

[Descrição breve da tarefa]

**Camada:** [uma das camadas declaradas em `<stack_projeto>`]

<regras>
### Regras e convenções aplicáveis

[Liste as regras do projeto que esta tarefa precisa respeitar: as de `<limites_projeto>`, as
convenções de `<qualidade_extensoes>` e de `<colaboracao_projeto>`, e as baselines de segurança
e de qualidade do bundle SDD. Carregue tudo antes de implementar.]
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

<critical>Antes de escrever ou refatorar qualquer teste, leia a baseline de qualidade do bundle SDD, seção "Qualidade dos testes". Use a biblioteca de mock declarada em `<stack_projeto>` e nenhuma outra. O nome do teste descreve o comportamento esperado e a condição.</critical>

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
