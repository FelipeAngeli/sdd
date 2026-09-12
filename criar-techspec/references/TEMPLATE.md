# Especificação técnica

## Resumo executivo

[Fornecer uma visão técnica breve da abordagem da solução. Resumir as principais decisões de arquitetura e a estratégia de implementação em 1–2 parágrafos.]

## Arquitetura do sistema

### Visão dos componentes

[Descrição breve dos principais componentes e suas responsabilidades:

- Nomes dos componentes e funções principais **Certifique-se de listar cada componente novo ou modificado**
- Principais relacionamentos entre componentes
- Visão geral do fluxo de dados]

### Mapeamento por camada

<critical>
**OBRIGATÓRIO.** Toda funcionalidade precisa declarar em qual camada cada arquivo vive, usando
as camadas e a ordem de dependência declaradas em `<stack_projeto>` do `sdd.config.md`. As
regras de direção de dependência do projeto não podem ser violadas: nenhuma camada pode importar
de uma camada que dependa dela.
</critical>

| Camada | Arquivo | Responsabilidade | Novo ou modificado |
| --- | --- | --- | --- |
| [camada do config] | `[caminho real no projeto]` | [responsabilidade] | novo/modificado |

[Uma linha por arquivo. Se o projeto não tem camadas formais, agrupe por responsabilidade
(entrada, regra de negócio, acesso a dados, apresentação) e declare isso explicitamente.]

### Navegação e registro de dependências

[- Pontos de entrada/rotas registrados seguindo o padrão de roteamento declarado em
  `<stack_projeto>` — **sem instanciar dependência direto no ponto de entrada**
- Dependências novas e o escopo de cada uma (instância única, por chamada, etc.), seguindo o
  padrão de injeção de dependência do projeto
- Guards/middlewares de acesso aplicáveis
- Módulo/camada pai que expõe essas dependências, quando a arquitetura do projeto tiver essa
  noção]

### Reuso do código compartilhado do projeto

[O que já existe e será reusado em vez de recriado: componentes de UI, tema/estilos, cliente
HTTP, tipos de erro, logger, armazenamento — conforme mapeado na análise exploratória.
**Justifique cada abstração nova introduzida no código compartilhado.**]

## Design de implementação

### Principais interfaces

