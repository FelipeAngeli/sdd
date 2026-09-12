# Relatório de Code Review - [Nome da Funcionalidade]

## Resumo
- Data: [data]
- Branch: [branch]
- Status: APROVADO / APROVADO COM RESSALVAS / REPROVADO
- Issue vinculada: [identificador no rastreador configurado ou "sem issue"]

## Conformidade Arquitetural (bloqueante)
[Uma linha por regra de direção de dependência de `<stack_projeto>` e por regra inviolável de
`<limites_projeto>`.]

| Regra | Status | Observações |
|-------|--------|-------------|
| Direção de dependência entre camadas respeitada | OK/NOK | [obs] |
| Camada de regra de negócio isolada conforme o padrão arquitetural de `<stack_projeto>` | OK/NOK | [obs] |
| Erros tratados conforme "Tratamento de erro" de `<stack_projeto>` | OK/NOK | [obs] |
| Pontos de entrada/rotas registrados conforme "Roteamento / pontos de entrada" de `<stack_projeto>` | OK/NOK | [obs] |
| Injeção de dependência conforme "Injeção de dependência" de `<stack_projeto>` | OK/NOK | [obs] |
| Logging pelo mecanismo declarado em "Logging" de `<stack_projeto>` | OK/NOK | [obs] |
| Área read-only de `<limites_projeto>` intocada | OK/NOK | [obs] |
| SOLID e coesão | OK/NOK | [obs] |

## Conformidade com as Regras do Projeto
| Regra | Status | Observações |
|-------|--------|-------------|
| [regra de `<limites_projeto>` ou `<qualidade_extensoes>`] | OK/NOK | [obs] |

## Aderência à TechSpec
| Decisão Técnica | Implementado | Observações |
|-----------------|--------------|-------------|
| [decisão] | SIM/NÃO | [obs] |

## Tasks Verificadas
| Task | Status | Observações |
|------|--------|-------------|
| [task] | COMPLETA/INCOMPLETA | [obs] |

## Gate automatizado
| Etapa | Comando | Resultado |
|-------|---------|-----------|
| Format | [comando do config] | OK / NOK |
| Lint | [comando do config] | [N] issues (esperado: 0) |
| Testes | [comando do config] | TODOS PASSANDO / [N] falhando |
| Cobertura | [comando do config] | [resultado] — threshold [X]% OK / FAIL |

## Segurança
| Verificação | Status | Observações |
|-------------|--------|-------------|
| Baseline (`_shared/SECURITY_BASELINE.md`) | OK / [N] desvios | [obs] |
| Scanner de dependências vulneráveis | [N] achados (alta/crítica: [N]) | [obs] |
| Extensões de `<seguranca_extensoes>` | OK / [N] desvios | [obs] |

## Testes
- Total de Testes: [X] · Passando: [Y] · Falhando: [Z]
- **Cobertura: [%]** (threshold de `<comandos_projeto>`: [X]%)
- Validação visual para telas novas: OK / AUSENTE / N/A — [quais]
- Testes ponta a ponta atualizados: SIM / NÃO / N/A

### Qualidade dos testes
[Conforme `_shared/QUALITY_BASELINE.md`, seção "Qualidade dos testes".]

| Verificação | Status | Observações |
|-------------|--------|-------------|
| Testam comportamento, não implementação | OK/NOK | [obs] |
| Sem asserção fraca isolada | OK/NOK | [obs] |
| Caminhos de falha pareados com caminho feliz | OK/NOK | [obs] |
| Erro concreto e específico nos testes de falha | OK/NOK | [obs] |
| Mock na fronteira certa, biblioteca declarada no config | OK/NOK | [obs] |

## Convenções de colaboração
| Item | Status | Observações |
|------|--------|-------------|
| Convenção de commit de `<colaboracao_projeto>` | OK/NOK | [obs] |
| Convenção de branch/PR | OK/NOK | [obs] |
| Política de comentários no código | OK/NOK | [obs] |

## Documentação
[Conforme `<documentacao_projeto>`.]

| Item | Status | Caminho |
|------|--------|---------|
| Documentação da feature no repositório | OK/AUSENTE/N/A | [caminho] |
| Registro externo obrigatório | OK/AUSENTE/N/A | [caminho] |

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
