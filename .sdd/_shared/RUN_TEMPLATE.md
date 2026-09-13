# Modelo do `run.yaml` — processo do fluxo SDD

> Um arquivo por feature, na raiz da pasta da feature em `<artefatos_sdd>` — e, quando a camada de
> QA contínuo está ativa, mais um na raiz da árvore de `<qa_continuo>`, com o mesmo esquema. É a
> **fonte de verdade sobre o que já foi feito e onde parou**. Toda skill do fluxo lê este arquivo
> ANTES de começar e o atualiza **passo a passo enquanto trabalha** — não só no fim.
>
> Ele existe para que uma sessão nova saiba onde retomar sem reler PRD, TechSpec e tasks inteiros —
> e, diferente de um resumo por etapa, ele desce ao **passo dentro da etapa**: se a sessão caiu no
> passo 6 de 11 do review, é isso que o arquivo diz.
>
> O *porquê* das decisões não mora aqui: mora na memória de cada skill, em
> `memoria/<skill>.md` (modelo em `MEMORIA_TEMPLATE.md`). O `run.yaml` responde "o que e onde";
> a memória responde "por quê".

<critical>`etapas` é append-only. Rodada nova ACRESCENTA entrada — nunca sobrescreve a anterior. É o histórico de rodadas que sustenta o limite de escalonamento; sobrescrever apaga a prova de que o laço não convergiu.</critical>

## Esquema

```yaml
# Processo do fluxo SDD — <slug>

feature: <slug>
issue: <identificador no rastreador | null>
bundle: "<conteúdo de VERSION na configuração>"
atualizado_em: AAAA-MM-DD

# Onde retomar. Único campo que toda skill lê primeiro.
proxima_acao:
  skill: <nome-da-skill>
  passo: <n>                   # em que passo retomar; 0 = etapa ainda não começou
  porque: <uma linha>

# Decisão pendente do usuário, causa raiz em área read-only, ambiente indisponível.
bloqueios: []

# Copiada de tasks.md por criar-tasks. A feature só fecha com todos os itens `ok: true`.
definition_of_done:
  - { item: "Todas as tarefas concluídas", ok: false }
  - { item: "Todo critério de aceite verificado em qa.md", ok: false }
  - { item: "executar-qa APROVADO na última rodada", ok: false }
  - { item: "executar-review APROVADO ou APROVADO COM RESSALVAS na última rodada", ok: false }
  - { item: "Documentação registrada conforme <documentacao_projeto>", ok: false }
  - { item: "Issue do rastreador atualizada, quando houver", ok: false }

# Uma entrada por EXECUÇÃO de skill. Rodada nova = entrada nova.
etapas:
  - skill: criar-prd
    rodada: 1
    alvo: null                 # nº da tarefa em executar-task; lista de bugs em executar-bugfix
    status: concluida          # pendente | em_andamento | concluida | bloqueada | nao_aplicavel
    inicio: AAAA-MM-DD
    fim: AAAA-MM-DD
    veredito: "8 RFs, 19 critérios de aceite"
    memoria: memoria/criar-prd.md
    passos:
      # status do passo: pendente | em_andamento | concluido | pulado | falhou
      - { n: 0, nome: "Verificar se a feature já existe", status: concluido, nota: "pasta nova" }
      - { n: 1, nome: "Coletar referências de entrada",   status: pulado,    nota: "sem issue nem design" }
      - { n: 2, nome: "Esclarecer",                       status: concluido, nota: "7 perguntas — respostas na memória" }
      - { n: 3, nome: "Planejar",                         status: concluido, nota: "" }
      - { n: 4, nome: "Rascunhar o PRD",                  status: concluido, nota: "8 RFs" }
      - { n: 5, nome: "Criar diretório e salvar",         status: concluido, nota: "prd.md + run.yaml + memoria/" }
      - { n: 6, nome: "Relatar resultados",               status: concluido, nota: "" }
```

## Como cada skill usa este arquivo

1. **Ao começar** — ler `proxima_acao` e a última entrada de `etapas` da própria skill. Se ela
   estiver `em_andamento`, **retomar no primeiro passo que não está `concluido`/`pulado`**, não do
   começo. Abrir entrada nova só quando a anterior estiver fechada.
2. **Ao abrir a etapa** — acrescentar a entrada com `status: em_andamento`, `inicio`, `memoria` e a
   lista completa de `passos` do catálogo abaixo, todos `pendente`.
3. **A cada passo** — marcar `em_andamento` ao entrar; ao sair, gravar `concluido`, `pulado`
   (condição do passo não se aplica) ou `falhou`, com uma `nota` de uma linha.