[Definir os principais contratos de código **na linguagem do projeto** (declarada em
`<stack_projeto>`), com no máximo 20 linhas por bloco de código:

- O contrato entre a camada que orquestra a regra de negócio e a camada que acessa dados
  externos (repositório, cliente HTTP, storage)
- Uma unidade de regra de negócio por bloco, com a assinatura de entrada/saída e o tipo de erro
  que ela pode retornar

Indique a linguagem do bloco de código (ex.: ` ```typescript `) para manter o realce de sintaxe.]

### Máquina de estados

[Declarar a máquina de estados da funcionalidade. Todo fluxo precisa cobrir carregando, sucesso,
**vazio** e **erro** — não só o caminho feliz.

| Estado | Quando é emitido | O que a interface mostra |
| --- | --- | --- |
| `[NomeInitial]` | [Condição] | [Comportamento inicial] |
| `[NomeLoading]` | [Condição] | [Indicador de carregamento, ação desabilitada] |
| `[NomeSuccess]` | [Condição] | [Conteúdo] |
| `[NomeEmpty]` | [Condição] | [Estado vazio] |
| `[NomeError]` | [Condição] | [Mensagem e ação de retry] |

Se a stack não usa máquina de estados explícita, declare os mesmos quatro casos como
comportamento esperado da camada de apresentação/saída.]

### Modelos de dados

[Preencher apenas quando a feature consome APIs. Caso contrário, declare "não aplicável" e pule
o restante desta seção.]

<critical>
**OBRIGATÓRIO — formatação visual e legível:**

- **NÃO** descrever contratos como tipos inline em bullet points (ex.: `location: { name, admin1, ... }`).
- **SEMPRE** documentar cada entidade/contrato em subseção própria (`#### \`NomeDoTipo\` — descrição`).
- **SEMPRE** incluir tabela de campos (Campo | Tipo | Obrigatório | Descrição) antes do exemplo formatado.
- **SEMPRE** incluir um bloco de código formatado (ex.: ` ```json `), indentado, com valores realistas (não placeholders genéricos).
- **SEMPRE** indicar o tipo correspondente na linguagem do projeto e se o campo é opcional/nulo no modelo interno.
- Variantes/degradações (ex.: campo ausente ou nulo) devem ter exemplo separado + blockquote explicando o comportamento.
- Envelopes de erro: tabela Código | Status | Tipo de erro no projeto + exemplo formatado.
- Mapeamentos entre payload e as estruturas internas: tabela Origem | Destino.
- Campos ausentes no payload e como são normalizados — mencionar isso no parágrafo introdutório.
</critical>

Contratos de dados consumidos da API — prontos para mapear nas estruturas internas e exibir na
interface. [Completar contexto específico da feature. Declarar como campos ausentes no payload
são normalizados.]

#### `[NomeDoTipo]` — [descrição curta]

| Campo | Tipo no payload | Tipo no projeto | Obrigatório | Descrição |
| --- | --- | --- | --- | --- |
| `[campo]` | `[string/number/...]` | `[tipo na linguagem do projeto]` | sim/não | [Descrição] |

```json
{
  "[campo]": "[valor realista]"
}
```

[Repetir o padrão acima para cada entidade/contrato principal: payload agregado, tipos de entrada, tipos de erro, etc.]

> **[Variante/degradação (se aplicável)]:** [Explicar quando ocorre e o impacto no payload.]

```json
{
  "[secao_afetada]": null
}
```

#### `[NomeDoErro]` — envelope de erro tipado

| Código | Status | Tipo de erro no projeto | Significado |
| --- | --- | --- | --- |
| `[codigo]` | `[status]` | `[tipo de erro declarado na arquitetura do projeto]` | [Descrição] |

```json
{
  "error": {
    "code": "[codigo]",
    "message": "[mensagem conforme padrão do serviço]"
  }
}
```

#### Mapeamento entre payload e estruturas internas

| Origem (payload) | Destino (modelo de dados) | Destino (entidade/estrutura de domínio) |
| --- | --- | --- |
| `[campo_origem]` | `[campoModelo]` | `[campoEntidade]` |

## Contratos de API consumidos

[Preencher apenas quando a feature consome APIs. Caso contrário, declare "não aplicável" e pule
o restante desta seção.]

<critical>
Se a fonte de verdade do contrato for acessível (código do serviço, especificação de contrato —
ex.: OpenAPI —, ou schema), confirme cada rota nela antes de documentar aqui — não suponha.
Respeite as áreas read-only declaradas em `<limites_projeto>`: consultar sim, editar não.
</critical>

<critical>
**OBRIGATÓRIO — formatação visual e legível:**

- **NÃO** listar rotas como bullet points compactos com tudo inline.
- **SEMPRE** começar com tabela de visão geral (Método | Rota | Autenticação | Componente do projeto que consome).
- **SEMPRE** documentar cada rota em subseção própria (`#### \`MÉTODO /rota\``).
- **SEMPRE** incluir, por rota: tabela de query params/body, tabela de respostas (Status | Corpo | Tipo de erro resultante | Quando), bloco ` ```http ` com a requisição e blocos ` ```json ` com exemplos de resposta.
- Cobrir **todos** os cenários relevantes: sucesso, lista vazia, erro de validação, erro do servidor, timeout, sem conectividade.
- Usar blockquote (`>`) para comportamentos não óbvios (ex.: "lista vazia não é erro HTTP — a interface mostra estado vazio").
- Separar rotas com `---`.
</critical>

### Visão geral

| Método | Rota | Autenticação | Componente do projeto que consome | Origem no serviço |
| --- | --- | --- | --- | --- |
| `[GET/POST/...]` | `[ /rota ]` | [ex.: Bearer/pública] | `[componente que faz a chamada]` | `[caminho no código-fonte do serviço, quando acessível]` |

---

#### `[MÉTODO] [ /rota ]`

[Descrição breve do propósito da rota.]

**Query params** _(ou **Body** para POST/PUT/PATCH)_

| Param | Tipo | Default | Regras (conforme a fonte de verdade do contrato) |
| --- | --- | --- | --- |
| `[param]` | `[tipo]` | `[default ou —]` | [Validações e regras] |

**Respostas**

| Status | Corpo | Tipo de erro resultante | Quando |
| --- | --- | --- | --- |
| `[200]` | `[TipoResposta]` | — | [Condição de sucesso] |
| `[400/422]` | `[TipoErro]` | `[erro de validação declarado na arquitetura do projeto]` | [Erro de validação com campo] |
| `[401]` | `[TipoErro]` | `[erro de autenticação declarado na arquitetura do projeto]` | [Token ausente/expirado] |
| `[500]` | `[TipoErro]` | `[erro de servidor declarado na arquitetura do projeto]` | [Falha do servidor] |
| — | — | `[erro de rede declarado na arquitetura do projeto]` | [Timeout / sem conectividade] |

**Exemplo — sucesso**

```http
[MÉTODO] [ /rota?param=valor ]
Authorization: Bearer [token]
```

```json
{
  "[campo]": "[valor realista]"
}
```

**Exemplo — [cenário alternativo, ex.: lista vazia]**

```http
[MÉTODO] [ /rota?param=valor ]
```

```json
{
  "[corpo]": []
}
```

> [Nota sobre o comportamento resultante na interface, se aplicável.]

**Exemplo — [cenário de erro]**

```http
[MÉTODO] [ /rota ]
```

```json
{
  "error": {
    "code": "[codigo]",
    "message": "[mensagem]"
  }
}
```

[Repetir o padrão acima para cada rota. Para payloads grandes já documentados em Modelos de dados, referenciar o exemplo JSON existente em vez de duplicar.]

---

## Design e tokens

[Preencher apenas se `<integracoes_projeto>` indicar uma ferramenta de design ativa (ex.: Figma)
e o PRD tiver referências de design na seção "Referências de design". Caso contrário, declare
"não aplicável" e pule o restante desta seção.

| Tela | Referência | Elemento de teste visual correspondente |
| --- | --- | --- |
| `[Nome da tela]` | `[identificador]` | `[caminho do teste, conforme <ui_projeto>]` |

**Tokens** — reusar token existente sempre que possível:

| Token do design | Equivalente no projeto | Ação |
| --- | --- | --- |
| `[color/primary]` | `[...]` | reusar / criar |

- Divergências entre design e implementação atual e a decisão tomada
- Assets a baixar e destino no projeto]

## Pontos de integração

[Incluir apenas se a funcionalidade exigir integrações externas:

- Serviços, SDKs ou APIs externos (ex.: mapas, pagamento, push, analytics)
- Requisitos de autenticação e onde o token é lido
- Permissões ou configuração de plataforma necessárias, conforme `<ui_projeto>` (ex.: manifesto Android / Info.plist no iOS)
- Abordagem de tratamento de erros]

## Abordagem de testes

<critical>
Threshold de cobertura do projeto: ver `<comandos_projeto>`. Popule esta seção com o **máximo de
casos de teste possível** — ela é a entrada direta do `criar-tasks`.
</critical>

<critical>
Antes de escrever qualquer cenário de teste, audite os testes existentes do projeto: mapeie o
que já está coberto por camada e reuse os helpers/fixtures encontrados na análise exploratória —
só então declare os cenários novos. Use a biblioteca de mock/dublê declarada em `<stack_projeto>`
como única forma de dublê.
</critical>

### Níveis de teste

[Uma subseção por nível de teste suportado pela stack, conforme `<stack_projeto>`. Para cada
nível, liste os cenários concretos — caminho feliz **e** caminho de falha.

Níveis comuns, use os que se aplicam:

- **Unitário** — regra de negócio isolada, sem dependência externa
- **Integração entre componentes** — a unidade com suas colaboradoras reais
- **Contrato/API** — request e response conferidos contra a fonte de verdade
- **Interface** — cada estado renderiza o que deve, incluindo vazio, carregando e erro
- **Visual** — apenas se `<ui_projeto>` indicar ferramenta de validação visual
- **Ponta a ponta** — apenas os fluxos principais do usuário

Para cada cenário de falha, especifique o erro concreto esperado, não uma exceção genérica.]

### Meta de cobertura

[Threshold do projeto: ver `<comandos_projeto>`. Declare a meta por área e como verificar.
Se o projeto não tem gate de cobertura, declare "sem gate" e liste mesmo assim os cenários
obrigatórios.]

## Sequenciamento do desenvolvimento

### Ordem de construção

[Definir sequência de implementação seguindo as camadas declaradas em `<stack_projeto>` —
dependências antes dos dependentes. Sequência genérica de exemplo, adapte às camadas reais do
projeto:

1. Camada de domínio/regra de negócio — testável isoladamente, sem dependência de framework
2. Camada de acesso a dados — modelos/mapeadores, cliente do serviço externo, implementação do repositório
3. Camada de apresentação — máquina de estados, depois componentes, depois a tela/rota
4. Registro de módulo/dependências e rotas
5. Testes de integração/ponta a ponta do fluxo completo]

### Dependências técnicas

[Listar bloqueadores de dependências:

- APIs/serviços consumidos já disponíveis? em qual ambiente?
- Design finalizado, quando `<integracoes_projeto>` indicar ferramenta de design ativa (cobrindo todos os temas suportados, se houver)?
- Dependências/pacotes novos no manifesto do projeto — justificar cada um
- Permissões ou configuração de plataforma necessárias, conforme `<ui_projeto>`]

## Monitoramento e observabilidade

[Definir abordagem usando a infraestrutura existente do projeto:

- Eventos de analytics a registrar (nome e propriedades), se aplicável
- Erros reportados à ferramenta de observabilidade do projeto (ex.: Sentry, Crashlytics) e com qual contexto
- Logs via o mecanismo de logging já existente no projeto — nível apropriado e redaction obrigatória de dado sensível (nunca logar token, credencial ou dado pessoal em claro)
- Como diagnosticar a funcionalidade em produção]

## Considerações técnicas

### Principais decisões

[Documentar decisões técnicas importantes:

- Escolha da abordagem e justificativa
- Trade-offs considerados
- Alternativas descartadas e por quê]

### Riscos conhecidos

[Identificar riscos técnicos:

- Desafios potenciais
- Abordagens de mitigação
- Áreas que precisam de pesquisa]

### Segurança

[Verifique contra `_shared/SECURITY_BASELINE.md` e registre aqui o que é específico desta
feature, além das regras de `<seguranca_extensoes>`.]

### Conformidade com as regras do projeto

[Listar as regras aplicáveis e como a especificação as respeita:

- Regras invioláveis registradas em `<limites_projeto>` do config
- `_shared/SECURITY_BASELINE.md` e `_shared/QUALITY_BASELINE.md`
- Convenções específicas de `<qualidade_extensoes>` e `<seguranca_extensoes>`
- Desvios, se houver, com justificativa]

### Documentação da feature

[Conforme `<documentacao_projeto>` do config:

- Local de documentação técnica da feature e idioma
- Se `<integracoes_projeto>` indicar uma base de conhecimento externa ativa (ex.: Obsidian,
  Notion), registre também lá, com link de volta para a documentação técnica. Caso contrário,
  pule esta etapa.

Se `<documentacao_projeto>` indicar "não aplicável", declare isso e siga.]

### Arquivos relevantes e dependentes

[Listar aqui os arquivos relevantes e dependentes, separados por camada e incluindo os arquivos de teste correspondentes]
