# Relatório de Bugfix - [Nome da Funcionalidade]

## Resumo
- Data: [data]
- Branch: [branch]
- Total de Bugs: [X]
- Bugs Corrigidos: [Y]
- Bugs Não Corrigidos: [Z] (ver "Bloqueios")
- Testes de Regressão Criados: [N]
- Issue do Linear: [LIG-XXX ou "sem issue"]

## Detalhes por Bug
| ID | Severidade | Status | Camada da causa raiz | Correção | Testes de regressão criados |
|----|------------|--------|----------------------|----------|------------------------------|
| BUG-01 | Alta | Corrigido | datasource / repository / cubit / widget / navegação | [descrição] | [arquivos e nomes dos testes] |

## Verificação
| Etapa | Comando | Resultado |
|-------|---------|-----------|
| Formatação | `make format-check` | OK / NOK |
| Análise estática | `make analyze` | [N] issues (esperado: 0) |
| Suíte de testes | `make test` | TODOS PASSANDO / [N] falhando |
| Cobertura | `make coverage` | `Lines: [hit]/[total] ([X]%)` — gate 90% OK / FAIL |
| Golden tests | `flutter test --tags golden` | TODOS PASSANDO / [N] falhando |
| Integration tests | `make e2e-[feature] DEVICE=[id]` | PASSOU / FALHOU / NÃO EXECUTADO |

> Goldens regenerados nesta rodada? [não / sim — aprovado pelo usuário em [contexto]]

## Evidências visuais
| BUG | Antes | Depois | Tema verificado |
|-----|-------|--------|-----------------|
| BUG-0X | [screenshot] | [screenshot] | claro e escuro |

## Bloqueios
[Bugs cuja causa raiz está no `backend/` (read-only) ou que dependem de decisão do usuário. Descrever o achado e o que é necessário para destravar.]

## Conclusão
[Parecer final]
