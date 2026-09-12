---
name: executar-bugfix
description: Leia o relatório de bugs de uma feature do app Flutter mobile, analise e corrija cada defeito na causa raiz, crie testes de regressão começando pelo teste que reproduz a falha, valide bugs visuais por golden test e app em device, e gere o relatório final de correções. Use sempre que o usuário pedir para corrigir bugs, executar bugfix, tratar defeitos do bugs.md, resolver problemas reportados em QA, ou criar testes de regressão para correções.
---

<prd>`--prd`</prd>
<template>`./references/TEMPLATE.md`</template>
<linear>`--linear` (identificador da issue, ex: LIG-123, opcional)</linear>
<device>`--device` (id do emulador/device, ex: emulator-5554, opcional)</device>

<contexto_projeto>
App **Flutter mobile** (Android/iOS) — a pasta `web/` existe mas **não é plataforma alvo**.
Clean Architecture + `flutter_modular` + BLoC/Cubit + `dartz` (`Either<Failure, T>`) + `dio`.
Regras invioláveis do projeto: `@.claude/CLAUDE.md`. Roteador de skills: `@AGENTS.md` + `.claude/skills/flutter/_INDEX.md`.
`backend/` (NestJS) é **read-only**: se a causa raiz estiver no servidor, **não corrija** — documente o achado e sinalize para o time de backend.
</contexto_projeto>

<comandos>
`make ci` (= `format-check` + `analyze` + `coverage` com gate de 90%) · `make test` · `make analyze` · `make coverage`
Goldens: `flutter test --tags golden` · regenerar só com aprovação: `flutter test --update-goldens`
Integration tests: `make e2e-<feature> DEVICE=<id>` · app em device: `flutter run --flavor dev -d <id>`
</comandos>

## Persona

Você é um desenvolvedor Flutter especializado na correção de defeitos. Sua tarefa é ler o relatório de bugs e analisar cada um deles, implementar as correções na causa raiz e criar testes de regressão para garantir que os problemas não voltem a acontecer.

<critical>Carregue as skills ANTES de corrigir: `load_skills_for_files` do MCP `agent-skills-standard`; se indisponível, siga `@AGENTS.md` e `.claude/skills/flutter/_INDEX.md`</critical>
<critical>TDD também vale para bugfix: escreva PRIMEIRO o teste de regressão que **falha reproduzindo o bug**, confirme que ele falha pelo motivo certo, e só então corrija</critical>
<critical>Corrija a CAUSA RAIZ, nunca o sintoma. Se a correção é um `try/catch` engolindo o erro ou um `if` defensivo na UI, provavelmente está na camada errada</critical>
<critical>NÃO use automação de navegador — este é um app mobile. Bug visual valida por golden test + app em emulador/device</critical>
<critical>NUNCA rode `flutter test --update-goldens` para "resolver" um golden vermelho sem aprovação explícita do usuário — isso apaga a evidência da regressão visual</critical>
<critical>O bugfix NÃO está completo enquanto `make ci` não passar (formatação, `flutter analyze` zero issues, cobertura ≥90%)</critical>

## Localização dos arquivos

- PRD: `./tasks/prd-[nome-da-funcionalidade]/prd.md`
- TechSpec: `./tasks/prd-[nome-da-funcionalidade]/techspec.md`
- Tasks: `./tasks/prd-[nome-da-funcionalidade]/tasks.md`
- Bugs: `./tasks/prd-[nome-da-funcionalidade]/bugs.md`
- Relatório de Correções: `./tasks/prd-[nome-da-funcionalidade]/bugfixes.md`
- Relatório de QA: `./tasks/prd-[nome-da-funcionalidade]/qa.md`
- Evidências (telas): `./tasks/prd-[nome-da-funcionalidade]/evidences`

Utilize o `nome-da-funcionalidade` como o <prd>

## Etapas para Executar

### 1. Análise de Contexto (Obrigatório)

- Ler o arquivo `bugs.md` e extrair TODOS os bugs documentados
- Ler o PRD para entender os requisitos afetados por cada bug
- Ler a TechSpec para entender as decisões técnicas relevantes
- Revisar `@.claude/CLAUDE.md`, `@rules.md` e as skills aplicáveis para garantir conformidade nas correções
- Se houver <linear>: `mcp__linear-server__get_issue` e `list_comments` para contexto adicional

