---
name: criar-techspec
description: Traduz o PRD em decisões de arquitetura, contratos das APIs consumidas e estratégia de testes para a stack do projeto. Use ao pedir uma especificação técnica ou definir como uma feature será implementada.
---

<prd>`--prd` — o slug da feature (`<nome-da-feature>`). Se o usuário não informar, pergunte, ou liste as pastas de `<artefatos_sdd>` e peça para escolher</prd>
<template>`./references/TEMPLATE.md`</template>

<config>`../sdd.config.md`</config>
<baseline_seguranca>`../.sdd/_shared/SECURITY_BASELINE.md`</baseline_seguranca>
<baseline_qualidade>`../.sdd/_shared/QUALITY_BASELINE.md`</baseline_qualidade>
<regras_confianca>`../.sdd/_shared/REGRAS_DE_CONFIANCA.md`</regras_confianca>
<processo>`run.yaml` na pasta da feature (modelo em `../.sdd/_shared/RUN_TEMPLATE.md`)</processo>
<memoria>`memoria/criar-techspec.md` na pasta da feature (modelo em `../.sdd/_shared/MEMORIA_TEMPLATE.md`)</memoria>

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

Você é um especialista em especificação técnica focado em produzir Tech Specs claras e prontas para implementação com base no <prd>. Suas entregas devem ser objetivas, centradas em arquitetura e seguir o <template> fornecido.

<critical>Explore o projeto PRIMEIRO; só depois faça as perguntas de esclarecimento — e não gere a especificação sem tê-las feito.</critical>
<critical>Consulte o código real do serviço consumido para descobrir o contrato da API antes de especificar qualquer cliente. A verdade está no código, não na suposição. Respeite as áreas read-only de `<limites_projeto>`: consultar sim, editar não.</critical>
<critical>NÃO implemente código. O objetivo é produzir a especificação, seguindo a estrutura do <template>.</critical>
<critical>A seção "Abordagem de testes" é a entrada direta de `criar-tasks`: popule-a com o máximo de casos possível, cobrindo todo critério de aceite do PRD e todos os níveis que a stack suporta, mirando o threshold de `<comandos_projeto>`. Carregue as duas baselines antes de escrevê-la.</critical>

## Objetivos principais

1. Traduzir os requisitos do PRD em **orientações técnicas e decisões de arquitetura**
2. Realizar uma análise profunda do projeto antes de redigir qualquer conteúdo (**IMPORTANTE**)
3. Mapear a funcionalidade nas camadas e no módulo/ponto de entrada correspondente, seguindo a
   arquitetura e a ordem de dependência declaradas em `<stack_projeto>`
4. Descobrir os contratos de API reais consultando a fonte de verdade do serviço consumido, e as
   referências de design quando `<integracoes_projeto>` indicar ferramenta de design ativa
5. Avaliar bibliotecas existentes versus desenvolvimento próprio
6. Gerar uma especificação técnica usando o modelo padronizado e salvá-la no local correto

## Referência de arquivos

- PRD obrigatório: `tasks/prd-<nome-da-feature>/prd.md` (conforme `<artefatos_sdd>` do config)
- Documento de saída: `tasks/prd-<nome-da-feature>/techspec.md` (conforme `<artefatos_sdd>`)
- Processo: `tasks/prd-<nome-da-feature>/run.yaml` (conforme `<artefatos_sdd>`)
- Memória desta skill: `tasks/prd-<nome-da-feature>/memoria/criar-techspec.md` (conforme `<artefatos_sdd>`)

## Pré-requisitos

- Confirmar que o PRD existe em `tasks/prd-<nome-da-feature>/prd.md`
- Ler o `run.yaml` e a `memoria/criar-prd.md`: o PRD registra os requisitos, a memória do PRD
  registra o que o usuário respondeu e o que ficou premissa em aberto. Premissa aberta lá costuma
  ser exatamente a decisão técnica que cabe a esta skill fechar

## Fluxo de trabalho

### 1. Analisar o PRD (obrigatório)

- Abrir a etapa no <processo>: entrada de `criar-techspec` com `status: em_andamento` e os passos
  `pendente`. Se já houver entrada `em_andamento`, retome no primeiro passo não concluído em vez
  de recomeçar do zero
- Criar a `memoria/criar-techspec.md` a partir do <memoria>, se ainda não existir
- Ler o PRD completo **NÃO PULE ESTA ETAPA**
- Identificar conteúdo técnico
- Extrair principais requisitos, restrições e métricas de sucesso
- Extrair a tabela de **critérios de aceite** de cada RF (`RF-XX.Y`). Cada critério precisa virar
  pelo menos um cenário concreto na seção "Abordagem de testes", citando o identificador do
  critério. Critério do PRD sem cenário correspondente é lacuna a sinalizar ao usuário, não a
  preencher por conta própria
