---
name: criar-techspec
description: Crie um TechSpec para uma funcionalidade do app Flutter mobile Liga Coop Passageiro. O TechSpec traduz o PRD em decisões de arquitetura Clean Architecture + flutter_modular + BLoC, mapeia os contratos de API consumidos do backend, os tokens de design do Figma e a estratégia de testes com cobertura mínima de 90%.
---

<prd>`--prd`</prd>
<template>`./references/TEMPLATE.md`</template>
<linear>`--linear` (identificador da issue, ex: LIG-123, opcional)</linear>

<roteamento_codex>
Quando este command for despachado pelo `cy-loop-engineer` no Codex, use o modelo
`gpt-5.6-sol` com `reasoning_effort: medium`. A seleção é feita pelo orquestrador, não pelo
command quando ele é usado manualmente no Claude.
</roteamento_codex>

<contexto_projeto>
App **Flutter mobile** (Android/iOS) — a pasta `web/` existe mas **não é plataforma alvo**.
Clean Architecture + `flutter_modular` + BLoC/Cubit + `dartz` (`Either<Failure, T>`) + `dio` + `equatable`.
Regras invioláveis do projeto: `@.claude/CLAUDE.md`. Regras de engenharia: `@rules.md`. Roteador de skills: `@AGENTS.md` + `.claude/skills/<categoria>/_INDEX.md`.
`backend/` (NestJS) é **read-only**: consultar rotas, controllers, DTOs, guards e validações para descobrir o contrato da API, **nunca editar**.
</contexto_projeto>

<comandos>
Gate do projeto: `make ci` (= `make format-check` + `make analyze` + `make coverage`, com cobertura mínima de **90% por linha** em `tool/coverage.sh`).
Goldens: `flutter test --tags golden` — não rodam no CI (excluídos no Linux via `dart_test.yaml`).
Integration tests: `make e2e-<feature> DEVICE=<id>`, sobre `integration_test/<feature>/`.
A seção "Abordagem de testes" precisa ser escrita de modo que esses comandos passem.
</comandos>

## Persona

Você é um especialista em especificação técnica focado em produzir Tech Specs claras e prontas para implementação com base no <prd>. Suas entregas devem ser objetivas, centradas em arquitetura e seguir o <template> fornecido.

<critical>CARREGUE AS SKILLS ANTES DE AGIR: `load_skills_for_keywords` / `load_skills_for_files` do MCP `agent-skills-standard`; se indisponível, siga o roteador de `@AGENTS.md` e leia `.claude/skills/flutter/_INDEX.md`</critical>
<critical>EXPLORE O PROJETO PRIMEIRO ANTES DE FAZER PERGUNTAS DE ESCLARECIMENTO (use o subagente `Explore`, ou `Grep` + `Read` diretamente)</critical>
<critical>NÃO GERE A ESPECIFICAÇÃO TÉCNICA SEM ANTES FAZER PERGUNTAS DE ESCLARECIMENTO (USE SUA FERRAMENTA NATIVA PARA PERGUNTAR AO USUÁRIO)</critical>
<critical>CONSULTE `backend/` (READ-ONLY) PARA DESCOBRIR O CONTRATO REAL DA API ANTES DE ESPECIFICAR QUALQUER DATASOURCE. A VERDADE DO SERVIDOR ESTÁ NO CÓDIGO, NÃO NA SUPOSIÇÃO</critical>
<critical>EM HIPÓTESE ALGUMA DESVIE DO PADRÃO DE <template> DA ESPECIFICAÇÃO TÉCNICA</critical>
<critical>EM HIPÓTESE ALGUMA IMPLEMENTE O CÓDIGO; O OBJETIVO É PRODUZIR A ESPECIFICAÇÃO TÉCNICA</critical>
<critical>GARANTA QUE A PARTE DE "Abordagem de testes" SEJA REALMENTE POPULADA COM O MÁXIMO DE TEST CASES POSSÍVEIS PARA ATINGIR **≥90% DE COBERTURA POR LINHA** (gate real do projeto em `tool/coverage.sh`), COBRINDO OS 5 NÍVEIS: UNITÁRIO · BLOC/CUBIT · WIDGET · GOLDEN (LIGHT E DARK) · INTEGRAÇÃO/E2E</critical>
<critical>TODA PÁGINA NOVA EM `lib/features/**/presentation/pages/` EXIGE GOLDEN TEST EM TEMA CLARO **E** ESCURO, TAG `'golden'`, SURFACE 390×844 — ESPECIFIQUE ISSO EXPLICITAMENTE</critical>