<critical>NÃO PULE ESTA ETAPA — Entender o contexto completo é fundamental para correções de qualidade</critical>

### 2. Diagnóstico por bug (Obrigatório)

Para cada bug, antes de tocar em código:

1. **Localizar o código afetado** — ler e entender os arquivos envolvidos
2. **Identificar a camada da causa raiz** — `datasource` (contrato/parse), `repository` (mapeamento exceção → `Failure`), `bloc/cubit` (transição de estado), `widget` (renderização), ou navegação/DI
3. **Formular a hipótese** — qual comportamento está errado e por quê
4. Se a causa estiver no `backend/`: **pare**, documente e sinalize — o backend é read-only

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

Regras de escrita (ver `flutter-test-behavior-audit` e `flutter-tdd-testing`):

- `mocktail` apenas; `bloc_test` para Bloc/Cubit
- Nome no formato `should <expected> when <condition>`
- Falha de rede com `DioException` real e `DioExceptionType` correto — nunca `Exception` genérica
- Para `Either<Failure, T>`, asserte o lado **e** o subtipo de `Failure` **e** a mensagem quando relevante
- Mock na camada certa: teste de repositório mocka o datasource

### 4. Validação de bugs visuais (Obrigatório para bugs de UI)

Este app não roda em navegador. Para bugs que afetam a interface:

1. **Golden test** — se a tela já tem golden, rode `flutter test --tags golden` e verifique o diff. Se não tem, **crie** (light e dark) como parte da correção
2. Confirme visualmente no device: `flutter run --flavor dev -d <device>`
3. Capture evidência (screenshot) em `evidences/`
4. Verifique em tema **claro e escuro**, e nos estados vazio/carregando/erro
5. Só depois de o usuário aprovar a nova aparência, regenere: `flutter test --update-goldens`
6. Se houver design no Figma, compare com `mcp__claude_ai_Figma__get_screenshot`

### 5. Verificação completa (Obrigatório)

```bash
make format-check   # dart format sem diff
make analyze        # flutter analyze — zero issues
make test           # toda a suíte
make coverage       # gate de 90% por linha
```

- Nenhum teste existente pode ter quebrado
- Se o bug tinha cobertura de integração, rode também `make e2e-<feature> DEVICE=<id>` — pedindo **confirmação antes** dos alvos que escrevem no DEV (`e2e-help-safety`, `e2e-lost-items`)

### 6. Atualização do bugs.md (Obrigatório)

Após corrigir cada bug, atualize o arquivo `bugs.md` adicionando ao final de cada bug:

```
- **Status:** Corrigido
- **Causa raiz:** [camada e explicação]
- **Correção aplicada:** [descrição breve da correção]
- **Testes de regressão:** [lista dos testes criados]
```

### 7. Relatório de Correções (Obrigatório)

Gerar um resumo final seguindo o formato definido em <template>. Atualizar também o `qa.md` com os bugs resolvidos. Se houver issue do Linear vinculada, poste um comentário com a lista de bugs corrigidos e o resultado de `make ci` via `mcp__linear-server__save_comment`.

## Checklist de Qualidade

- [ ] Arquivo bugs.md lido e todos os bugs identificados
- [ ] PRD e TechSpec revisados para contexto
- [ ] Causa raiz identificada por camada para cada bug
- [ ] Teste de regressão escrito **antes** da correção e confirmado falhando
- [ ] Correções implementadas na causa raiz (não no sintoma)
- [ ] Nenhuma alteração em `backend/`
- [ ] Bugs visuais validados por golden (light e dark) e no device
- [ ] Goldens regenerados apenas com aprovação explícita
- [ ] `make format-check analyze test` verde
- [ ] `make coverage` ≥90%
- [ ] Arquivo bugs.md atualizado com status das correções
- [ ] Relatório final gerado em bugfixes.md
- [ ] Relatório qa.md atualizado com os bugs corrigidos
- [ ] Comentário postado na issue do Linear (se houver)

<critical>TDD também vale para bugfix: teste de regressão falhando PRIMEIRO, correção depois</critical>
<critical>Corrija a CAUSA RAIZ, nunca o sintoma</critical>
<critical>NUNCA regenere goldens sem aprovação explícita do usuário</critical>
<critical>O bugfix NÃO está completo enquanto `make ci` não passar</critical>
