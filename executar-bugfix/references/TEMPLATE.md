# Relatório de Bugfix - [Nome da Funcionalidade]

## Resumo
- Data: [data]
- Branch: [branch]
- Total de Bugs: [X]
- Bugs Corrigidos: [Y]
- Bugs Não Corrigidos: [Z] (ver "Bloqueios")
- Testes de Regressão Criados: [N]
- Issue vinculada: [identificador no rastreador configurado ou "sem issue"]

## Detalhes por Bug
| ID | Severidade | Status | Camada da causa raiz | Correção | Testes de regressão criados |
|----|------------|--------|----------------------|----------|------------------------------|
| BUG-01 | Alta | Corrigido | [camada de `<stack_projeto>`] | [descrição] | [arquivos e nomes dos testes] |

## Verificação
[Uma linha por comando de `<comandos_projeto>` que se aplica.]

| Etapa | Comando | Resultado |
|-------|---------|-----------|
| Format | [comando do config] | OK / NOK |
| Lint | [comando do config] | [N] issues (esperado: 0) |
| Testes | [comando do config] | TODOS PASSANDO / [N] falhando |
| Cobertura | [comando do config] | [resultado] — threshold [X]% OK / FAIL |
| Validação visual | [comando do config, se aplicável] | TODOS PASSANDO / [N] falhando / N/A |
| Ponta a ponta | [comando do config, se aplicável] | PASSOU / FALHOU / NÃO EXECUTADO |

> Artefatos de validação visual regenerados nesta rodada? [não / sim — aprovado pelo usuário em [contexto]]

## Evidências visuais
[Preencher apenas se `<ui_projeto>` indicar validação visual.]

| BUG | Antes | Depois | Variações verificadas |
|-----|-------|--------|-----------------------|
| BUG-0X | [evidência] | [evidência] | [ex.: temas, tamanhos de tela] |

## Bloqueios
[Bugs cuja causa raiz está em área read-only de `<limites_projeto>`, ou que dependem de decisão
do usuário. Descreva o achado e o que é necessário para destravar.]

## Conclusão
[Parecer final]
