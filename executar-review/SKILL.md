---
name: executar-review
description: Revisa o diff da feature: conformidade arquitetural, gate de qualidade, segurança, aderência à TechSpec e convenções do projeto, e gera codereview.md. Use ao pedir code review ou revisar o código de uma feature.
---

<prd>`--prd` — o slug da feature (`<nome-da-feature>`). Se o usuário não informar, pergunte, ou liste as pastas de `<artefatos_sdd>` e peça para escolher</prd>
<template>`./references/TEMPLATE.md`</template>

<config>`../sdd.config.md`</config>
<baseline_seguranca>`../.sdd/_shared/SECURITY_BASELINE.md`</baseline_seguranca>
<baseline_qualidade>`../.sdd/_shared/QUALITY_BASELINE.md`</baseline_qualidade>
<regras_confianca>`../.sdd/_shared/REGRAS_DE_CONFIANCA.md`</regras_confianca>
<processo>`run.yaml` na pasta da feature (modelo em `../.sdd/_shared/RUN_TEMPLATE.md`)</processo>
<memoria>`memoria/executar-review.md` na pasta da feature (modelo em `../.sdd/_shared/MEMORIA_TEMPLATE.md`)</memoria>

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

Você é um assistente especializado em Code Review de aplicações de qualquer stack. Sua tarefa é analisar o **código produzido**, verificar se está de acordo com as regras e padrões do projeto, se o gate de qualidade passa, e se a implementação segue a TechSpec e as Tasks definidas.

<critical>O review não está completo até o gate de `<comandos_projeto>` passar inteiro, e não pode ser APROVADO com teste falhando ou cobertura abaixo do threshold. Nunca aprove uma redução do threshold como forma de passar o gate.</critical>
<critical>Qualquer alteração em área read-only de `<limites_projeto>` REPROVA o review</critical>
<critical>Confira a implementação contra `../.sdd/_shared/SECURITY_BASELINE.md` — achado de segurança de severidade alta ou crítica reprova o review</critical>

## Localização dos arquivos

- PRD: `./tasks/prd-<nome-da-feature>/prd.md` (conforme `<artefatos_sdd>`)
- TechSpec: `./tasks/prd-<nome-da-feature>/techspec.md` (conforme `<artefatos_sdd>`)
- Tasks: `./tasks/prd-<nome-da-feature>/tasks.md` (conforme `<artefatos_sdd>`)
- Bugs: `./tasks/prd-<nome-da-feature>/bugs.md` (conforme `<artefatos_sdd>`)
- Relatório de QA: `./tasks/prd-<nome-da-feature>/qa.md` (conforme `<artefatos_sdd>`)
- Relatório de Code Review: `./tasks/prd-<nome-da-feature>/codereview.md` (conforme `<artefatos_sdd>`)
- Processo: `./tasks/prd-<nome-da-feature>/run.yaml` (conforme `<artefatos_sdd>`)
- Memória desta skill: `./tasks/prd-<nome-da-feature>/memoria/executar-review.md` (conforme `<artefatos_sdd>`)
- Evidências: pasta de evidências declarada em `<artefatos_sdd>`

Use o mesmo `<nome-da-feature>` do <prd> para localizar todos os artefatos.

## Etapas do Processo

### 1. Delimitar o escopo e ler a documentação (Obrigatório)

Antes do diff, leia o <processo> e as memórias. O <processo> diz em que rodada de review a feature
está e o que a rodada anterior reprovou; as memórias das etapas anteriores dizem **por que** o
código é como é. Uma decisão registrada com motivo na `memoria/criar-techspec.md` não é achado de
review — é decisão tomada. Já uma **premissa em aberto** que ninguém fechou é achado, e as
alternativas descartadas evitam que você recomende de volta o que já foi avaliado e rejeitado.

Abra a entrada desta rodada no <processo> com `status: em_andamento`, `rodada` incrementada e os
11 passos `pendente`; se já houver entrada `em_andamento`, retome no primeiro passo não concluído.

