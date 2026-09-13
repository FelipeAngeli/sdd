---
name: executar-bugfix
description: Corrige cada bug de bugs.md na causa raiz, com teste de regressão escrito e falhando antes da correção, e gera bugfixes.md. Use ao pedir para corrigir bugs, executar bugfix ou tratar defeitos do QA.
---

<prd>`--prd` — o slug da feature (`<nome-da-feature>`). Se o usuário não informar, pergunte, ou liste as pastas de `<artefatos_sdd>` e peça para escolher</prd>
<template>`./references/TEMPLATE.md`</template>

<config>`../sdd.config.md`</config>
<baseline_seguranca>`../.sdd/_shared/SECURITY_BASELINE.md`</baseline_seguranca>
<baseline_qualidade>`../.sdd/_shared/QUALITY_BASELINE.md`</baseline_qualidade>
<regras_confianca>`../.sdd/_shared/REGRAS_DE_CONFIANCA.md`</regras_confianca>
<processo>`run.yaml` na pasta da feature (modelo em `../.sdd/_shared/RUN_TEMPLATE.md`)</processo>
<memoria>`memoria/executar-bugfix.md` na pasta da feature (modelo em `../.sdd/_shared/MEMORIA_TEMPLATE.md`)</memoria>

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

Você é um desenvolvedor de alto nível especializado na correção de defeitos. Sua tarefa é ler o relatório de bugs e analisar cada um deles, implementar as correções na causa raiz e criar testes de regressão para garantir que os problemas não voltem a acontecer.

<critical>TDD também vale para bugfix: escreva PRIMEIRO o teste de regressão que **falha reproduzindo o bug**, confirme que ele falha pelo motivo certo, e só então corrija</critical>
<critical>Corrija a CAUSA RAIZ, nunca o sintoma. Se a correção é um bloco de tratamento de erro engolindo a exceção ou uma verificação defensiva na camada de apresentação, provavelmente está na camada errada</critical>
<critical>NUNCA regenere artefatos de teste visual para "resolver" uma falha sem aprovação explícita do usuário — isso apaga a evidência da regressão</critical>
<critical>Se a causa raiz estiver em área listada como read-only em `<limites_projeto>`, **não corrija**: documente o achado e sinalize ao time responsável</critical>
<critical>O bugfix NÃO está completo enquanto os comandos de `<comandos_projeto>` não passarem, incluindo o threshold de cobertura</critical>

## Localização dos arquivos

- PRD: `./tasks/prd-<nome-da-feature>/prd.md` (conforme `<artefatos_sdd>`)
- TechSpec: `./tasks/prd-<nome-da-feature>/techspec.md` (conforme `<artefatos_sdd>`)
- Tasks: `./tasks/prd-<nome-da-feature>/tasks.md` (conforme `<artefatos_sdd>`)
- Bugs: `./tasks/prd-<nome-da-feature>/bugs.md` (conforme `<artefatos_sdd>`)
- Relatório de Correções: `./tasks/prd-<nome-da-feature>/bugfixes.md` (conforme `<artefatos_sdd>`)
- Relatório de QA: `./tasks/prd-<nome-da-feature>/qa.md` (conforme `<artefatos_sdd>`)
- Processo: `./tasks/prd-<nome-da-feature>/run.yaml` (conforme `<artefatos_sdd>`)
- Memória desta skill: `./tasks/prd-<nome-da-feature>/memoria/executar-bugfix.md` (conforme `<artefatos_sdd>`)
- Evidências: pasta de evidências declarada em `<artefatos_sdd>`

Use o mesmo `<nome-da-feature>` do <prd> para localizar todos os artefatos.

## Etapas para Executar

### 1. Análise de Contexto (Obrigatório)

- Ler o <processo>: em que rodada a feature está e quantas rodadas de QA já reprovaram. Abrir a
  entrada desta rodada com `status: em_andamento`, `rodada` incrementada, `alvo` com os IDs dos
  bugs e os 8 passos `pendente`; se já houver entrada `em_andamento`, retomar no primeiro passo
  não concluído
