# Relatório de QA - [Nome da Funcionalidade]

## Resumo
- Data: [data]
- Branch: [branch]
- Status: APROVADO / APROVADO COM RESSALVAS / REPROVADO
- Total de Requisitos: [X]
- Requisitos Atendidos: [Y]
- Bugs Encontrados: [Z]
- Issue do Linear: [LIG-XXX ou "sem issue"]

## Gate automatizado (`make ci`)
| Etapa | Comando | Resultado |
|-------|---------|-----------|
| Formatação | `make format-check` | OK / NOK — [arquivos desformatados] |
| Análise estática | `make analyze` | [N] issues (esperado: 0) |
| Cobertura | `make coverage` | `Lines: [hit]/[total] ([X]%)` — gate 90% OK / FAIL |

> Gate reprovado bloqueia o QA.

## Golden tests
| Tela | Light | Dark | Observações |
|------|-------|------|-------------|
| [nome da tela] | PASSOU/FALHOU/AUSENTE | PASSOU/FALHOU/AUSENTE | [obs] |

> Comando: `flutter test --tags golden`. Golden ausente para página nova = bug Alta.
> Goldens não rodam no CI (excluídos no Linux) — este gate é local no macOS.

## Integration tests (`integration_test/`)
- Device utilizado: [id do emulador/device ou "nenhum disponível"]
- Flavor: [dev / homolog / prod]

| Alvo | Cenário | Flavor | Resultado | Escreveu no DEV? |
|------|---------|--------|-----------|------------------|
| `make e2e-[feature]` | [cenário coberto] | [dev] | PASSOU/FALHOU/NÃO EXECUTADO | sim/não |

> Se não executado, registrar o motivo (ex.: sem device disponível) como pendência explícita.
> Alvos que escrevem no DEV (`e2e-help-safety`, `e2e-lost-items`) exigem confirmação prévia do usuário.

## Requisitos Verificados
| ID | Requisito | Status | Evidência |
|----|-----------|--------|-----------|
| RF-01 | [descrição] | PASSOU/FALHOU | [screenshot em evidences/] |

## Fluxos testados no app
| Fluxo | Estados verificados | Resultado | Observações |
|-------|---------------------|-----------|-------------|
| [fluxo] | carregando / vazio / erro / offline | PASSOU/FALHOU | [obs] |

## Acessibilidade
| Verificação | Resultado | Observações |
|-------------|-----------|-------------|
| Rótulos em elementos interativos (`Semantics` / `semanticLabel` / `labelText`) | OK / NOK | [obs] |
| Leitor de tela (TalkBack / VoiceOver) | OK / NOK | [obs] |
| Contraste em tema claro e escuro | OK / NOK | [obs] |
| Alvos de toque | OK / NOK | [obs] |
| Escala de fonte até 2.0 sem quebra de layout | OK / NOK | [obs] |
| Mensagens de erro associadas ao campo correto | OK / NOK | [obs] |

## Verificação visual
| Tela | node-id do Figma | Aderência ao design | Observações |
|------|------------------|---------------------|-------------|
| [tela] | [node-id ou —] | OK / divergente | [obs] |

- Tamanhos de tela verificados: [pequena / grande]
- Orientações verificadas: [retrato / paisagem]
- Temas verificados: claro e escuro

## Bugs Encontrados
| ID | Descrição | Severidade | Evidência |
|----|-----------|------------|-----------|
| BUG-01 | [descrição] | Alta/Média/Baixa | [link] |

## Pendências
[Itens não verificados e o motivo — ex.: integration test não executado por falta de device]

## Conclusão
[Parecer final do QA]
