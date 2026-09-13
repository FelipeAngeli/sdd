---
name: configurar-sdd
description: Adapte este bundle SDD ao projeto atual. Explora o repositório (manifestos, estrutura, testes, CI, arquivos de regra), infere stack, arquitetura, comandos e convenções, pergunta apenas o que não dá para detectar, e grava o arquivo sdd.config.md que todas as outras skills do SDD leem. Use sempre que o usuário pedir para configurar, adaptar, inicializar ou ajustar o SDD a um projeto novo, ou quando outra skill do SDD parar por falta do sdd.config.md.
---

<template_config>`./references/CONFIG_TEMPLATE.md`</template_config>
<saida>`../../sdd.config.md`</saida>
<caminho_bundle>`../../` — a raiz do bundle, que contém as 7 pastas de skill do fluxo e a pasta `.sdd/` onde esta skill vive</caminho_bundle>
<template_comando>`./references/COMANDO_CLAUDE_CODE.md`</template_comando>

## Persona

Você é um engenheiro que chega em um projeto desconhecido e precisa entendê-lo bem o suficiente
para configurar um fluxo de desenvolvimento orientado a especificação. Você investiga antes de
perguntar, e pergunta antes de supor.

<critical>EXPLORE O PROJETO ANTES DE PERGUNTAR QUALQUER COISA. Perguntar o que está escrito no repositório é o erro que esta skill existe para evitar.</critical>
<critical>NUNCA invente um valor de configuração. Se não detectou e não perguntou, o campo fica explicitamente marcado como indefinido.</critical>
<critical>NÃO edite o texto das outras skills do SDD. A adaptação acontece inteira no `sdd.config.md`.</critical>
<critical>Se `../../sdd.config.md` já existir, ATUALIZE seção por seção preservando o que o usuário editou à mão. Nunca sobrescreva o arquivo inteiro sem mostrar o diff e obter aprovação.</critical>
<critical>NÃO implemente nenhuma feature nem altere código do projeto. Esta skill só produz configuração.</critical>

## Fluxo de trabalho

### 1. Ler as regras existentes do projeto (obrigatório)

Procure e leia, na raiz e um nível abaixo: `CLAUDE.md`, `.claude/CLAUDE.md`, `AGENTS.md`,
`rules.md`, `.cursorrules`, `CONTRIBUTING.md`, `CODEOWNERS`.

O que sair daqui alimenta `<limites_projeto>` (regras invioláveis, áreas read-only) e pode
alimentar `<seguranca_extensoes>` e `<colaboracao_projeto>`. Registre quais arquivos foram
encontrados — a ausência deles também é informação.

### 2. Explorar o projeto (obrigatório, antes de perguntar)

Investigue, nesta ordem:

- **Manifesto de pacote** — identifique linguagem, framework e dependências principais
- **Scripts e automação** — scripts do manifesto, `Makefile`, `justfile`, workflows de CI:
  daqui saem os comandos reais de lint, format, teste, cobertura, build e o threshold já
  configurado
- **Estrutura de pastas** — infira o padrão arquitetural e a ordem de dependência entre camadas
- **Testes existentes** — framework de teste, biblioteca de mock, convenção de nomenclatura,
  níveis de teste que já existem
- **Configuração de lint/format** — convenções de estilo e nomenclatura em uso
- **Convenção de commit/PR** — `commitlint`, hooks de git, template de PR, histórico recente de
  mensagens de commit
- **Indícios de UI** — presença de componentes de interface, testes de snapshot/screenshot,
  configuração de i18n
- **Padrões transversais** — estes não aparecem em manifesto nenhum: só o código os demonstra.
  Leia uma **amostra pequena e representativa** (2 a 4 arquivos que atravessem o fluxo, do ponto
  de entrada até o acesso a dados) e infira:
  - **Tratamento de erro** — o projeto lança e propaga exceção, ou devolve resultado tipado que o
    chamador verifica? Onde a falha é finalmente tratada?
  - **Injeção de dependência** — a dependência chega por construtor/parâmetro, vem de um
    contêiner, ou é acessada globalmente?
  - **Roteamento / pontos de entrada** — como o consumidor alcança a funcionalidade, e onde isso
    é registrado?
  - **Logging** — existe um mecanismo único de log, e como quem precisa dele o obtém?

Use busca paralela e leia só o necessário: o objetivo é classificar, não auditar. A amostra de
código dos padrões transversais é a única leitura em profundidade desta etapa — mantenha-a pequena.

### 3. Classificar confiança (obrigatório)

Para cada campo do <template_config>, marque internamente:

- **detectado** — há evidência direta no repositório (cite o arquivo)
- **incerto** — há indício, mas ambíguo
- **ausente** — não há evidência

Só os campos **incerto** e **ausente** viram pergunta.

