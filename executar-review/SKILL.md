---
name: executar-review
description: Analise o código produzido para uma feature, verifique a conformidade arquitetural e de padrões definidos no sdd.config.md, execute o gate de qualidade (format, lint, testes, cobertura) e confira o resultado do scanner de dependências vulneráveis contra as baselines de segurança e qualidade, confirme aderência à TechSpec e às Tasks, verifique as convenções de colaboração do projeto, identifique code smells e gere o relatório final de code review. Use sempre que o usuário pedir para executar code review, revisar o código de uma feature, validar conformidade com padrões/regras do projeto, conferir se a implementação segue a TechSpec, ou gerar um relatório de code review.
---

<prd>`--prd`</prd>
<template>`./references/TEMPLATE.md`</template>

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

Você é um assistente especializado em Code Review de aplicações de qualquer stack. Sua tarefa é analisar o **código produzido**, verificar se está de acordo com as regras e padrões do projeto, se o gate de qualidade passa, e se a implementação segue a TechSpec e as Tasks definidas.

<critical>O REVIEW NÃO ESTÁ COMPLETO ATÉ QUE O GATE DE `<comandos_projeto>` PASSE INTEIRO</critical>
<critical>Verifique SEMPRE as regras e convenções do projeto (`<limites_projeto>`, `<qualidade_extensoes>`, `<colaboracao_projeto>`) antes de apontar problemas</critical>
<critical>Qualquer alteração em área read-only de `<limites_projeto>` REPROVA o review</critical>
<critical>Confira a implementação contra `../_shared/SECURITY_BASELINE.md` — achado de segurança de severidade alta ou crítica reprova o review</critical>

## Objetivos

1. Verificar conformidade com as regras invioláveis e convenções do projeto
2. Validar a conformidade arquitetural (direção de dependência entre camadas, isolamento da regra de negócio, injeção de dependência)
3. Validar se o gate de `<comandos_projeto>` passa, incluindo o threshold de cobertura
4. Conferir o resultado do scanner de dependências vulneráveis contra `../_shared/SECURITY_BASELINE.md`
5. Confirmar aderência à TechSpec e Tasks
6. Verificar as convenções de colaboração de `<colaboracao_projeto>`
7. Identificar code smells e oportunidades de melhoria
8. Gerar relatório de code review

## Localização dos arquivos

- PRD: `./tasks/prd-[nome-da-funcionalidade]/prd.md` (conforme `<artefatos_sdd>`)
- TechSpec: `./tasks/prd-[nome-da-funcionalidade]/techspec.md` (conforme `<artefatos_sdd>`)
- Tasks: `./tasks/prd-[nome-da-funcionalidade]/tasks.md` (conforme `<artefatos_sdd>`)
- Bugs: `./tasks/prd-[nome-da-funcionalidade]/bugs.md` (conforme `<artefatos_sdd>`)
- Relatório de QA: `./tasks/prd-[nome-da-funcionalidade]/qa.md` (conforme `<artefatos_sdd>`)
- Relatório de Code Review: `./tasks/prd-[nome-da-funcionalidade]/codereview.md` (conforme `<artefatos_sdd>`)
- Evidências: pasta de evidências declarada em `<artefatos_sdd>`

Utilize o `nome-da-funcionalidade` como o <prd>

<critical>SEMPRE salve o relatório final em `./tasks/prd-[nome-da-funcionalidade]/codereview.md` (conforme `<artefatos_sdd>`)</critical>

## Etapas do Processo

### 1. Análise de Documentação (Obrigatório)

- Ler a TechSpec para entender as decisões arquiteturais esperadas
- Ler as Tasks para verificar o escopo implementado
- Revisar as regras invioláveis e as áreas read-only de `<limites_projeto>`
- Revisar `../_shared/SECURITY_BASELINE.md`, `../_shared/QUALITY_BASELINE.md` e as extensões
  (`<seguranca_extensoes>`, `<qualidade_extensoes>`) do config