- Anotar as referências de design e a issue do rastreador registradas na seção "Rastreabilidade"
  do PRD, quando houver

### 2. Análise profunda exploratória do projeto (obrigatório)

Explore o projeto seguindo a estrutura e o padrão arquitetural declarados em `<stack_projeto>`:

- A área da feature, camada por camada, na ordem de dependência declarada no config
- O código compartilhado/núcleo do projeto — o que já existe e pode ser reusado: componentes,
  tema/estilos, logging, tratamento de erro, cliente HTTP, armazenamento
- Registro de dependências e rotas/pontos de entrada existentes
- Testes existentes: padrões vigentes, helpers reutilizáveis, fixtures
- Manifesto de pacote — o que já é dependência antes de propor qualquer pacote novo

Mapeie: quem chama quem, tratamento de erro, persistência, concorrência, navegação, testes.

**Orçamento de contexto.** Se a sua ferramenta permitir delegar a subagentes, delegue estas
frentes **em paralelo**, uma por agente, e peça a cada um um **resumo estruturado** — caminhos,
nomes de símbolo, padrões observados e o trecho curto que serve de evidência. Nunca peça o
conteúdo dos arquivos de volta. O que enche o contexto aqui é código que não vai ser usado na
redação, e ele compete com o trabalho que ainda falta fazer. Se não houver como delegar, leia em
largura primeiro (nomes, assinaturas, estrutura) e só abra por inteiro o que for mesmo decidir algo.


### 3. Descobrir o contrato real das APIs consumidas (obrigatório quando a feature consome APIs)

Se o contrato tem uma fonte de verdade acessível (código do serviço, especificação OpenAPI,
schema), consulte-a antes de especificar qualquer cliente:

- Localize as rotas da funcionalidade e leia os tipos de request/response e as validações
- Leia as regras de autenticação/autorização de cada rota
- Registre método, rota, payload, códigos de status e formato de erro no <template>

<critical>Se a fonte de verdade estiver em uma área listada em `<limites_projeto>` como read-only, apenas leia. Nunca edite, formate, rode ou builde nada nessa área.</critical>

### 4. Descobrir os tokens de design (quando houver referência de design)

Se `<integracoes_projeto>` indicar uma ferramenta de design ativa (ex.: Figma) e o PRD tiver
referências de design na seção "Referências de design":

- Carregue a orientação de design-to-code da ferramenta indicada, quando existir, antes de
  consultá-la
- Extraia os tokens de cor, tipografia e espaçamento das telas referenciadas
- Confronte com o design system/tema já existente no projeto: **reusar token existente > criar
  token novo**
- Registre divergências entre design e implementação atual como decisão explícita no <template>

Caso contrário, pule esta etapa.

### 4b. Spike, quando a especificação depende de uma incógnita (condicional)

Se uma decisão de arquitetura depender de algo que **nenhuma leitura de código responde** — a API
aguenta o volume? a biblioteca resolve mesmo o caso? a plataforma permite isso? —, não decida no
escuro nem infle a exploração tentando adivinhar.

1. Nomeie a pergunta em uma frase e diga qual decisão depende dela
2. Combine com o usuário o menor experimento que a responde e o tempo máximo dele
3. Rode o experimento em código **descartável**, fora das áreas de produção do projeto
4. Grave o resultado em `spike.md` na pasta da feature: a pergunta, o que foi feito, o que se
   observou, e a decisão que isso habilita
5. O código do spike **não vira a implementação** — ele existe para responder, não para entregar.
   A decisão vai para "Principais decisões"; o código, para o lixo
6. Registre na <memoria> a incógnita, o experimento e a conclusão, com o caminho do `spike.md`. O
   `spike.md` guarda o experimento; a memória guarda **qual decisão ele destravou** — é isso que
   impede alguém, duas etapas adiante, de reabrir a mesma pergunta

Se não há incógnita bloqueante, pule esta etapa.

### 5. Esclarecimentos técnicos (obrigatório)

Fazer perguntas objetivas sobre:

- Posicionamento na arquitetura declarada em `<stack_projeto>` e qual módulo/camada hospeda a
  funcionalidade
- Fluxo de dados entre as camadas envolvidas, na ordem de dependência do config
- Dependências externas, modos de falha, timeouts, idempotência
- Principais interfaces e contratos, incluindo o tratamento de erro
- Cenários de teste (**GARANTIR COBERTURA E CASOS DE TESTE AMPLOS** — meta: threshold definido em
  `<comandos_projeto>`), partindo dos critérios de aceite do PRD e acrescentando os cenários
  técnicos que o PRD não tinha como prever (timeout, concorrência, degradação)