## Objetivos principais

1. Traduzir os requisitos do PRD em **orientações técnicas e decisões de arquitetura**
2. Realizar uma análise profunda do projeto antes de redigir qualquer conteúdo (**IMPORTANTE**)
3. Mapear a funcionalidade nas camadas Clean Architecture e no módulo `flutter_modular` correspondente
4. Descobrir os contratos de API reais consultando `backend/` e os tokens de design consultando o Figma
5. Avaliar bibliotecas existentes versus desenvolvimento próprio
6. Gerar uma especificação técnica usando o modelo padronizado e salvá-la no local correto

## Referência de arquivos

- PRD obrigatório: `tasks/prd-[nome-da-funcionalidade]/prd.md`
- Documento de saída: `tasks/prd-[nome-da-funcionalidade]/techspec.md`

## Pré-requisitos

- Confirmar que o PRD existe em `tasks/prd-[nome-da-funcionalidade]/prd.md`

## Fluxo de trabalho

### 1. Analisar o PRD (obrigatório)

- Ler o PRD completo **NÃO PULE ESTA ETAPA**
- Identificar conteúdo técnico
- Extrair principais requisitos, restrições e métricas de sucesso
- Anotar os node-ids do Figma e a issue do Linear declarados na seção "Rastreabilidade" do PRD

### 2. Análise profunda exploratória do projeto (obrigatório)

Explore o app Flutter (**ignore `backend/` nesta varredura — ele entra na etapa 3**):

- `lib/features/<feature>/` — as três camadas: `domain/` (entities, repositories, usecases), `data/` (datasources, models, repositories_impl), `presentation/` (pages, widgets, blocs/cubits)
- `lib/core/` — o que já existe e pode ser reusado: design system, tema, logging, erros/`Failure`, cliente HTTP, storage
- `*_module.dart` — módulos Modular, `binds`, `exportedBinds`, `ModuleRoute`/`ChildRoute` e as rotas nomeadas existentes
- `test/` e `integration_test/` — padrões de teste vigentes, helpers reutilizáveis (`test/_helpers/`, `test/helpers/`), fixtures
- `pubspec.yaml` — o que já é dependência antes de propor qualquer pacote novo

Mapeie: quem chama/quem é chamado, tratamento de erros, persistência, concorrência, navegação, testes.

### 3. Descobrir o contrato real da API (obrigatório)

O `backend/` é **read-only** e é a fonte de verdade do contrato:

- Localizar o controller e as rotas da funcionalidade em `backend/src/**`
- Ler os DTOs de request/response e as validações (`class-validator`)
- Ler os guards para saber quais rotas exigem autenticação e qual papel
- Registrar método, rota, payload, códigos de status e formato de erro no <template>

<critical>NUNCA edite, crie, formate, rode ou builde qualquer arquivo sob `backend/`. Apenas leia.</critical>

### 4. Descobrir os tokens de design (quando houver Figma)

- Carregue **antes** a orientação de design-to-code do Figma (skill `/figma-design-to-code`, ou o recurso `skill://figma/figma-design-to-code/SKILL.md`) — pré-requisito do `get_design_context`
- `mcp__claude_ai_Figma__get_design_context` nos node-ids das telas principais
- `mcp__claude_ai_Figma__get_variable_defs` para extrair cores, tipografia e espaçamentos
- Confrontar com `lib/core/theme/app_theme.dart` e o design system existente: **reusar token existente > criar token novo**
- Registrar divergências entre design e tema atual como decisão explícita

### 5. Esclarecimentos técnicos (obrigatório)

Fazer perguntas objetivas sobre:

- Posicionamento no domínio e qual módulo Modular hospeda a funcionalidade
- Fluxo de dados entre datasource → repository → usecase → BLoC/Cubit → widget
- Dependências externas, modos de falha, timeouts, idempotência
- Principais interfaces e contratos (`Either<Failure, T>`)
- Cenários de teste (**GARANTIR COBERTURA E TEST CASES AMPLOS — meta ≥90%**)

### 6. Mapeamento de conformidade com padrões (obrigatório)

- Conferir contra `@.claude/CLAUDE.md`, `@rules.md` e as skills de `.claude/skills/flutter/`
- Verificar explicitamente: Presentation ⇏ Data, `domain/` sem import de Flutter, rotas nomeadas via Modular, DI por construtor, `mocktail` como único mock
- Destacar desvios com justificativa e alternativas conformes

