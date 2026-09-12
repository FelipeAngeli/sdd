# Baseline de qualidade de código (universal)

Checklist obrigatório, válido para qualquer linguagem ou framework. É carregado pelas skills de
TechSpec, execução de task, bugfix, QA e code review.

Convenções específicas da linguagem ou do time ficam em `<qualidade_extensoes>` do
`sdd.config.md` e **somam** a este arquivo. Este arquivo não deve ser editado por projeto.

## Estrutura e complexidade

- [ ] Funções e métodos têm uma responsabilidade só e cabem na tela
- [ ] Aninhamento profundo foi achatado (retorno antecipado em vez de cascata de condicionais)
- [ ] Arquivo que cresceu demais foi dividido por responsabilidade, não por camada técnica
- [ ] Nenhuma função recebe mais parâmetros do que consegue justificar

## Reuso

- [ ] O que já existe no projeto foi reusado antes de criar algo novo
- [ ] Nenhuma duplicação evitável de lógica; regra de negócio vive em um lugar só
- [ ] Abstração nova só foi criada com uso real comprovado (nada especulativo)

## Nomenclatura

- [ ] Nomes descrevem intenção, não implementação
- [ ] Convenção da linguagem/projeto seguida de forma consistente (ver `<qualidade_extensoes>`
      e a configuração de lint do projeto)
- [ ] Nenhuma abreviação obscura nem nome genérico (`data`, `info`, `handle`, `temp`)

## Tratamento de erro

- [ ] Erros são tratados explicitamente, nunca engolidos em silêncio
- [ ] O tipo de erro carrega informação suficiente para o chamador decidir o que fazer
- [ ] Falhas esperadas (rede, validação, ausência de permissão) têm caminho definido na UI/API
- [ ] Nenhum `catch` genérico que esconde defeito de programação

## Comentários e código morto

- [ ] Comentário explica o porquê, não o que o código já diz
- [ ] Nenhum código comentado, nenhum TODO órfão sem dono ou referência
- [ ] A política de comentários do projeto (`<colaboracao_projeto>`) foi respeitada

## Performance

- [ ] Nenhum trabalho redundante em caminho quente (recálculo, requisição ou leitura repetida)
- [ ] Coleções grandes não são carregadas inteiras quando um recorte resolve
- [ ] Recursos (conexões, arquivos, listeners, timers) são liberados

## Qualidade dos testes

- [ ] Testes verificam comportamento observável, não detalhe de implementação
- [ ] Todo caminho feliz tem o caminho de falha correspondente testado
- [ ] Nenhuma asserção fraca isolada (só "não é nulo", só "não lançou exceção")
- [ ] Nome do teste descreve o comportamento esperado e a condição
- [ ] Mock aplicado na fronteira certa — a unidade sob teste continua sendo exercitada de verdade
- [ ] O threshold de cobertura de `<comandos_projeto>` é tratado como absoluto: se faltar
      cobertura, escreve-se mais teste; **nunca** se baixa o threshold

## Extensões do projeto

- [ ] As convenções de `<qualidade_extensoes>` do `sdd.config.md` foram lidas e verificadas