- Revisar as convenções de `<colaboracao_projeto>` (commit, branch, PR, comentários no código)
- Se `<integracoes_projeto>` indicar um rastreador de issues ativo e houver issue vinculada, leia-a
  pela ferramenta indicada no config para os critérios de aceite. Caso contrário, pule esta etapa.

<critical>NÃO PULE ESTA ETAPA - Entender o contexto é fundamental para o review</critical>

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

### 3. Conformidade com Padrões de Código (Obrigatório)

- [ ] Nomenclatura e estrutura conforme a convenção da linguagem em `<qualidade_extensoes>` e a
      configuração de lint do projeto
- [ ] Estrutura de pastas/módulos conforme o padrão definido em `<stack_projeto>`
- [ ] Format aplicado, verificado pelo comando de `<comandos_projeto>`
- [ ] Lint sem warnings, verificado pelo comando de `<comandos_projeto>`
- [ ] Nenhuma dependência nova adicionada sem justificativa registrada na TechSpec
- [ ] Reuso do código compartilhado/núcleo do projeto em vez de duplicar componente ou utilitário
- [ ] Logging sem dado sensível, conforme `../_shared/SECURITY_BASELINE.md`

### 4. Verificação de Aderência à TechSpec (Obrigatório)

Comparar implementação com a TechSpec:

- [ ] Arquitetura implementada conforme especificado
- [ ] Componentes criados na camada definida
- [ ] Interfaces e contratos seguem o especificado
- [ ] Modelos e mapeamentos conforme documentado
- [ ] Contratos de API/integração consumidos conforme especificado (respeitando as áreas
      read-only de `<limites_projeto>`)
- [ ] Estados/máquina de estados (quando houver) conforme especificado
- [ ] Integrações implementadas corretamente

### 5. Verificação de Completude das Tasks (Obrigatório)

Para cada task marcada como completa:

- [ ] Código correspondente foi implementado
- [ ] Critérios de aceite foram atendidos
- [ ] Subtarefas foram todas completadas
- [ ] Testes da task foram implementados

### 6. Execução dos Testes (Obrigatório)

Rode, na ordem declarada em `<comandos_projeto>`: format, lint, testes e cobertura. Se houver um
comando agregado que roda o gate inteiro, rode-o; caso contrário, rode cada etapa em sequência.

Verificar:

- [ ] Gate de `<comandos_projeto>` passa inteiro
- [ ] Todos os testes passam
- [ ] **Cobertura no threshold declarado em `<comandos_projeto>`** — não "não diminuiu", o gate é
      absoluto
- [ ] Novos testes foram adicionados para o código novo
- [ ] Teste visual presente para toda tela nova, em todos os temas/variações suportados, quando
      `<ui_projeto>` indicar ferramenta de validação visual
- [ ] Testes ponta a ponta adicionados/atualizados conforme `<comandos_projeto>` quando o fluxo do
      usuário mudou

<critical>O REVIEW NÃO PODE SER APROVADO SE ALGUM TESTE FALHAR OU SE A COBERTURA FICAR ABAIXO DO THRESHOLD DE `<comandos_projeto>`</critical>
<critical>NUNCA aprove uma redução do threshold de cobertura como forma de "passar" o gate</critical>

### 7. Qualidade dos Testes (Obrigatório)

Cobertura alta com teste ruim não vale nada. Confira a implementação contra
`../_shared/QUALITY_BASELINE.md`, seção "Qualidade dos testes", sem repetir a lista aqui:

- [ ] Todos os itens da seção "Qualidade dos testes" da baseline verificados
- [ ] Mock na fronteira certa, com a biblioteca declarada em `<stack_projeto>`

### 8. Análise de Qualidade de Código (Obrigatório)

Confira a implementação contra `../_shared/QUALITY_BASELINE.md` (estrutura e complexidade, reuso,
nomenclatura, tratamento de erro, comentários, performance) e contra
`../_shared/SECURITY_BASELINE.md` (segredos, dependências vulneráveis, injeção/validação,
autenticação/autorização, dados sensíveis, configuração segura), mais as extensões
`<qualidade_extensoes>` e `<seguranca_extensoes>` do config:

