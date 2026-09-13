# Resumo das tarefas de implementação de [Funcionalidade]

> Ordem de sequenciamento: conforme a ordem de dependência entre camadas de `<stack_projeto>`.
> Gate de conclusão da feature: comandos de `<comandos_projeto>` verdes, cobertura no threshold
> declarado.

## Tarefas

| # | Tarefa | Camada | RFs atendidos | Depende de | Paralelizável com |
| --- | --- | --- | --- | --- | --- |
| 1.0 | [título] | `[camada]` | `RF-01` | — | — |
| 2.0 | [título] | `[camada]` | `RF-01, RF-02` | 1.0 | 3.0 |
| N.0 | Documentação da feature (conforme `<documentacao_projeto>`) | — | — | todas | — |

- [ ] 1.0 Título da tarefa `[camada]`
- [ ] 2.0 Título da tarefa `[camada]`
- [ ] N.0 Documentação da feature (conforme `<documentacao_projeto>`)

## Definition of done da feature

A feature só está concluída quando **todos** os itens abaixo estiverem verdadeiros:

- [ ] Todas as tarefas acima marcadas como completas
- [ ] Todo critério de aceite do PRD verificado em `qa.md`
- [ ] `executar-qa` com veredito APROVADO na última rodada
- [ ] `executar-review` com veredito APROVADO ou APROVADO COM RESSALVAS na última rodada
- [ ] Documentação registrada conforme `<documentacao_projeto>`
- [ ] Issue do rastreador atualizada, quando houver