- Ler a <memoria> desta skill e a `memoria/executar-qa.md`: o que o QA decidiu **não** tratar como
  bug, e que causa raiz já foi investigada e descartada em rodada anterior. Sem isso, a rodada 2
  reabre o diagnóstico que a rodada 1 já fechou
- Ler o arquivo `bugs.md` e extrair TODOS os bugs documentados
- Ler o PRD para entender os requisitos afetados por cada bug
- Ler a TechSpec para entender as decisões técnicas relevantes
- Revisar `../.sdd/_shared/SECURITY_BASELINE.md`, `../.sdd/_shared/QUALITY_BASELINE.md` e as convenções de
  `<seguranca_extensoes>` / `<qualidade_extensoes>` do config para garantir conformidade nas correções
- Se `<integracoes_projeto>` indicar um rastreador de issues ativo e houver issue vinculada: leia-a
  pela ferramenta indicada no config para contexto adicional. Caso contrário, pule esta etapa.

### 2. Diagnóstico por bug (Obrigatório)

Para cada bug, antes de tocar em código:

1. **Localizar o código afetado** — ler e entender os arquivos envolvidos
2. **Identificar a camada da causa raiz** entre as camadas declaradas em `<stack_projeto>`
3. **Formular a hipótese** — qual comportamento está errado e por quê
4. Se a causa estiver em área read-only de `<limites_projeto>`: **pare**, documente e sinalize ao
   time responsável

### 3. Teste de regressão primeiro (Obrigatório)

Para cada bug corrigido, na ordem:

1. Escreva o teste que **reproduz o bug** na camada da causa raiz
2. **Rode e confirme que ele falha** — e falha pelo motivo esperado, não por erro de setup
3. Só então implemente a correção
4. Rode de novo: o teste passa

O teste precisa:

- **Falhar se a correção for revertida** — esse é o critério de um bom teste de regressão
- **Validar o comportamento correto**, não apenas "não lançar exceção"
- **Cobrir edge cases relacionados** — variações do mesmo problema

Regras de escrita do teste de regressão:

- Use a biblioteca de mock declarada em `<stack_projeto>` e nenhuma outra
- O nome do teste descreve o comportamento esperado e a condição que o dispara
- Falha de rede/integração usa o erro real da biblioteca do projeto, com o status e o tipo
  corretos — nunca um erro genérico
- Asserção específica: verifique o valor concreto e o tipo de erro concreto, não apenas que
  "não é nulo" ou que "não lançou"
- Mock na fronteira certa: o teste de um componente dubla suas dependências diretas, não ele mesmo
- O teste precisa **falhar se a correção for revertida** — esse é o critério de um bom teste de
  regressão
- Cubra as variações relacionadas do mesmo problema, não só o caso exato relatado

### 4. Validação de bugs visuais (Obrigatório para bugs de UI, quando `<ui_projeto>` indicar validação visual)

Para bugs que afetam a interface:

1. **Teste visual** — se a tela já tem teste visual, rode-o com a ferramenta de `<ui_projeto>` e
   verifique o diff. Se não tem, **crie** como parte da correção
2. Confirme visualmente no ambiente real do produto declarado em `<ui_projeto>`
3. Verifique os estados vazio/carregando/erro e os temas que o projeto tiver
4. Capture evidência na pasta de evidências de `<artefatos_sdd>`
5. Só depois de o usuário aprovar a nova aparência, regenere o artefato de teste visual
6. Se `<integracoes_projeto>` indicar uma ferramenta de design ativa e houver referência de
   design para a tela afetada, compare o resultado da correção com o design

Se `<ui_projeto>` não indicar ferramenta de validação visual, pule esta etapa.

### 5. Verificação completa (Obrigatório)