4. **Ao fechar a etapa** — gravar `status`, `fim`, `veredito`, atualizar `atualizado_em` e
   reapontar `proxima_acao`.
5. **Ao parar no meio** (bloqueio, pergunta ao usuário, limite de rodadas) — deixar a etapa
   `em_andamento` ou `bloqueada`, registrar o motivo em `bloqueios` e apontar `proxima_acao` para o
   passo onde retomar.

## Catálogo de passos

Os nomes abaixo são os que vão no campo `nome` de cada passo. Use-os **literalmente** — é o que
faz um `run.yaml` escrito por uma sessão ser legível pela seguinte.

- **criar-prd** — 0 Verificar se a feature já existe · 1 Coletar referências de entrada ·
  2 Esclarecer · 3 Planejar · 4 Rascunhar o PRD · 5 Criar diretório e salvar · 6 Relatar resultados
- **criar-techspec** — 1 Analisar o PRD · 2 Análise profunda exploratória do projeto ·
  3 Descobrir o contrato real das APIs consumidas · 4 Descobrir os tokens de design ·
  4b Spike · 5 Esclarecimentos técnicos · 6 Mapeamento de conformidade com padrões ·
  7 Gerar a especificação técnica · 8 Salvar a especificação técnica
- **criar-tasks** — 1 Analisar PRD e especificação técnica · 2 Gerar a estrutura de tarefas ·
  3 Gerar arquivos individuais de tarefas
- **executar-task** — 1 Preparação · 2 Análise · 3 Design · 4 Plano/Abordagem ·
  5 Implementação em ciclos TDD · 6 Verificação · 7 Fechamento
- **executar-qa** — 1 Análise · 2 Gate automatizado · 3 Verificação de segurança ·
  4 Validação visual · 5 Testes ponta a ponta · 6 Verificação funcional ·
  7 Verificação de Acessibilidade e i18n · 8 Verificação visual comparativa · 9 Relatório de QA
- **executar-bugfix** — 1 Análise de Contexto · 2 Diagnóstico por bug ·
  3 Teste de regressão primeiro · 4 Validação de bugs visuais · 5 Verificação completa ·
  6 Atualização do bugs.md · 7 Relatório de Correções · 8 Devolver o fluxo para o QA
- **executar-review** — 1 Delimitar o escopo e ler a documentação · 2 Conformidade arquitetural ·
  3 Conformidade com Padrões de Código · 4 Verificação de Aderência à TechSpec ·
  5 Verificação de Completude das Tasks · 6 Execução dos Testes · 7 Qualidade dos Testes ·
  8 Análise de Qualidade de Código · 9 Convenções de Colaboração · 10 Documentação ·
  11 Relatório de Code Review
- **criar-plano-qa** — 1 Resolver a árvore · 2 Estabelecer personas · 3 Mapear jornadas ·
  4 Derivar cenários · 5 Planejar charters · 6 Reconciliar duplicatas · 7 Validar completude ·
  8 Fechar o plano
- **executar-sessao-qa** — 1 Preparação e pré-condições · 2 Montar a matriz de cobertura ·
  3 Percorrer as jornadas · 4 Tour e edge cases · 5 Lentes de risco ·
  6 Registrar e deduplicar achados · 7 Encaminhar bugs · 8 Fechar a sessão

Os dois últimos são as skills do grupo `qa/` e escrevem no `run.yaml` da árvore de
`<qa_continuo>`, não no de uma feature. O campo `feature:` do cabeçalho vira `escopo:` nesse
arquivo (ex.: `escopo: produto-inteiro`), e `definition_of_done` não se aplica — a árvore de QA
não fecha, ela é contínua.

Passo condicional que não se aplica ao projeto entra como `pulado`, com a `nota` dizendo qual
campo do config decidiu — nunca é omitido da lista. Passo omitido é indistinguível de passo
esquecido.

## Rodadas de verificação

<critical>
QA e bugfix formam um laço: `executar-bugfix` **não encerra a feature**, ele devolve o fluxo para
`executar-qa`. O mesmo vale para review → correção → review. Uma rodada de QA que aprovou antes
das correções não vale para as correções.
</critical>

Cada volta do laço acrescenta uma entrada em `etapas` com `rodada` incrementada. Para saber em que
rodada a feature está, conte as entradas da skill; para saber se o laço convergiu, olhe o
`veredito` da última.

- **Limite:** a partir da **terceira** entrada de `executar-qa` com veredito REPROVADO na mesma
  feature, pare e escale ao usuário em vez de abrir a quarta. Três rodadas sem convergir indicam
  problema no PRD, na TechSpec ou na causa raiz — não mais um bug para corrigir. Registre a
  escalação em `bloqueios`.
