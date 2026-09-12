# Tarefa X.0: [Título da tarefa]

## Visão geral

[Descrição breve da tarefa]

**Camada:** [`domain` / `data` / `presentation` / `módulo e rotas` / `integração` / `documentação`]

<skills>
### Conformidade com skills

[Pesquisar as skills aplicáveis em `.claude/skills/flutter/_INDEX.md` (File Match pelos arquivos que a tarefa toca, Keyword Match pelo tema) e listá-las abaixo. Carregar todas antes de implementar — ou usar `load_skills_for_files` do MCP `agent-skills-standard`.]
</skills>

<requirements>
[Lista de requisitos obrigatórios (usar os RFs quando aplicável)]
</requirements>

## Subtarefas

<critical>A subtarefa de teste vem SEMPRE antes da subtarefa de implementação correspondente (TDD). Para páginas, o golden test (light e dark) vem antes da página.</critical>

- [ ] X.1 [Teste falhando que descreve o comportamento esperado]
- [ ] X.2 [Implementação mínima para o teste passar]
- [ ] X.3 [Refactor / próximo comportamento]

## Detalhes de implementação

[Seções pertinentes da especificação técnica **NÃO É NECESSÁRIO MOSTRAR A IMPLEMENTAÇÃO COMPLETA, APENAS REFERENCIAR techspec.md**]

## Critérios de sucesso

- [Resultados mensuráveis]
- [Requisitos de qualidade]
- [ ] `make format-check analyze test` passa
- [ ] Cobertura da tarefa não derruba o gate de ≥90% (`make coverage`)

## Testes da tarefa

<critical>Antes de escrever ou refatorar qualquer teste, carregar `.claude/skills/flutter/flutter-test-behavior-audit/SKILL.md` (auditoria → mapa de cenários → implementação) e depois `flutter-tdd-testing`. `mocktail` é o único mock permitido. Nome no formato `should <expected> when <condition>`.</critical>

### Testes unitários (domain / data)

- [ ] ...
  <!-- usecase: sucesso e falha · repository: exceção → Failure correta (mocka o datasource) ·
       datasource: DioException real com DioExceptionType correto · model: fromJson completo,
       com campo ausente e com tipo inesperado -->

### Testes de Bloc/Cubit

- [ ] ...
  <!-- bloc_test por transição: carregando → sucesso · carregando → vazio · carregando → erro.
       Usar expect: orderedEquals([...]) quando a ordem importar. -->

### Testes de widget

- [ ] ...
  <!-- cada estado renderiza o esperado: conteúdo, vazio, erro no lugar certo, loading,
       botão desabilitado durante envio. pumpWidget com MaterialApp + BlocProvider.
       Sem bootstrap de Module e sem Modular.get<T>(). -->

### Testes golden (obrigatório para toda página nova)

- [ ] Golden em tema **claro**
- [ ] Golden em tema **escuro**
  <!-- Helper: pumpScreenWithTheme (test/_helpers/pump_screen_with_theme.dart), surface 390×844.
       tags: 'golden'. Arquivos em goldens/<tela>_light.png e goldens/<tela>_dark.png ao lado do teste.
       Escritos ANTES da implementação da página. Regenerar só com aprovação: flutter test --update-goldens.
       Atenção: goldens são excluídos no Linux (dart_test.yaml) — o gate é local no macOS. -->

### Testes de integração/E2E (se aplicável)

- [ ] ...
  <!-- integration_test/[feature]/ — cenário compartilhado em integration_test/scenarios/.
       Rodar com `make e2e-[feature] DEVICE=<id>`. Exige device + rede.
       Declarar aqui se o teste ESCREVE dados no ambiente DEV. -->

## Arquivos relevantes

**Implementação**

- [`lib/features/[feature]/[camada]/...`]

**Testes**

- [`test/features/[feature]/[camada]/..._test.dart`]
- [`test/features/[feature]/presentation/pages/goldens/*.png` (se houver página)]

**Módulo e rotas**

- [`lib/features/[feature]/[feature]_module.dart`]
