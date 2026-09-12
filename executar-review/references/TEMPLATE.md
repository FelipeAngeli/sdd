# Relatório de Code Review - [Nome da Funcionalidade]

## Resumo
- Data: [data]
- Branch: [branch]
- Status: APROVADO / APROVADO COM RESSALVAS / REPROVADO
- Issue do Linear: [LIG-XXX ou "sem issue"]

## Conformidade Arquitetural (bloqueante)
| Regra | Status | Observações |
|-------|--------|-------------|
| Presentation ⇏ Data (camadas respeitadas) | OK/NOK | [obs] |
| `domain/` é Dart puro (sem Flutter, dio, Modular) | OK/NOK | [obs] |
| Contratos retornam `Either<Failure, T>` | OK/NOK | [obs] |
| Rotas nomeadas via `flutter_modular` (sem `MaterialPageRoute` inline) | OK/NOK | [obs] |
| DI por construtor (sem `Modular.get<T>()` na produção) | OK/NOK | [obs] |
| `backend/` intocado | OK/NOK | [obs] |
| SOLID e OOP | OK/NOK | [obs] |

## Conformidade com Rules e Skills
| Rule / Skill | Status | Observações |
|--------------|--------|-------------|
| [rule ou skill] | OK/NOK | [obs] |

## Aderência à TechSpec
| Decisão Técnica | Implementado | Observações |
|-----------------|--------------|-------------|
| [decisão] | SIM/NÃO | [obs] |

## Tasks Verificadas
| Task | Status | Observações |
|------|--------|-------------|
| [task] | COMPLETA/INCOMPLETA | [obs] |

## Gate automatizado (`make ci`)
| Etapa | Comando | Resultado |
|-------|---------|-----------|
| Formatação | `make format-check` | OK / NOK |
| Análise estática | `make analyze` | [N] issues (esperado: 0) |
| Cobertura | `make coverage` | `Lines: [hit]/[total] ([X]%)` — gate 90% OK / FAIL |

## Testes
- Total de Testes: [X]
- Passando: [Y]
- Falhando: [Z]
- **Coverage: [%]** (gate: ≥90% por linha)
- Golden tests (light/dark) para páginas novas: OK / AUSENTE — [quais]
- Integration tests atualizados: SIM / NÃO / N/A

### Qualidade dos testes
| Verificação | Status | Observações |
|-------------|--------|-------------|
| Testam comportamento, não implementação | OK/NOK | [obs] |
| Sem asserção fraca isolada (`isNotNull`, `isA<T>()`) | OK/NOK | [obs] |
| Caminhos de falha pareados com caminho feliz | OK/NOK | [obs] |
| `DioException` real com tipo correto | OK/NOK | [obs] |
| Mock na camada certa; `mocktail` apenas | OK/NOK | [obs] |

## Documentação
| Item | Status | Caminho |
|------|--------|---------|
| `docs/<feature>/` no repo (PT-BR) | OK/AUSENTE | [caminho] |
| Registro estratégico no container Obsidian | OK/AUSENTE | [caminho] |

## Problemas Encontrados
| Severidade | Arquivo | Linha | Descrição | Sugestão |
|------------|---------|-------|-----------|----------|
| Alta/Média/Baixa | [file] | [line] | [desc] | [fix] |

## Pontos Positivos
- [pontos positivos identificados]

## Recomendações
- [recomendações de melhoria]

## Conclusão
[Parecer final do review]
