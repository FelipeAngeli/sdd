# Especificação técnica

## Resumo executivo

[Fornecer uma visão técnica breve da abordagem da solução. Resumir as principais decisões de arquitetura e a estratégia de implementação em 1–2 parágrafos.]

## Arquitetura do sistema

### Visão dos componentes

[Descrição breve dos principais componentes e suas responsabilidades:

- Nomes dos componentes e funções principais **Certifique-se de listar cada componente novo ou modificado**
- Principais relacionamentos entre componentes
- Visão geral do fluxo de dados]

### Mapeamento Clean Architecture

<critical>
**OBRIGATÓRIO.** Toda funcionalidade precisa declarar em qual camada cada arquivo vive. Presentation **nunca** acessa Data diretamente; `domain/` é Dart puro (sem `package:flutter`, sem `dio`, sem Modular).
</critical>

| Camada | Arquivo | Responsabilidade | Novo ou modificado |
| --- | --- | --- | --- |
| `domain/entities` | `lib/features/[feature]/domain/entities/[nome]_entity.dart` | [Entidade pura, `Equatable`] | novo/modificado |
| `domain/repositories` | `lib/features/[feature]/domain/repositories/[nome]_repository.dart` | [Contrato abstrato retornando `Either<Failure, T>`] | novo/modificado |
| `domain/usecases` | `lib/features/[feature]/domain/usecases/[acao]_usecase.dart` | [Uma regra de negócio por usecase] | novo/modificado |
| `data/models` | `lib/features/[feature]/data/models/[nome]_model.dart` | [`fromJson`/`toJson` + mapper para entity] | novo/modificado |
| `data/datasources` | `lib/features/[feature]/data/datasources/[nome]_remote_datasource.dart` | [Chamadas `dio`; lança exceções tipadas] | novo/modificado |
| `data/repositories` | `lib/features/[feature]/data/repositories/[nome]_repository_impl.dart` | [Traduz exceção → `Failure`] | novo/modificado |
| `presentation/bloc` | `lib/features/[feature]/presentation/bloc/[nome]_cubit.dart` | [Estados: inicial, carregando, sucesso, vazio, erro] | novo/modificado |
| `presentation/pages` | `lib/features/[feature]/presentation/pages/[nome]_page.dart` | [Tela — **exige golden test light + dark**] | novo/modificado |
| `presentation/widgets` | `lib/features/[feature]/presentation/widgets/[nome]_widget.dart` | [Componente reutilizável] | novo/modificado |
| módulo | `lib/features/[feature]/[feature]_module.dart` | [`binds` + `ChildRoute` com rota nomeada] | novo/modificado |

### Navegação e injeção de dependência

[- Rotas nomeadas registradas (via `flutter_modular`) — **sem `MaterialPageRoute` inline e sem instanciar widget no ponto de navegação**
- Binds novos e seu escopo (`addLazySingleton` / `addSingleton` / `addInstance`)
- Guards de rota aplicáveis
- Módulo pai que importa/exporta esses binds]

### Reuso de `lib/core/`

[O que já existe e será reusado em vez de recriado: design system, `AppTheme`, cliente HTTP, `Failure`s, logger, storage seguro. **Justifique cada componente novo em `core/`.**]

## Design de implementação

### Principais interfaces

[Definir os principais contratos em **Dart** (≤20 linhas por exemplo):

```dart
// Contrato de repositório — domain/ é Dart puro
abstract interface class NomeRepository {
  Future<Either<Failure, NomeEntity>> buscar(String id);
}

// Usecase — uma responsabilidade
class BuscarNomeUsecase {
  const BuscarNomeUsecase(this._repository);
  final NomeRepository _repository;

  Future<Either<Failure, NomeEntity>> call(String id) => _repository.buscar(id);
}
```

]

### Estados do BLoC/Cubit

[Declarar a máquina de estados. Todo fluxo precisa cobrir carregando, sucesso, **vazio** e **erro** — não só o caminho feliz.

| Estado | Quando é emitido | O que a UI mostra |
| --- | --- | --- |
| `[NomeInitial]` | [Condição] | [Comportamento visual] |
| `[NomeLoading]` | [Condição] | [Indicador de carregamento, botão desabilitado] |
| `[NomeSuccess]` | [Condição] | [Conteúdo] |
| `[NomeEmpty]` | [Condição] | [Estado vazio] |
| `[NomeError]` | [Condição] | [Mensagem e ação de retry] |
]

