# Relatório de QA - [Nome da Funcionalidade]

## Resumo
- Data: [data]
- Branch: [branch]
- Status: APROVADO / APROVADO COM RESSALVAS / REPROVADO
- Total de Requisitos: [X]
- Requisitos Atendidos: [Y]
- Bugs Encontrados: [Z]
- Issue vinculada: [identificador no rastreador configurado ou "sem issue"]

## Gate automatizado
[Uma linha por comando de `<comandos_projeto>`.]

| Etapa | Comando | Resultado |
|-------|---------|-----------|
| Format | [comando do config] | OK / NOK — [arquivos desformatados] |
| Lint | [comando do config] | [N] issues (esperado: 0) |
| Testes | [comando do config] | TODOS PASSANDO / [N] falhando |
| Cobertura | [comando do config] | [resultado] — threshold [X]% OK / FAIL |

> Gate reprovado bloqueia o QA.

## Segurança
| Verificação | Resultado | Observações |
|-------------|-----------|-------------|
| Scanner de dependências vulneráveis | [comando] — [N] achados (alta/crítica: [N]) | [obs] |
| Baseline de segurança (`_shared/SECURITY_BASELINE.md`) | OK / [N] desvios | [obs] |
| Extensões de `<seguranca_extensoes>` | OK / [N] desvios | [obs] |

> Achado de severidade alta ou crítica reprova o QA e vira bug.

## Validação visual
[Preencher apenas se `<ui_projeto>` declarar ferramenta de validação visual.]

| Tela | Variações verificadas | Resultado | Observações |
|------|-----------------------|-----------|-------------|
| [nome da tela] | [ex.: temas suportados] | PASSOU/FALHOU/AUSENTE | [obs] |

> Comando: [comando do config]. Ausência de validação visual para tela nova = bug de severidade Alta.

## Testes ponta a ponta
- Ambiente/dispositivo utilizado: [descrição ou "nenhum disponível"]

| Alvo | Cenário | Resultado | Escreveu em ambiente compartilhado? |
|------|---------|-----------|--------------------------------------|
| [comando do config] | [cenário coberto] | PASSOU/FALHOU/NÃO EXECUTADO | sim/não |

> Se não executado, registre o motivo como pendência explícita — não reprove por isso.
> Alvos que escrevem em ambiente compartilhado exigem confirmação prévia do usuário.

## Requisitos Verificados
| ID | Requisito | Status | Evidência |
|----|-----------|--------|-----------|
| RF-01 | [descrição] | PASSOU/FALHOU | [evidência na pasta de evidências] |

## Fluxos testados
| Fluxo | Estados verificados | Resultado | Observações |
|-------|---------------------|-----------|-------------|
| [fluxo] | carregando / vazio / erro / sem conexão | PASSOU/FALHOU | [obs] |

## Acessibilidade
[Preencher apenas se `<ui_projeto>` indicar acessibilidade aplicável.]

| Verificação | Resultado | Observações |
|-------------|-----------|-------------|
| Rótulo descritivo em elementos interativos | OK / NOK | [obs] |
| Leitor de tela anuncia corretamente | OK / NOK | [obs] |
| Contraste adequado em todos os temas suportados | OK / NOK | [obs] |
| Alvo de interação com tamanho mínimo | OK / NOK | [obs] |
| Layout suporta aumento de escala de fonte | OK / NOK | [obs] |
| Navegação por teclado e ordem de foco | OK / NOK | [obs] |
| Mensagens de erro associadas ao campo correto | OK / NOK | [obs] |
| i18n: strings vêm do mecanismo declarado (sem texto fixo), layout suporta texto mais longo | OK / NOK / N/A | [obs] |

## Verificação visual comparativa
[Preencher apenas se `<integracoes_projeto>` indicar ferramenta de design.]

| Tela | Referência de design | Aderência | Observações |
|------|----------------------|-----------|-------------|
| [tela] | [identificador ou —] | OK / divergente | [obs] |

- Tamanhos de tela verificados: [lista]
- Temas verificados: [lista]

## Bugs Encontrados
| ID | Descrição | Severidade | Evidência |
|----|-----------|------------|-----------|
| BUG-01 | [descrição] | Alta/Média/Baixa | [link] |

## Pendências
[Itens não verificados e o motivo.]

## Conclusão
[Parecer final do QA]
