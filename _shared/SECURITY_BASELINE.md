# Baseline de segurança (universal)

Checklist obrigatório, válido para qualquer linguagem, framework ou plataforma. É carregado
pelas skills de TechSpec, execução de task, bugfix, QA e code review.

Regras específicas deste projeto ficam em `<seguranca_extensoes>` do `sdd.config.md` e **somam**
a este arquivo — nunca o substituem. Este arquivo não deve ser editado por projeto.

## Segredos e credenciais

- [ ] Nenhuma chave, token, senha, certificado ou connection string hardcoded no código-fonte
- [ ] Nenhum segredo versionado (inclusive em arquivos de exemplo, fixtures, testes e logs de CI)
- [ ] Segredos vêm de variável de ambiente ou cofre; o repositório contém apenas o arquivo de
      exemplo sem valores reais
- [ ] Nenhum segredo impresso em log, mensagem de erro ou resposta de API

## Dependências vulneráveis

- [ ] O scanner de vulnerabilidades da stack foi executado (o comando correto está em
      `<comandos_projeto>`; se não houver, use o scanner padrão do gerenciador de pacotes do
      projeto — ex.: `npm audit`, `pip-audit`, `govulncheck`, `cargo audit`, `bundler-audit`)
- [ ] Achados de severidade **alta** ou **crítica** são tratados como bloqueantes
- [ ] Nenhuma dependência nova foi adicionada sem justificativa registrada na TechSpec
- [ ] O lockfile está commitado e coerente com o manifesto

## Injeção e validação de entrada

- [ ] Toda entrada vinda de fora (usuário, API, arquivo, variável de ambiente, deep link) é
      validada antes do uso
- [ ] Consultas a banco são parametrizadas — nenhuma concatenação de string com entrada do usuário
- [ ] Nenhuma entrada do usuário chega a um interpretador de comando do sistema sem sanitização
- [ ] Saída é codificada para o contexto em que é renderizada (evita XSS e equivalentes)
- [ ] Caminhos de arquivo derivados de entrada são normalizados e restritos (evita path traversal)
- [ ] Requisições a URLs derivadas de entrada são restritas a destinos permitidos (evita SSRF)
- [ ] Desserialização de dados não confiáveis usa formato e tipos restritos

## Autenticação e autorização

- [ ] Verificação de permissão acontece no servidor/camada de confiança, nunca só no cliente
- [ ] Toda rota, endpoint ou operação sensível tem verificação explícita de quem pode executá-la
- [ ] Identificadores previsíveis não dão acesso a recurso de outro usuário (referência direta
      insegura a objeto)
- [ ] Sessão/token tem expiração e é invalidado no logout

## Dados sensíveis

- [ ] Dados pessoais, credenciais e tokens nunca aparecem em log em texto claro
- [ ] Dados sensíveis em repouso usam o armazenamento seguro da plataforma
- [ ] Tráfego sensível trafega criptografado
- [ ] Mensagens de erro expostas ao usuário não vazam detalhe interno (stack trace, query, caminho)

## Configuração segura

- [ ] Modo debug/verbose desligado fora de ambiente de desenvolvimento
- [ ] CORS e headers de segurança restritos ao necessário
- [ ] Permissões e escopos seguem o princípio do menor privilégio
- [ ] Nenhum endpoint de diagnóstico/administração exposto publicamente sem autenticação

## Extensões do projeto

- [ ] As regras de `<seguranca_extensoes>` do `sdd.config.md` foram lidas e verificadas
- [ ] Requisitos de conformidade declarados no config (ex.: proteção de dados pessoais, setor
      regulado) foram endereçados