### Modelos de dados

<critical>
**OBRIGATÓRIO — formatação visual e legível:**

- **NÃO** descrever contratos como tipos inline em bullet points (ex.: `location: { name, admin1, ... }`).
- **SEMPRE** documentar cada entidade/contrato em subseção própria (`#### \`NomeDoTipo\` — descrição`).
- **SEMPRE** incluir tabela de campos (Campo | Tipo | Obrigatório | Descrição) antes do JSON de exemplo.
- **SEMPRE** incluir bloco ` ```json ` formatado e indentado com valores realistas (não placeholders genéricos).
- **SEMPRE** indicar o tipo Dart correspondente e se o campo é nullable no model.
- Variantes/degradações (ex.: campo `null`) devem ter JSON separado + blockquote explicando o comportamento.
- Envelopes de erro: tabela Código | HTTP | `Failure` correspondente + exemplo JSON.
- Mapeamentos JSON → model → entity: tabela Origem | Destino.
- Campos ausentes no payload normalizados para `null` — mencionar isso no parágrafo introdutório.
</critical>

Contratos JSON consumidos da API — prontos para mapear em models e exibir na UI. [Completar contexto específico da feature. Campos ausentes no payload são normalizados para `null`.]

#### `[NomeDoTipo]` — [descrição curta]

| Campo | Tipo JSON | Tipo Dart | Obrigatório | Descrição |
| --- | --- | --- | --- | --- |
| `[campo]` | `[string/number/...]` | `[String/int/DateTime?]` | sim/não | [Descrição] |

```json
{
  "[campo]": "[valor realista]"
}
```

[Repetir o padrão acima para cada entidade/contrato principal: payload agregado, tipos de entrada, tipos de erro, etc.]

> **[Variante/degradação (se aplicável)]:** [Explicar quando ocorre e o impacto no payload.]

```json
{
  "[secao_afetada]": null
}
```

#### `[NomeDoErro]` — envelope de erro tipado

| Código | HTTP | `Failure` no app | Significado |
| --- | --- | --- | --- |
| `[codigo]` | `[status]` | `[ServerFailure/ValidationFailure/AuthFailure]` | [Descrição] |

```json
{
  "error": {
    "code": "[codigo]",
    "message": "[mensagem conforme padrão do backend]"
  }
}
```

#### Mapeamento JSON → model → entity

| Origem (payload) | Destino (model) | Destino (entity) |
| --- | --- | --- |
| `[campo_origem]` | `[campoModel]` | `[campoEntity]` |

## Contratos de API consumidos

<critical>
O `backend/` (NestJS) é **read-only** e é a **fonte de verdade** deste contrato. Confirme cada rota lendo o controller e o DTO em `backend/src/**` antes de documentar aqui — não suponha.
</critical>

<critical>
**OBRIGATÓRIO — formatação visual e legível:**

- **NÃO** listar rotas como bullet points compactos com tudo inline.
- **SEMPRE** começar com tabela de visão geral (Método | Rota | Autenticação | Datasource que consome).
- **SEMPRE** documentar cada rota em subseção própria (`#### \`MÉTODO /rota\``).
- **SEMPRE** incluir, por rota: tabela de query params/body, tabela de respostas (Status | Corpo | `Failure` resultante | Quando), bloco ` ```http ` com a requisição e blocos ` ```json ` com exemplos de resposta.
- Cobrir **todos** os cenários relevantes: sucesso, lista vazia, erro de validação, erro do servidor, timeout, sem conectividade.
- Usar blockquote (`>`) para comportamentos não óbvios (ex.: "lista vazia não é erro HTTP — a UI mostra estado vazio").
- Separar rotas com `---`.
</critical>

### Visão geral

| Método | Rota | Autenticação | Datasource que consome | Origem no backend |
| --- | --- | --- | --- | --- |
| `[GET/POST/...]` | `[ /rota ]` | [Bearer/pública] | `[nome]_remote_datasource.dart` | `backend/src/[modulo]/[nome].controller.ts` |

---

#### `[MÉTODO] [ /rota ]`

[Descrição breve do propósito da rota.]

**Query params** _(ou **Body** para POST/PUT/PATCH)_

| Param | Tipo | Default | Regras (conforme DTO do backend) |
| --- | --- | --- | --- |
| `[param]` | `[tipo]` | `[default ou —]` | [Validações e regras] |

**Respostas**

| Status | Corpo | `Failure` resultante | Quando |
| --- | --- | --- | --- |
| `[200]` | `[TipoResposta]` | — | [Condição de sucesso] |
| `[400/422]` | `[TipoErro]` | `ValidationFailure` | [Erro de validação com campo] |
| `[401]` | `[TipoErro]` | `AuthFailure` | [Token ausente/expirado] |
| `[500]` | `[TipoErro]` | `ServerFailure` | [Falha do servidor] |
| — | — | `NetworkFailure` | [`DioExceptionType.connectionTimeout` / `SocketException`] |

**Exemplo — sucesso**

```http
[MÉTODO] [ /rota?param=valor ]
Authorization: Bearer [token]
```

```json
{
  "[campo]": "[valor realista]"
}
```

**Exemplo — [cenário alternativo, ex.: lista vazia]**

```http
[MÉTODO] [ /rota?param=valor ]
```

```json
{
  "[corpo]": []
}
```

> [Nota sobre o comportamento resultante na UI, se aplicável.]

**Exemplo — [cenário de erro]**

```http
[MÉTODO] [ /rota ]
```

```json
{
  "error": {
    "code": "[codigo]",
    "message": "[mensagem]"
  }
}
```

[Repetir o padrão acima para cada rota. Para payloads grandes já documentados em Modelos de dados, referenciar o exemplo JSON existente em vez de duplicar.]

---

## Design e tokens (Figma)

[Preencher quando houver referência de design. Obtido via `mcp__claude_ai_Figma__get_design_context` e `get_variable_defs`.

| Tela | node-id | Golden test correspondente |
| --- | --- | --- |
| `[Nome da tela]` | `[node-id]` | `test/features/[feature]/presentation/pages/[nome]_page_test.dart` |

**Tokens** — reusar token existente sempre que possível:

| Token do Figma | Equivalente no app | Ação |
| --- | --- | --- |
| `[color/primary]` | `AppTheme.[...]` | reusar / criar |

- Divergências entre design e `AppTheme` atual e a decisão tomada
- Assets a baixar (`mcp__claude_ai_Figma__download_assets`) e destino em `assets/`]

## Pontos de integração

[Incluir apenas se a funcionalidade exigir integrações externas:

- Serviços, SDKs ou APIs externos (mapas, pagamento, push, analytics)
- Requisitos de autenticação e onde o token é lido
- Permissões de plataforma necessárias (Android `AndroidManifest.xml` / iOS `Info.plist`)
- Abordagem de tratamento de erros]

## Abordagem de testes

<critical>
Cobertura mínima do projeto: **≥90% por linha** (gate real em `tool/coverage.sh`, executado por `make coverage`). Popule esta seção com o **máximo de casos de teste possível** — ela é a entrada direta do `criar-tasks`.
</critical>

<critical>
Antes de escrever qualquer teste, a skill `.claude/skills/flutter/flutter-test-behavior-audit/SKILL.md` é obrigatória: auditoria dos testes existentes → mapa de cenários por camada → só então implementação. `mocktail` é o único mock permitido.
</critical>

### Testes unitários (domain + data)

[- **Usecases**: um teste por regra de negócio, sucesso e falha
- **Repositories**: mapeiam exceção do datasource → `Failure` correta (mockar o **datasource**, nunca o repositório)
- **Datasources**: usar `DioException` real com `DioExceptionType` correto e `Response` com status real — não `Exception` genérica
- **Models/mappers**: `fromJson` com payload completo, com campos ausentes, com tipo inesperado
- Nomes no formato `should <expected> when <condition>`
- Listar aqui os cenários críticos, incluindo: 400/422 com erro de campo, 401, 500, timeout, `SocketException`, body malformado]

### Testes de BLoC/Cubit

[- `bloc_test` para cada transição de estado
- `expect: orderedEquals([...])` quando a ordem importa
- Cobrir: carregando → sucesso, carregando → vazio, carregando → erro
- Duplo toque no botão de submit: a segunda ação é ignorada enquanto a primeira está em voo]

### Testes de widget

[- Cada estado do Cubit renderiza o que deve: conteúdo, estado vazio, mensagem de erro **no lugar certo**, indicador de carregamento, botão desabilitado durante o envio
- `pumpWidget` com `MaterialApp` (não `MaterialApp.router`), Cubit injetado via `BlocProvider`
- Sem bootstrap de `Module` e sem `Modular.get<T>()` em teste unitário
- Semantics de acessibilidade quando relevante]

### Testes golden (obrigatórios para toda página nova)

[- Um golden em **tema claro** e um em **tema escuro** por tela
- Helper: `pumpScreenWithTheme(tester, screen: ..., brightness: ..., surfaceSize: const Size(390, 844))` de `test/_helpers/pump_screen_with_theme.dart`
- Tag `'golden'` (declarada em `dart_test.yaml`; excluída automaticamente no Linux)
- Escritos **antes** da implementação da página
- Regenerar apenas com aprovação explícita: `flutter test --update-goldens`
- Listar as telas e os estados que ganham golden]

### Testes de integração / E2E

[- Local: `integration_test/[feature]/`
- Execução: `make e2e-[feature] DEVICE=<id>` (ver `Makefile`) ou `flutter test integration_test/[feature] --flavor dev -d <device>`
- Exigem **device/emulador real e rede** — não rodam no gate padrão de CI
- Declarar explicitamente se o teste **escreve** dados no ambiente DEV
- Cobrir apenas os fluxos de usuário principais, não casos de borda]

### Meta de cobertura

[| Camada | Arquivos | Meta |
| --- | --- | --- |
| domain | `[...]` | 100% |
| data | `[...]` | ≥90% |
| presentation | `[...]` | ≥90% |

Verificação: `make coverage` (falha abaixo de 90%).]

## Sequenciamento do desenvolvimento

### Ordem de construção

[Definir sequência de implementação seguindo as camadas — dependências antes dos dependentes:

1. `domain/` — entities, contrato de repositório, usecases (Dart puro, testável isoladamente)
2. `data/` — models/mappers, datasource, implementação do repositório
3. `presentation/` — Cubit/BLoC, depois widgets, depois a página
4. Módulo e rotas nomeadas (binds + `ChildRoute`)
5. Testes de integração/E2E do fluxo completo]

### Dependências técnicas

[Listar bloqueadores de dependências:

- Endpoints do backend já disponíveis? em qual ambiente (dev/homolog/prod)?
- Design finalizado no Figma (claro e escuro)?
- Pacotes novos no `pubspec.yaml` — justificar cada um
- Permissões/configuração nativa (Android/iOS) necessárias]

## Monitoramento e observabilidade

[Definir abordagem usando a infraestrutura existente do app:

- Eventos de analytics a registrar (nome e propriedades)
- Erros reportados a Crashlytics/Sentry e com qual contexto
- Logs via `lib/core/logging` — nível apropriado e **redaction obrigatória de PII** (nunca logar token, CPF, telefone ou e-mail em claro)
- Como diagnosticar a funcionalidade em produção]

## Considerações técnicas

### Principais decisões

[Documentar decisões técnicas importantes:

- Escolha da abordagem e justificativa
- Trade-offs considerados
- Alternativas descartadas e por quê]

### Riscos conhecidos

[Identificar riscos técnicos:

- Desafios potenciais
- Abordagens de mitigação
- Áreas que precisam de pesquisa]

### Segurança

[- Onde tokens/credenciais são armazenados (storage seguro, nunca `SharedPreferences` em claro)
- Nenhum segredo hardcoded no source
- Validação de entrada por deep link
- Dados pessoais em log — confirmar redaction]

### Conformidade com regras e skills

[Listar as regras e skills aplicáveis e como a especificação as respeita:

- `@.claude/CLAUDE.md` — critical rules do projeto
- `@rules.md` — regras de engenharia
- Skills relevantes de `.claude/skills/flutter/` (ver `_INDEX.md`): clean-architecture, bloc-state-management, modular-di, modular-routing, error-handling, logging, test-behavior-audit, tdd-testing, code-quality, feature-docs
- Desvios, se houver, com justificativa]

### Documentação da feature

[Toda feature entrega documentação dupla (critical rule de `@.claude/CLAUDE.md`):

- `docs/[feature]/` — doc técnica oficial em PT-BR (propósito, arquitetura, fluxos, edge cases)
- Registro estratégico no container Obsidian do projeto, com link de volta para `docs/[feature]/`]

### Arquivos relevantes e dependentes

[Listar aqui os arquivos relevantes e dependentes, separados por camada e incluindo os arquivos de teste correspondentes]