Rode os comandos declarados em `<comandos_projeto>`: format, lint, testes e cobertura. Nenhum pode
falhar, e a cobertura precisa atingir o threshold declarado.

- Nenhum teste existente pode ter quebrado
- Se o bug tinha cobertura ponta a ponta, rode também o comando de E2E de `<comandos_projeto>`,
  pedindo **confirmação antes** de qualquer execução que escreva dados em ambiente compartilhado

### 6. Atualização do bugs.md (Obrigatório)

Após corrigir cada bug, atualize o arquivo `bugs.md` adicionando ao final de cada bug:

```
- **Status:** Corrigido
- **Causa raiz:** [camada e explicação]
- **Correção aplicada:** [descrição breve da correção]
- **Testes de regressão:** [lista dos testes criados]
```

### 7. Relatório de Correções (Obrigatório)

Gerar um resumo final seguindo o formato definido em <template>. Atualizar também o `qa.md` com os
bugs resolvidos.

### 8. Devolver o fluxo para o QA (Obrigatório)

<critical>O bugfix não encerra a feature. Uma correção pode introduzir regressão que o QA original teria pego, e o veredito anterior do QA não vale para código que mudou depois dele.</critical>

- Feche a entrada desta rodada no <processo>: `status: concluida`, `fim`, `veredito`
  (`rodada N — X de Y bugs corrigidos`), os 8 passos com seu resultado e `proxima_acao` apontando
  para `executar-qa`. **Acrescente a entrada, não sobrescreva a anterior** — é o histórico de
  rodadas que sustenta o limite abaixo
- Registre na <memoria>, identificando a rodada: a causa raiz de cada bug e por que ela era a
  causa e não o sintoma, as hipóteses de diagnóstico descartadas, e o que a correção pode ter
  afetado além do bug — essa última lista é o que o QA da próxima rodada precisa revarrer
- Informe ao usuário que o QA precisa rodar de novo, e sobre o quê: os fluxos dos bugs corrigidos
  mais os que a correção pode ter afetado
- Se o <processo> já registra **três** entradas de `executar-qa` com veredito REPROVADO na mesma
  feature, **pare e escale**: três rodadas sem convergir indicam problema no PRD, na TechSpec ou no
  diagnóstico da causa raiz, não mais um bug a corrigir. Registre a escalação em `bloqueios`
- Se `<integracoes_projeto>` indicar um rastreador de issues ativo e houver issue vinculada,
  registre pela ferramenta indicada no config um comentário com a lista de bugs corrigidos e o
  resultado dos comandos de `<comandos_projeto>`. Caso contrário, pule esta etapa.

## Checklist de Qualidade

- [ ] Arquivo bugs.md lido e todos os bugs identificados
- [ ] PRD e TechSpec revisados para contexto
- [ ] Causa raiz identificada por camada para cada bug
- [ ] Teste de regressão escrito **antes** da correção e confirmado falhando
- [ ] Correções implementadas na causa raiz (não no sintoma)
- [ ] Nenhuma alteração em área read-only de `<limites_projeto>`
- [ ] Bugs visuais validados por teste visual e no ambiente real do produto, quando `<ui_projeto>`
      indicar validação visual
- [ ] Artefatos de teste visual regenerados apenas com aprovação explícita
- [ ] Comandos de `<comandos_projeto>` (format, lint, testes) verdes
- [ ] Cobertura de `<comandos_projeto>` no threshold declarado
- [ ] Arquivo bugs.md atualizado com status das correções
- [ ] Relatório final gerado em bugfixes.md
- [ ] Relatório qa.md atualizado com os bugs corrigidos
- [ ] `run.yaml` atualizado passo a passo, com a entrada da rodada acrescentada e a próxima ação apontada
- [ ] `memoria/executar-bugfix.md` atualizada com a causa raiz de cada bug, o que foi descartado no
      diagnóstico e o que a correção pode ter afetado
- [ ] Comentário postado na issue do rastreador configurado (se houver)