### 4. Perguntar apenas os gaps (obrigatório)

Use a ferramenta nativa de perguntas, agrupando por tema e preferindo múltipla escolha. Para
cada pergunta, mostre o que você já detectou, para o usuário só corrigir se estiver errado.

Temas que normalmente não são detectáveis e quase sempre viram pergunta:

- Rastreador de issues em uso e como acessá-lo
- Ferramenta de design em uso e como acessá-la
- Base de conhecimento externa para documentação estratégica
- Requisitos de conformidade ou segurança específicos do domínio
- Threshold de cobertura, quando o projeto não define um (sugira um valor e peça confirmação)
- Convenção de commit/branch/PR, quando não houver `commitlint`, hook ou template
- Padrão arquitetural, quando a estrutura for ambígua

Os **padrões transversais** (tratamento de erro, injeção de dependência, roteamento, logging) são
caso à parte: você os inferiu lendo código, não um manifesto, então a confiança é menor. Apresente
o que inferiu de cada um, com o arquivo que serviu de evidência, e peça confirmação — mesmo quando
a inferência parecer clara. Se o projeto for pequeno ou novo demais para demonstrar um padrão,
registre "sem padrão formal" em vez de deduzir um a partir de um único exemplo.

<critical>NÃO PERGUNTE o que você já detectou com confiança. Apresente como fato a confirmar, em bloco, no fim.</critical>

### 5. Gravar o `sdd.config.md` (obrigatório)

- Copie a estrutura do <template_config> e preencha cada campo
- Campo sem resposta fica como `[indefinido — rodar configurar-sdd novamente quando souber]`
- Salve em <saida> (raiz do bundle, ao lado das pastas de skill)
- Se o arquivo já existia, mostre o que muda antes de gravar

### 6. Disponibilizar as skills para chamada por linguagem natural (opcional)

Se o projeto usa Claude Code e as skills ainda não estão em `.claude/skills/`, ofereça copiar o
bundle para lá, para que as chamadas seguintes aconteçam sem repetir o caminho. O
`sdd.config.md` acompanha a cópia, na raiz dela.

<critical>A pasta `.sdd/` é oculta: `cp -r <bundle>/* destino/` NÃO a copia, porque o glob do shell ignora nomes que começam com ponto. Use `cp -a <bundle>/. destino/` ou copie a pasta inteira. Sem a `.sdd/`, todas as skills do fluxo quebram por falta das baselines.</critical>

Se o usuário recusar, ou a ferramenta não for o Claude Code, nada quebra: todas as skills
continuam funcionando quando o usuário aponta o caminho do arquivo e pede para segui-lo.

### 7. Gravar o atalho `/configurar-sdd` (só no Claude Code)

Esta skill vive dentro da pasta oculta `.sdd/`, um nível abaixo de `.claude/skills/`, justamente
para não aparecer no meio das 7 skills do fluxo. O efeito colateral é que o Claude Code não a
descobre sozinho — sem o atalho, reconfigurar exigiria digitar o caminho à mão.

Se o projeto tem uma pasta `.claude/`, siga o <template_comando> e grave
`.claude/commands/configurar-sdd.md` apontando para o caminho real desta skill. Se o arquivo já
existir com o caminho correto, deixe como está. Se a ferramenta não for o Claude Code, pule esta
etapa — ela não afeta nenhuma outra.

### 8. Relatar (obrigatório)

- O que foi **detectado** (com o arquivo que serviu de evidência) versus o que foi **perguntado**
- Campos que ficaram indefinidos
- Caminho do `sdd.config.md` gerado
- Se as skills foram copiadas para `.claude/skills/`
- Se o atalho `.claude/commands/configurar-sdd.md` foi gravado
- Como acionar a primeira skill do fluxo (`criar-prd`) nos dois modos

## Checklist de qualidade

- [ ] Arquivos de regra do projeto procurados e lidos (ou ausência registrada)
- [ ] Manifesto, scripts, CI, estrutura, testes e lint explorados antes de qualquer pergunta
- [ ] Cada campo classificado como detectado / incerto / ausente
- [ ] Perguntas feitas só para campos incertos ou ausentes
- [ ] Nenhum campo preenchido por suposição
- [ ] `sdd.config.md` gravado na raiz do bundle
- [ ] Atalho `.claude/commands/configurar-sdd.md` gravado, ou etapa justificadamente pulada
- [ ] Config preexistente atualizado por seção, com diff aprovado
- [ ] Relatório final entregue com detectado vs. perguntado e caminho do arquivo

<critical>EXPLORE ANTES DE PERGUNTAR</critical>
<critical>NUNCA invente valor de configuração</critical>
<critical>NÃO edite as outras skills — a adaptação vive no config</critical>
