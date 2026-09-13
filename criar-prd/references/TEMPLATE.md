# Documento de Requisitos do Produto (PRD)

## Visão Geral

[Forneça uma visão geral do seu produto/funcionalidade. Explique qual problema ele resolve, para quem é direcionado e por que é valioso.

- Faça no máximo 2 paragráfos para esta parte]

## Objetivos

[Listar objetivos específicos e mensuráveis para esta funcionalidade:

- O que significa ter sucesso
- Principais métricas a serem acompanhadas
- Metas de negócios a serem alcançadas]

## Histórias de Usuário

[Detalhe narrativas de usuários descrevendo o uso e os benefícios da funcionalidade:

- Como [tipo de usuário], eu quero [realizar uma ação] para que [benefício]
- Inclua personas de usuário primárias e secundárias
- Cubra fluxos principais e casos de borda

Formato use bullet list com US$NUM (US1)]

## Principais funcionalidades

[Liste e descreva as principais funcionalidades do produto. Para cada uma, inclua:

- O que faz
- Por que é importante
- Como funciona em alto nível]

## Requisitos funcionais e critérios de aceite

<critical>
**OBRIGATÓRIO.** Esta seção é a espinha do fluxo SDD: cada RF daqui vira caso de teste na
TechSpec, subtarefa de teste em `criar-tasks` e linha do checklist em `executar-qa`. RF sem
critério de aceite testável faz as três etapas seguintes inventarem, cada uma, a própria
interpretação do mesmo requisito.
</critical>

Um bloco por requisito. O critério descreve **comportamento observável** — nunca implementação.

### RF-01 — [título curto do requisito]

- **Prioridade:** MVP | incremento
- **Histórias atendidas:** [US1, US2]

| # | Dado | Quando | Então |
| --- | --- | --- | --- |
| RF-01.1 | [pré-condição ou estado inicial] | [ação do usuário ou evento] | [resultado observável] |
| RF-01.2 | [estado que leva à falha] | [mesma ação] | [erro específico e o que o consumidor vê] |

[Todo RF precisa de pelo menos um critério de caminho feliz **e** um de caminho de falha. Estados
vazio e de carregamento entram aqui sempre que forem observáveis. Se um critério não puder ser
verificado sem olhar o código por dentro, ele está escrito no nível errado — reescreva.]

[Repita o bloco para cada RF.]

## Plataforma alvo

[Preencha conforme `<ui_projeto>` do `sdd.config.md`. Cubra apenas o que se aplica ao tipo
deste projeto:

- Plataformas/ambientes alvo e diferenças de comportamento entre eles
- Permissões, capacidades ou limites do ambiente, e o que acontece quando faltam
- Comportamento sem conectividade, se aplicável
- Formas de entrada além do fluxo principal (link direto, notificação, chamada externa)

Se o projeto não tem interface de usuário final, descreva aqui os consumidores do que está
sendo construído e os contratos que eles dependem.]

## Referências de design

[Preencha apenas se `<integracoes_projeto>` indicar uma ferramenta de design ativa. Uma linha
por tela:

| Tela | Referência | Observações |
| --- | --- | --- |
| [Nome da tela] | `[identificador ou link]` | [estados cobertos: vazio, carregando, erro] |

- Se o projeto tem temas (ex.: claro e escuro), sinalize se o design cobriu todos
- Se não houver design, declare "sem referência de design" e trate como risco]

## Experiência do usuário

[Preencha esta seção apenas se `<ui_projeto>` indicar que o projeto tem interface de usuário final. Se não tiver, escreva "não aplicável — ver 'Plataforma alvo' para os consumidores e contratos" e siga para a próxima seção.

Descreva a jornada e a experiência do usuário:

- Personas e necessidades
- Fluxos principais e interações
- Considerações e requisitos de UI/UX (incluindo tema claro e escuro)
- Estados de carregamento, vazio e erro de cada tela
- Requisitos de acessibilidade, se `<ui_projeto>` indicar acessibilidade aplicável: rótulos para
  leitor de tela, contraste, alvo de interação, suporte a aumento de escala de fonte, navegação
  por teclado quando aplicável]

## Requisitos não funcionais

[Números, não adjetivos. "Rápido" não é verificável; "p95 abaixo de 300 ms" é. O que for
verdadeiramente desconhecido fica como "a definir" — e vira risco, não silêncio.

| Requisito | Alvo | Como será verificado |
| --- | --- | --- |
| [Desempenho percebido — ex.: tempo até o primeiro conteúdo] | [alvo] | [método] |
| [Volume esperado — ex.: itens por usuário, requisições por minuto] | [alvo] | [método] |
| [Disponibilidade, quando aplicável] | [alvo] | [método] |
| [Limite de uso de serviço externo consumido] | [cota] | [método] |

- Comportamento esperado quando o alvo é estourado: degradar, enfileirar, recusar?
- Retenção e privacidade do dado gerado por esta feature]

## Custo e compromissos externos

[Preencher se a feature depender de serviço pago, cota de terceiro ou licença restritiva —
é decisão de negócio e precisa aparecer aqui, não só na TechSpec. Se não houver, "nenhum".]

## Restrições técnicas de alto nível

[Capture apenas restrições e considerações de alto nível:

- Serviços, APIs ou sistemas existentes que a funcionalidade consome, e quais deles estão fora
  do escopo de alteração conforme `<limites_projeto>`
- Integrações externas obrigatórias ou sistemas existentes com os quais interagir
- Exigências de conformidade, regulatórias ou de segurança (ex.: dados pessoais, LGPD, armazenamento de token)
- Metas de desempenho percebido (tempo até o primeiro conteúdo, limites de latência aceitáveis)
- Considerações sobre sensibilidade/privacidade de dados
- Requisitos de tecnologia ou protocolo não negociáveis

Os detalhes de implementação serão tratados na Especificação Técnica.]

## Fora do escopo

[Declare claramente o que esta feature NÃO incluirá para gerir o escopo:

- Funcionalidades explicitamente excluídas
- Considerações futuras fora do escopo
- Limites e restrições

(Nota: riscos técnicos de implementação serão detalhados na Especificação Técnica.)]

## Rastreabilidade

[- Issue no rastreador configurado: `[identificador]` ou "sem issue vinculada"
- Referência de design: `[link]` ou "sem design"
- Documentação da feature a ser produzida: conforme `<documentacao_projeto>`]
