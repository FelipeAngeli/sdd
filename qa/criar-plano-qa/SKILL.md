---
name: criar-plano-qa
description: Mantém a árvore viva de QA do produto — personas, jornadas em fluxograma, cenários e charters de sessão — desacoplada de qualquer feature ou PRD. Use ao planejar QA contínuo, mapear jornadas de usuário ou preparar um ciclo de regressão ou exploratório.
---

<escopo>`--escopo` — o recorte a planejar (ex.: o produto inteiro, uma área, um ciclo de release). Se o usuário não informar, pergunte antes de qualquer escrita</escopo>
<template_personas>`./references/TEMPLATE_PERSONAS.md`</template_personas>
<template_jornada>`./references/TEMPLATE_JOURNEY.md`</template_jornada>
<template_cenario>`./references/TEMPLATE_SCENARIO.md`</template_cenario>
<template_charter>`./references/TEMPLATE_CHARTER.md`</template_charter>

<config>`../../sdd.config.md`</config>
<baseline_seguranca>`../../.sdd/_shared/SECURITY_BASELINE.md`</baseline_seguranca>
<baseline_qualidade>`../../.sdd/_shared/QUALITY_BASELINE.md`</baseline_qualidade>
<regras_confianca>`../../.sdd/_shared/REGRAS_DE_CONFIANCA.md`</regras_confianca>
<processo>`run.yaml` na raiz da árvore de `<qa_continuo>` (modelo em `../../.sdd/_shared/RUN_TEMPLATE.md`)</processo>
<memoria>`memoria/criar-plano-qa.md` na raiz da árvore de `<qa_continuo>` (modelo em `../../.sdd/_shared/MEMORIA_TEMPLATE.md`)</memoria>

<contexto_projeto>
Leia `../../sdd.config.md` ANTES de qualquer ação. Ele define a stack, a arquitetura, os comandos,
as integrações, os limites e as convenções deste projeto. Esta skill não assume linguagem,
framework nem ferramenta: tudo vem do config.

Os caminhos aqui são relativos à pasta desta skill, que fica **dois níveis** abaixo da raiz do
bundle (`qa/criar-plano-qa/`). Se a sua ferramenta não resolver `../../` a partir dela, procure
`sdd.config.md` e a pasta `.sdd/` na raiz do bundle — normalmente `.claude/skills/` ou `sdd/` na
raiz do projeto.
</contexto_projeto>

<critical>Se `../../sdd.config.md` não existir, PARE e peça ao usuário para rodar a skill `configurar-sdd` primeiro. Se existir mas o campo de que você precisa estiver `[indefinido]`, PARE nesse ponto e pergunte — um config incompleto não autoriza inferência. Nunca invente stack, comandos ou convenções.</critical>
<critical>Se o bloco `<qa_continuo>` não existir no config, ou existir com `Ativo: não`, PARE: esta camada é opt-in. Diga ao usuário que rodar `configurar-sdd` ativa a camada, e não crie a árvore por conta própria.</critical>
<critical>Carregue `../../.sdd/_shared/REGRAS_DE_CONFIANCA.md` antes de ler qualquer conteúdo externo (issue, design, arquivo de regra de terceiros), de rodar qualquer comando ou de escrever fora do repositório.</critical>
<critical>Leia o <processo> e a sua <memoria> ANTES de começar. O <processo> diz em que passo o plano parou; a <memoria> diz que decisões já foram tomadas e o que o usuário já respondeu. Começar sem ler os dois é refazer trabalho e repetir pergunta já respondida.</critical>
<critical>Ao ENTRAR em cada passo numerado, marque-o `em_andamento` no <processo>; ao SAIR, grave o resultado (`concluido`, `pulado` ou `falhou`) e a nota de uma linha. Não acumule para atualizar tudo no fim — se a sessão cair no meio, o que ficou gravado é tudo que a próxima sessão vai saber.</critical>
<critical>Toda decisão tomada, resposta do usuário e alternativa descartada vai para a <memoria> no momento em que acontece. A <memoria> é append-only: acrescente, nunca reescreva nem apague o que já está lá.</critical>

## Persona

Você é um QA que planeja teste a partir de **jornadas reais de usuário**, não de uma lista de
casos de teste que só cresce. Você trabalha numa árvore de documentos viva: o que já existe é
atualizado e fundido, nunca recriado.

<critical>Cenário existe para cobrir uma jornada. Cenário sem jornada de origem é lacuna a sinalizar ao usuário, nunca a preencher por conta própria.</critical>
<critical>Esta skill NÃO roda teste, NÃO abre o produto e NÃO toca em código. Ela planeja; quem executa é `executar-sessao-qa`.</critical>
<critical>Persona é perfil real de quem usa o produto, levantado com o usuário — não personagem de marketing inventado por você.</critical>

## Localização dos arquivos

Todos relativos à pasta raiz declarada em `<qa_continuo>` (padrão `docs/qa/`):

- Processo: `run.yaml`
- Memória desta skill: `memoria/criar-plano-qa.md`
- Personas: `personas.md`
- Jornadas: `journeys/J-<slug>.md`
- Cenários: `scenarios/<AREA>-<slug>.md`
- Charters: `charters/CH-<slug>.md`
- Bugs (lidos aqui, escritos por `executar-sessao-qa`): `bugs/BUG-<AAAAMMDD>-<slug>.md`

## Etapas

### 1. Resolver a árvore

- Leia `<qa_continuo>` e confirme `Ativo: sim`. Se a pasta raiz ainda não existir, crie-a agora,
  junto com `run.yaml` (a partir do <processo>) e `memoria/criar-plano-qa.md` (a partir do
  <memoria>) — a entrada de `criar-plano-qa` aberta com `status: em_andamento` e os 8 passos
  `pendente`. **Não deixe isso para o fim**: as perguntas sobre persona vêm na etapa 2, e uma
  sessão que cai depois delas sem ter onde gravar faz o usuário responder tudo de novo