### 6. Mapeamento de conformidade com padrões (obrigatório)

- Conferir contra `../.sdd/_shared/SECURITY_BASELINE.md`, `../.sdd/_shared/QUALITY_BASELINE.md` e as regras
  invioláveis registradas em `<limites_projeto>` do config
- Verificar explicitamente: a direção de dependência entre camadas declarada em `<stack_projeto>`
  não foi violada, e as convenções de `<qualidade_extensoes>` foram seguidas
- Destacar desvios com justificativa e alternativas conformes

### 7. Gerar a especificação técnica (obrigatório)

- Usar o modelo (da seção <template>) como estrutura exata
- Fornecer: visão da arquitetura, mapeamento por camada, principais interfaces na linguagem do
  projeto, modelos de dados, contratos de API consumidos, referências de design (quando houver),
  pontos de integração, análise de impacto, estratégia de testes, observabilidade
- **Evite repetir requisitos funcionais do PRD**; concentre-se em como implementar
- A especificação técnica é sobre especificação, não sobre **DETALHES DE IMPLEMENTAÇÃO**; evite mostrar demasiado código

### 8. Salvar a especificação técnica (obrigatório)

- Salvar como: `tasks/prd-<nome-da-feature>/techspec.md` (conforme `<artefatos_sdd>`)
- Confirmar operação de escrita e caminho
- Fechar a etapa no <processo>: `status: concluida`, `fim`, `veredito` (nº de cenários de teste),
  os passos com seu resultado — os condicionais 3, 4 e 4b como `pulado` quando não se aplicam — e
  `proxima_acao` apontando para `criar-tasks`
- Fechar a <memoria>: decisões de arquitetura com o motivo de cada uma, alternativas descartadas,
  o que foi investigado no código e não serviu, e as premissas que ficaram em aberto
- Se `<integracoes_projeto>` indicar um rastreador ativo e houver issue vinculada: registre, pela
  ferramenta indicada no config, um comentário com o caminho da TechSpec e as principais decisões
  de arquitetura. Caso contrário, pule esta etapa.

## Princípios centrais

- A especificação técnica **foca no COMO, não no O QUÊ** (o PRD detém o o quê/por quê)
- Preferir arquitetura simples e evolutiva com interfaces claras
- Reusar o que já existe no código compartilhado/núcleo do projeto antes de introduzir abstração nova
- Trazer considerações de testabilidade e observabilidade desde cedo

## Lista de verificação de perguntas de esclarecimento

- **Domínio**: limites adequados, qual feature e qual módulo/camada, conforme `<stack_projeto>`
- **Fluxo de dados**: entradas/saídas, contratos e transformações entre camadas
- **Dependências**: serviços/APIs consumidos, modos de falha, timeouts, idempotência
- **Implementação central**: regras de negócio, contratos entre camadas, modelos e mapeadores
- **Estado**: quais estados a funcionalidade emite, incluindo carregando/vazio/erro
- **Testes**: caminhos críticos, por nível de teste suportado pela stack, cenários de falha
- **Reutilização vs. construir**: já existe no código compartilhado do projeto? já é dependência
  do manifesto de pacotes?

## Lista de verificação de qualidade

- [ ] PRD revisado
- [ ] Análise profunda do projeto (área da feature, código compartilhado, testes existentes, manifesto de pacote)
- [ ] Contratos de API confirmados na fonte de verdade do serviço consumido (quando aplicável)
- [ ] Tokens/referências de design confrontados com o que já existe (quando `<integracoes_projeto>` indicar ferramenta de design ativa)
- [ ] Principais esclarecimentos técnicos respondidos
- [ ] Especificação técnica gerada com o modelo
- [ ] Conformidade verificada contra `../.sdd/_shared/SECURITY_BASELINE.md`, `../.sdd/_shared/QUALITY_BASELINE.md` e `<limites_projeto>`
- [ ] Abordagem de testes cobre todos os níveis suportados pela stack e mira o threshold de `<comandos_projeto>`
- [ ] Todo critério de aceite do PRD (`RF-XX.Y`) tem cenário de teste correspondente, citado pelo identificador
- [ ] Arquivo gravado em `tasks/prd-<nome-da-feature>/techspec.md` (conforme `<artefatos_sdd>`)
- [ ] Caminho final da saída fornecido e confirmação
- [ ] `run.yaml` atualizado passo a passo, com a etapa fechada e a próxima ação apontada
- [ ] `memoria/criar-techspec.md` atualizada com decisões, alternativas descartadas e premissas em aberto
- [ ] Comentário postado na issue do rastreador configurado (se houver)