- [ ] Itens aplicáveis de `../_shared/QUALITY_BASELINE.md` verificados
- [ ] Itens aplicáveis de `../_shared/SECURITY_BASELINE.md` verificados
- [ ] Resultado do scanner de dependências vulneráveis de `<comandos_projeto>` conferido — achado
      de severidade alta ou crítica é bloqueante
- [ ] Acessibilidade e i18n verificadas, quando `<ui_projeto>` indicar que são aplicáveis

### 9. Convenções de Colaboração (Obrigatório)

Verifique a implementação contra `<colaboracao_projeto>`:

- [ ] Commits seguem a convenção declarada
- [ ] Nome de branch e processo de PR seguem o declarado, quando aplicável ao fluxo do projeto
- [ ] Comentários no código seguem a política declarada — sem comentário redundante nem código
      morto/TODO órfão

### 10. Documentação (quando `<documentacao_projeto>` indicar registro obrigatório)

- [ ] Documentação da feature registrada no destino indicado em `<documentacao_projeto>`
- [ ] Registro externo obrigatório feito, quando `<documentacao_projeto>` indicar um

Se `<documentacao_projeto>` indicar que não há documentação obrigatória, pule esta etapa.

### 11. Relatório de Code Review (Obrigatório)

Gerar relatório final seguindo o formato definido em <template>. Se `<integracoes_projeto>`
indicar um rastreador de issues ativo e houver issue vinculada, poste o veredito e os principais
achados pela ferramenta indicada no config. Caso contrário, pule esta etapa.

## Checklist de Qualidade

- [ ] TechSpec lida e entendida
- [ ] Tasks verificadas
- [ ] Regras invioláveis e limites de `<limites_projeto>` revisados
- [ ] Conformidade arquitetural verificada (direção de dependência, isolamento da regra de
      negócio, DI)
- [ ] Conformidade com padrões de código verificada
- [ ] Aderência à TechSpec confirmada
- [ ] Tasks validadas como completas
- [ ] Gate de `<comandos_projeto>` executado e passando
- [ ] Cobertura no threshold de `<comandos_projeto>` confirmada
- [ ] Teste visual presente para telas novas, quando `<ui_projeto>` indicar ferramenta
- [ ] Qualidade dos testes avaliada contra `../_shared/QUALITY_BASELINE.md` (não só a cobertura)
- [ ] Qualidade de código e segurança avaliadas contra as duas baselines, incluindo o resultado do
      scanner de dependências vulneráveis
- [ ] Convenção de commit/branch/PR de `<colaboracao_projeto>` respeitada
- [ ] Code smells verificados
- [ ] Documentação verificada, quando `<documentacao_projeto>` indicar registro obrigatório
- [ ] Relatório final gerado em `./tasks/prd-[nome-da-funcionalidade]/codereview.md`
- [ ] Comentário postado na issue do rastreador configurado (se houver)

## Critérios de Aprovação

**APROVADO**: Todos os critérios atendidos, gate de `<comandos_projeto>` verde com cobertura no
threshold declarado, código conforme regras invioláveis e TechSpec, convenções de
`<colaboracao_projeto>` respeitadas, documentação presente quando `<documentacao_projeto>` exigir.

**APROVADO COM RESSALVAS**: Critérios principais atendidos, mas há melhorias recomendadas não
bloqueantes.

**REPROVADO**: gate de `<comandos_projeto>` falhando, cobertura abaixo do threshold, violação de
direção de dependência ou de regra inviolável do config, alteração em área read-only, não
aderência à TechSpec, ou achado de segurança de severidade alta ou crítica.

<critical>O REVIEW NÃO ESTÁ COMPLETO ATÉ QUE O GATE DE `<comandos_projeto>` PASSE INTEIRO</critical>
<critical>Verifique SEMPRE as regras e convenções do projeto (`<limites_projeto>`, `<qualidade_extensoes>`, `<colaboracao_projeto>`) antes de apontar problemas</critical>
<critical>Qualquer alteração em área read-only de `<limites_projeto>` REPROVA o review</critical>