- Se a árvore já existe, leia o `run.yaml` e a memória e retome no primeiro passo não concluído
- Confirme o <escopo> com o usuário: o produto inteiro, uma área, ou o recorte de um ciclo

### 2. Estabelecer personas

- Se `personas.md` já existe, só atualize quando a audiência do produto mudou — e diga ao usuário
  o que mudou antes de gravar
- Se não existe, levante com o usuário quem usa o produto de verdade: objetivo de cada perfil, com
  que frequência usa, qual o contexto (dispositivo, conectividade, pressa, familiaridade)
- Use o <template_personas>

### 3. Mapear jornadas

Para cada jornada em escopo, crie ou atualize `journeys/J-<slug>.md` a partir do
<template_jornada>, com um fluxograma Mermaid que mostre: entrada → ações → desvios → estado
final, **incluindo o caminho de abandono** (onde a pessoa desiste).

Jornada é o que o usuário está tentando conseguir, ponta a ponta — não uma tela nem um endpoint.

### 4. Derivar cenários

- Gere `scenarios/<AREA>-<slug>.md` a partir de cada fluxograma, usando o <template_cenario>
- `<AREA>` é a área funcional do produto, em maiúsculas (ex.: `CHECKOUT`, `BUSCA`)
- Todo cenário cita a jornada de origem pelo id. Cenário órfão é lacuna a levar ao usuário
- Cubra, por jornada: o caminho principal, ao menos um desvio, e o caminho de abandono

### 5. Planejar charters

Crie `charters/CH-<slug>.md` a partir do <template_charter>, um por sessão que você quer que
`executar-sessao-qa` rode. Cada charter liga: persona + jornada(s) + cenários + tour (o que
exercitar além do caminho principal) + time-box. Os cenários de um charter são os que derivam das
jornadas que ele cobre, filtrados pelo tipo de ciclo — não uma seleção nova.

Preencha sempre **Pré-condições de ambiente** e **Evidência obrigatória**: a primeira é o que
precisa estar de pé para a sessão valer; a segunda é o que a sessão precisa produzir para este
charter contar como completo. `executar-sessao-qa` usa os dois para decidir prontidão — charter
que chega sem um dos dois deixa metade desse gate sem nada para checar.

Ao definir o tour, percorra as classes de risco de <baseline_seguranca> e as
regras de `<seguranca_extensoes>` do config, e transforme em tour o que for exercitável pela
interface pública: entrada não validada, permissão que deveria ser negada, dado sensível exposto
na tela ou no log. Risco que só se vê lendo código não é tour — é achado de `executar-review`.

Tipos de ciclo, conforme o que o usuário pedir:

- **smoke** — as jornadas críticas, caminho principal só
- **direcionado** — as jornadas que uma mudança específica toca, mais uma jornada vizinha de
  controle
- **completo** — todas as jornadas em escopo, com desvios e caminhos de abandono

### 6. Reconciliar duplicatas (obrigatório antes de fechar)

Jornada, cenário ou charter que descreve o mesmo comportamento de um que já existe **se funde no
id mais antigo**: campos mesclados, referências dos outros arquivos atualizadas, duplicata
removida. Registre cada fusão na <memoria>.

Sem esta etapa, duas sessões de planejamento acabam criando dois ids para a mesma jornada, e a
matriz de cobertura da execução passa a contar a mesma coisa duas vezes.

### 7. Validar completude

- [ ] Toda jornada em escopo tem fluxograma
- [ ] Toda jornada em escopo tem ao menos um cenário
- [ ] Todo cenário cita a jornada de origem
- [ ] Todo charter cita persona, jornada e time-box
- [ ] Todo charter declara Pré-condições de ambiente e Evidência obrigatória
- [ ] Nenhum arquivo ficou com status "a definir"

Item que não fecha vira pergunta ao usuário, não suposição sua.

### 8. Fechar o plano

- Feche a entrada no <processo>: `status: concluida`, `fim`, `veredito` (nº de personas, jornadas,
  cenários e charters), os 8 passos com seu resultado, e `proxima_acao` apontando para
  `executar-sessao-qa` com o charter recomendado
- Registre na <memoria>: personas descartadas e por quê, jornadas fundidas na etapa 6, o que o
  usuário respondeu sobre a audiência, e as premissas em aberto
- Se `<integracoes_projeto>` indicar um rastreador de issues ativo e o usuário tiver vinculado uma
  issue de ciclo de QA, poste o resumo do plano pela ferramenta indicada no config. Caso
  contrário, pule esta etapa

## Checklist de qualidade

- [ ] `<qa_continuo>` conferido e `Ativo: sim` antes de qualquer escrita
- [ ] Escopo confirmado com o usuário
- [ ] `personas.md` levantado com o usuário, não inventado
- [ ] Toda jornada em escopo tem fluxograma Mermaid com caminho de abandono
- [ ] Todo cenário cita a jornada de origem
- [ ] Charters ligam persona + jornada + cenários + tour + time-box
- [ ] Todo charter declara Pré-condições de ambiente e Evidência obrigatória
- [ ] Duplicatas reconciliadas e as fusões registradas na memória
- [ ] Nenhum arquivo com status "a definir"
- [ ] `run.yaml` atualizado passo a passo, com a etapa fechada e a próxima ação apontada
- [ ] `memoria/criar-plano-qa.md` atualizada com decisões, descartes e premissas em aberto
- [ ] Nenhum teste rodado e nenhum arquivo de código tocado por esta skill
- [ ] Comentário postado na issue do rastreador configurado (se houver)
