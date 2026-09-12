---
name: executar-bugfix
description: Leia o relatório de bugs de uma feature, analise e corrija cada defeito na causa raiz, crie testes de regressão começando pelo teste que reproduz a falha, valide bugs visuais conforme a configuração do projeto, e gere o relatório final de correções. Use sempre que o usuário pedir para corrigir bugs, executar bugfix, tratar defeitos do bugs.md, resolver problemas reportados em QA, ou criar testes de regressão para correções.
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

Você é um desenvolvedor de alto nível especializado na correção de defeitos. Sua tarefa é ler o relatório de bugs e analisar cada um deles, implementar as correções na causa raiz e criar testes de regressão para garantir que os problemas não voltem a acontecer.

<critical>TDD também vale para bugfix: escreva PRIMEIRO o teste de regressão que **falha reproduzindo o bug**, confirme que ele falha pelo motivo certo, e só então corrija</critical>
<critical>Corrija a CAUSA RAIZ, nunca o sintoma. Se a correção é um bloco de tratamento de erro engolindo a exceção ou uma verificação defensiva na camada de apresentação, provavelmente está na camada errada</critical>
<critical>Se `<ui_projeto>` indicar ferramenta de validação visual, bug visual se valida por teste visual mais verificação no ambiente real do produto — não por suposição</critical>
<critical>NUNCA regenere artefatos de teste visual para "resolver" uma falha sem aprovação explícita do usuário — isso apaga a evidência da regressão</critical>
<critical>Se a causa raiz estiver em área listada como read-only em `<limites_projeto>`, **não corrija**: documente o achado e sinalize ao time responsável</critical>
<critical>O bugfix NÃO está completo enquanto os comandos de `<comandos_projeto>` não passarem, incluindo o threshold de cobertura</critical>

## Localização dos arquivos

- PRD: `./tasks/prd-[nome-da-funcionalidade]/prd.md` (conforme `<artefatos_sdd>`)
- TechSpec: `./tasks/prd-[nome-da-funcionalidade]/techspec.md` (conforme `<artefatos_sdd>`)
- Tasks: `./tasks/prd-[nome-da-funcionalidade]/tasks.md` (conforme `<artefatos_sdd>`)
- Bugs: `./tasks/prd-[nome-da-funcionalidade]/bugs.md` (conforme `<artefatos_sdd>`)
- Relatório de Correções: `./tasks/prd-[nome-da-funcionalidade]/bugfixes.md` (conforme `<artefatos_sdd>`)
- Relatório de QA: `./tasks/prd-[nome-da-funcionalidade]/qa.md` (conforme `<artefatos_sdd>`)
- Evidências: pasta de evidências declarada em `<artefatos_sdd>`

Utilize o `nome-da-funcionalidade` como o <prd>

## Etapas para Executar

### 1. Análise de Contexto (Obrigatório)

- Ler o arquivo `bugs.md` e extrair TODOS os bugs documentados
- Ler o PRD para entender os requisitos afetados por cada bug
- Ler a TechSpec para entender as decisões técnicas relevantes
- Revisar `../_shared/SECURITY_BASELINE.md`, `../_shared/QUALITY_BASELINE.md` e as convenções de
  `<seguranca_extensoes>` / `<qualidade_extensoes>` do config para garantir conformidade nas correções
- Se `<integracoes_projeto>` indicar um rastreador de issues ativo e houver issue vinculada: leia-a
  pela ferramenta indicada no config para contexto adicional. Caso contrário, pule esta etapa.

<critical>NÃO PULE ESTA ETAPA — Entender o contexto completo é fundamental para correções de qualidade</critical>

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
bugs resolvidos. Se `<integracoes_projeto>` indicar um rastreador de issues ativo e houver issue
vinculada, registre pela ferramenta indicada no config um comentário com a lista de bugs corrigidos
e o resultado dos comandos de `<comandos_projeto>`. Caso contrário, pule esta etapa.

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
- [ ] Comentário postado na issue do rastreador configurado (se houver)

<critical>TDD também vale para bugfix: teste de regressão falhando PRIMEIRO, correção depois</critical>
<critical>Corrija a CAUSA RAIZ, nunca o sintoma</critical>
<critical>NUNCA regenere artefatos de teste visual sem aprovação explícita do usuário</critical>
<critical>O bugfix NÃO está completo enquanto os comandos de `<comandos_projeto>` não passarem</critical>