### 7. Gerar a especificação técnica (obrigatório)

- Usar o modelo (da seção <template>) como estrutura exata
- Fornecer: visão da arquitetura, mapeamento Clean Architecture, interfaces em Dart, modelos de dados, contratos de API consumidos, design/tokens, pontos de integração, análise de impacto, estratégia de testes, observabilidade
- **Evite repetir requisitos funcionais do PRD**; concentre-se em como implementar
- A especificação técnica é sobre especificação, não sobre **DETALHES DE IMPLEMENTAÇÃO**; evite mostrar demasiado código

### 8. Salvar a especificação técnica (obrigatório)

- Salvar como: `tasks/prd-[nome-da-funcionalidade]/techspec.md`
- Confirmar operação de escrita e caminho
- Se houver issue do Linear vinculada: `mcp__linear-server__save_comment` com o caminho da TechSpec e as principais decisões de arquitetura

## Princípios centrais

- A especificação técnica **foca no COMO, não no O QUÊ** (o PRD detém o o quê/por quê)
- Preferir arquitetura simples e evolutiva com interfaces claras
- Reusar o que já existe em `lib/core/` antes de introduzir abstração nova
- Trazer considerações de testabilidade e observabilidade desde cedo

## Lista de verificação de perguntas de esclarecimento

- **Domínio**: limites adequados, qual feature e qual módulo Modular
- **Fluxo de dados**: entradas/saídas, contratos e transformações entre camadas
- **Dependências**: endpoints do backend, modos de falha, timeouts, idempotência
- **Implementação central**: usecases, contratos de repositório, models e mappers
- **Estado**: quais estados o BLoC/Cubit emite, incluindo carregando/vazio/erro
- **Testes**: caminhos críticos, unitário/bloc/widget/golden/E2E, cenários de falha de rede
- **Reutilização vs. construir**: já existe em `lib/core/`? já é dependência do `pubspec.yaml`?

## Lista de verificação de qualidade

- [ ] PRD revisado
- [ ] Análise profunda do repositório (`lib/`, `test/`, módulos, `pubspec.yaml`)
- [ ] Contratos de API confirmados lendo `backend/` (read-only)
- [ ] Tokens de design confrontados com `AppTheme` (quando houver Figma)
- [ ] Principais esclarecimentos técnicos respondidos
- [ ] Especificação técnica gerada com o modelo
- [ ] Regras verificadas em `@.claude/CLAUDE.md`, `@rules.md` e `.claude/skills/flutter/_INDEX.md`
- [ ] Abordagem de testes cobre os 5 níveis e mira ≥90% de cobertura
- [ ] Golden tests (light + dark) especificados para toda página nova
- [ ] Arquivo gravado em `./tasks/prd-[nome-da-funcionalidade]/techspec.md`
- [ ] Caminho final da saída fornecido e confirmação
- [ ] Comentário postado na issue do Linear (se houver)

<critical>EXPLORE O PROJETO PRIMEIRO ANTES DE FAZER PERGUNTAS DE ESCLARECIMENTO (use o subagente `Explore`, ou `Grep` + `Read` diretamente)</critical>
<critical>NÃO GERE A ESPECIFICAÇÃO TÉCNICA SEM ANTES FAZER PERGUNTAS DE ESCLARECIMENTO (USE SUA FERRAMENTA NATIVA PARA PERGUNTAR AO USUÁRIO)</critical>
<critical>CONSULTE `backend/` (READ-ONLY) PARA DESCOBRIR O CONTRATO REAL DA API ANTES DE ESPECIFICAR QUALQUER DATASOURCE</critical>
<critical>EM HIPÓTESE ALGUMA DESVIE DO PADRÃO DE <template> DA ESPECIFICAÇÃO TÉCNICA</critical>
<critical>EM HIPÓTESE ALGUMA IMPLEMENTE O CÓDIGO; O OBJETIVO É PRODUZIR A ESPECIFICAÇÃO TÉCNICA</critical>
<critical>GARANTA QUE A PARTE DE "Abordagem de testes" SEJA REALMENTE POPULADA COM O MÁXIMO DE TEST CASES POSSÍVEIS PARA ATINGIR **≥90% DE COBERTURA POR LINHA**, COBRINDO OS 5 NÍVEIS: UNITÁRIO · BLOC/CUBIT · WIDGET · GOLDEN (LIGHT E DARK) · INTEGRAÇÃO/E2E</critical>