**Delimite o diff antes de qualquer coisa** — é ele o objeto do review, não o repositório inteiro:

1. Use a branch base declarada em `<colaboracao_projeto>` (ex.: `git diff --stat <base>...HEAD`)
2. Se não houver convenção de branch declarada, use a branch principal do repositório
3. Se o trabalho ainda não foi commitado, some as alterações não commitadas ao diff
4. Se nada disso resolver, caia para os arquivos listados na seção "Arquivos relevantes" das tasks
   concluídas

Declare no relatório qual escopo foi usado e quantos arquivos ele cobre. Arquivo fora desse
escopo não é achado deste review.

Em seguida:

- Ler a TechSpec para entender as decisões arquiteturais esperadas
- Ler as Tasks para verificar o escopo implementado
- Revisar as regras invioláveis e as áreas read-only de `<limites_projeto>`
- Revisar `../.sdd/_shared/SECURITY_BASELINE.md`, `../.sdd/_shared/QUALITY_BASELINE.md` e as extensões
  (`<seguranca_extensoes>`, `<qualidade_extensoes>`) do config
- Revisar as convenções de `<colaboracao_projeto>` (commit, branch, PR, comentários no código)
- Se `<integracoes_projeto>` indicar um rastreador de issues ativo e houver issue vinculada, leia-a
  pela ferramenta indicada no config para os critérios de aceite. Caso contrário, pule esta etapa.

### 2. Conformidade arquitetural (obrigatório)

Revise **o diff da feature**, delimitado no passo 1 — não o repositório inteiro. Se o diff for
grande e a sua ferramenta permitir delegar, distribua os arquivos entre subagentes paralelos, cada
um com uma dimensão de revisão, e peça de volta apenas os achados (arquivo, linha, severidade,
descrição) — nunca o código revisado.

As regras invioláveis de `<limites_projeto>` são bloqueantes. Verifique:

- [ ] **Direção de dependência respeitada** — conforme a ordem de camadas de `<stack_projeto>`;
      nenhuma camada importa de quem depende dela
- [ ] **Camada de regra de negócio isolada** — sem dependência de framework de interface ou de
      biblioteca de acesso a dados, quando o padrão arquitetural de `<stack_projeto>` exigir isso
- [ ] **Erros tratados como parte do contrato** — conforme o campo "Tratamento de erro" de
      `<stack_projeto>`, sem falha engolida em silêncio nem exceção vazando fora do previsto
- [ ] **Pontos de entrada/rotas registrados** — conforme o campo "Roteamento / pontos de entrada"
      de `<stack_projeto>`
- [ ] **Injeção de dependência** — conforme o campo "Injeção de dependência" de `<stack_projeto>`,
      sem acesso global espalhado pelo código de produção
- [ ] **Logging pelo mecanismo declarado** — conforme o campo "Logging" de `<stack_projeto>`, sem
      saída direta para console no código de produção e sem dado sensível em claro
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
- [ ] Logging sem dado sensível, conforme `../.sdd/_shared/SECURITY_BASELINE.md`

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

### 7. Qualidade dos Testes (Obrigatório)

Cobertura alta com teste ruim não vale nada. Confira a implementação contra
`../.sdd/_shared/QUALITY_BASELINE.md`, seção "Qualidade dos testes", sem repetir a lista aqui:

- [ ] Todos os itens da seção "Qualidade dos testes" da baseline verificados
- [ ] Mock na fronteira certa, com a biblioteca declarada em `<stack_projeto>`

### 8. Análise de Qualidade de Código (Obrigatório)

Confira a implementação contra `../.sdd/_shared/QUALITY_BASELINE.md` (estrutura e complexidade, reuso,
nomenclatura, tratamento de erro, comentários, performance) e contra
`../.sdd/_shared/SECURITY_BASELINE.md` (segredos, dependências vulneráveis, injeção/validação,
autenticação/autorização, dados sensíveis, configuração segura), mais as extensões
`<qualidade_extensoes>` e `<seguranca_extensoes>` do config:

- [ ] Itens aplicáveis de `../.sdd/_shared/QUALITY_BASELINE.md` verificados
- [ ] Itens aplicáveis de `../.sdd/_shared/SECURITY_BASELINE.md` verificados
- [ ] Resultado do scanner de dependências vulneráveis de `<comandos_projeto>` conferido — achado
      de severidade alta ou crítica é bloqueante
- [ ] Acessibilidade e i18n verificadas, quando `<ui_projeto>` indicar que são aplicáveis

### 9. Convenções de Colaboração (Obrigatório)

Verifique a implementação contra `<colaboracao_projeto>`:

- [ ] Commits seguem a convenção declarada. Se `<colaboracao_projeto>` diz "livre" ou "sem
      convenção formal", ou se não houve commit no fluxo, registre N/A — não invente convenção
      para depois reprovar por ela
- [ ] Nome de branch e processo de PR seguem o declarado, quando aplicável ao fluxo do projeto
- [ ] Comentários no código seguem a política declarada — sem comentário redundante nem código
      morto/TODO órfão

### 10. Documentação (quando `<documentacao_projeto>` indicar registro obrigatório)

- [ ] Documentação da feature registrada no destino indicado em `<documentacao_projeto>`
- [ ] Registro externo obrigatório feito, quando `<documentacao_projeto>` indicar um

Se `<documentacao_projeto>` indicar que não há documentação obrigatória, pule esta etapa.

### 11. Relatório de Code Review (Obrigatório)

Gerar relatório final seguindo o formato definido em <template>.

Feche a entrada desta rodada no <processo>: `status: concluida`, `fim`, `veredito` (`rodada N —
APROVADO`, `APROVADO COM RESSALVAS` ou `REPROVADO`), os 11 passos com seu resultado — o
condicional 10 como `pulado` quando `<documentacao_projeto>` não exige registro — e
`proxima_acao`. Se o review REPROVOU, a próxima ação é corrigir e **rodar o review de novo**:
acrescente uma entrada nova na próxima rodada, não sobrescreva esta — veredito de review não
sobrevive a alteração de código feita depois dele.

Percorra o `definition_of_done` do <processo> item a item e marque `ok: true` só o que você
verificou nesta rodada. Item que continua `false` com o review APROVADO é contradição: ou o item
foi verificado e você não marcou, ou o veredito não se sustenta.

Registre na <memoria> desta skill: os achados que você decidiu **não** levantar e por quê, as
ressalvas não bloqueantes (para a próxima rodada não as redescobrir como novidade) e o que já foi
conferido e passou. Se `<integracoes_projeto>` indicar um rastreador de issues ativo e houver
issue vinculada, poste o veredito e os principais achados pela ferramenta indicada no config.
Caso contrário, pule esta etapa.

Antes de encerrar, confirme as duas escritas — não há checklist no fim desta skill para cobrá-las:
o `run.yaml` com a entrada da rodada fechada, o `definition_of_done` conferido e a próxima ação
apontada; e a `memoria/executar-review.md` com o que ficou de fora do relatório e por quê.

## Critérios de Aprovação

Os checkboxes dos passos 2 a 10 são a lista de verificação deste review — não existe uma segunda
lista no fim. O veredito sai deles.

**APROVADO**: Todos os critérios atendidos, gate de `<comandos_projeto>` verde com cobertura no
threshold declarado, código conforme regras invioláveis e TechSpec, convenções de
`<colaboracao_projeto>` respeitadas, documentação presente quando `<documentacao_projeto>` exigir.

**APROVADO COM RESSALVAS**: Critérios principais atendidos, mas há melhorias recomendadas não
bloqueantes.

**REPROVADO**: gate de `<comandos_projeto>` falhando, cobertura abaixo do threshold, violação de
direção de dependência ou de regra inviolável do config, alteração em área read-only, não
aderência à TechSpec, ou achado de segurança de severidade alta ou crítica.
