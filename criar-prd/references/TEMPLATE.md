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
- Como funciona em alto nível
- Requisitos funcionais (numerados para clareza)]

## Plataformas alvo

[App **Flutter mobile**. Preencha:

- Plataformas: Android e/ou iOS (a pasta `web/` do repo **não é alvo**)
- A funcionalidade se comporta igual nas duas plataformas? Se não, descreva a diferença
- Flavors afetados: `dev` / `homolog` / `prod`
- Permissões de sistema necessárias (localização, notificação, câmera, contatos, armazenamento) e o que acontece se o usuário negar
- Comportamento offline / sem conectividade
- Entrada por deep link ou push notification, se aplicável]

## Referências de design (Figma)

[Liste os frames/telas que compõem a funcionalidade. Uma linha por tela:

| Tela | node-id | Observações |
| --- | --- | --- |
| [Nome da tela] | `[node-id]` | [estados cobertos: vazio, carregando, erro] |

- Todas as telas precisam existir em **tema claro e escuro** — sinalize aqui se o design só cobriu um deles
- Se não houver design ainda, declare explicitamente "sem referência de design" e trate como risco]

## Experiência do usuário

[Descreva a jornada e a experiência do usuário:

- Personas e necessidades
- Fluxos principais e interações
- Considerações e requisitos de UI/UX (incluindo tema claro e escuro)
- Estados de carregamento, vazio e erro de cada tela
- Requisitos de acessibilidade mobile: rótulos para TalkBack/VoiceOver, contraste de texto, alvos de toque mínimos, suporte a aumento de escala de fonte]

## Restrições técnicas de alto nível

[Capture apenas restrições e considerações de alto nível:

- Endpoints do backend já existentes que a funcionalidade consome (o `backend/` é read-only — apenas consulta)
- Integrações externas obrigatórias ou sistemas existentes com os quais interagir
- Exigências de conformidade, regulatórias ou de segurança (dados pessoais, LGPD, armazenamento de token)
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

[- Issue do Linear: `[LIG-XXX]` ou "sem issue vinculada"
- Arquivo do Figma: `[URL]` ou "sem design"
- Documentação da feature a ser produzida: `docs/[nome-da-feature]/`]
