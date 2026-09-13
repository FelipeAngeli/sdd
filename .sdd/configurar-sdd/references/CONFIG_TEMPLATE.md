# Configuração do SDD para este projeto

> Gerado pela skill `configurar-sdd`. Pode ser editado à mão a qualquer momento.
> Rodar `configurar-sdd` de novo atualiza seção por seção, preservando edições manuais nas
> seções não afetadas.
>
> **Campo marcado como `[indefinido]` não pode ser adivinhado.** Qualquer skill do SDD que precise
> de um campo indefinido deve parar e perguntar ao usuário, ou pedir que `configurar-sdd` rode de
> novo — nunca inferir o valor a partir do código.

<stack_projeto>
Linguagem(ns): [ex.: TypeScript]
Framework principal: [ex.: React 18 + Vite]
Padrão arquitetural: [ex.: feature-first com hooks; ou "sem padrão formal"]
Gerenciador de pacotes: [ex.: pnpm]
Ordem de dependência entre camadas (usada para sequenciar tasks): [ex.: utils → hooks → componentes → páginas → rotas]
Framework de teste: [ex.: Vitest + Testing Library]
Biblioteca de mock/dublê: [ex.: vi.mock nativo do Vitest]
Tratamento de erro: [como o projeto representa e propaga falha — ex.: exceções tipadas propagadas até a borda, tratadas em um handler central; ou retorno de resultado tipado verificado pelo chamador]
Injeção de dependência: [como uma dependência chega a quem a usa — ex.: por construtor/parâmetro, montada no ponto de entrada; ou contêiner declarativo; ou "sem padrão formal"]
Roteamento / pontos de entrada: [como o consumidor alcança a funcionalidade — ex.: rotas nomeadas registradas em arquivo central; ou handlers descobertos por convenção de pasta; ou "não aplicável"]
Logging: [mecanismo único de log do projeto e como é obtido — ex.: logger estruturado injetado por módulo; ou "sem mecanismo definido"]
</stack_projeto>

<comandos_projeto>
Lint: [comando]
Format: [comando]
Testes: [comando]
Cobertura: [comando] — threshold: [ex.: 80% por linha; ou "sem gate definido"]
E2E: [comando; ou "não aplicável"]
Build: [comando]
Gate agregado (roda tudo que bloqueia entrega): [comando; ou "não existe — rodar lint, testes e cobertura em sequência"]
Scanner de vulnerabilidade de dependências: [ex.: pnpm audit]
</comandos_projeto>

<integracoes_projeto>
Rastreador de issues: [Linear | Jira | GitHub Issues | nenhum] — como acessar: [ex.: MCP do Linear; ou "não disponível"]
Ferramenta de design: [Figma | nenhuma] — como acessar: [ex.: MCP do Figma]
Base de conhecimento externa: [ex.: Obsidian, Notion; ou nenhuma]
Outras integrações relevantes: [ex.: ferramenta de observabilidade, serviço de notificação; ou "nenhuma"]
</integracoes_projeto>

<ui_projeto>
Tem interface de usuário final? [sim | não]
Tipo: [web | mobile | desktop | CLI | não aplicável]
Ferramenta de validação visual: [ex.: Playwright screenshot; ex.: golden test; ou "não aplicável"]
Acessibilidade aplicável: [sim — padrão alvo (ex.: WCAG 2.1 AA) | não]
i18n aplicável: [sim — como as strings são gerenciadas | não]
</ui_projeto>

<limites_projeto>
Áreas read-only ou fora de escopo: [ex.: `vendor/` é gerado, não editar; ou "nenhuma"]
Regras invioláveis do projeto (de CLAUDE.md / AGENTS.md / rules.md / CONTRIBUTING.md): [liste, ou "nenhuma encontrada"]
Arquivos de regra lidos na configuração: [caminhos; ou "nenhum encontrado"]
</limites_projeto>

<seguranca_extensoes>
[Regras de segurança específicas deste projeto, que somam ao `.sdd/_shared/SECURITY_BASELINE.md`.
Ex.: "token de sessão só em armazenamento seguro do sistema", "dados de usuário sujeitos a
proteção de dados pessoais". Se não houver, escreva "nenhuma além da baseline".]
</seguranca_extensoes>

<qualidade_extensoes>
[Convenções de estilo/nomenclatura específicas, que somam ao `.sdd/_shared/QUALITY_BASELINE.md`.
Ex.: regras relevantes da configuração de lint detectada. Se não houver, escreva
"nenhuma além da baseline".]
</qualidade_extensoes>

<colaboracao_projeto>
Convenção de commit: [ex.: Conventional Commits; ou "livre"] — detectada em: [ex.: commitlint.config.js; ou "perguntado ao usuário"]
Convenção de branch: [ex.: feature/<slug>; ou "sem convenção formal"]
Processo de PR: [ex.: template obrigatório, 1 aprovação, squash merge; ou "sem processo formal"]
Política de comentários no código: [ex.: comentar só onde a intenção não é óbvia]
</colaboracao_projeto>

<documentacao_projeto>
Onde documentar cada feature: [ex.: `docs/<feature>/`; ou "não aplicável"]
Registro externo obrigatório: [ex.: página na base de conhecimento com link de volta; ou "não"]
Idioma da documentação: [ex.: português]
</documentacao_projeto>

<artefatos_sdd>
Pasta raiz dos artefatos por feature: [padrão: `tasks/prd-<nome-da-feature>/`]
Arquivos por feature: run.yaml · prd.md · techspec.md · tasks.md · <num>_task.md · bugs.md · bugfixes.md · qa.md · codereview.md · memoria/<skill>.md · evidences/ · spike.md (só quando houver spike)
</artefatos_sdd>

<meta_sdd>
Versão do bundle SDD: [conteúdo do arquivo `VERSION` na raiz do bundle, no momento da configuração]
Data da última configuração: [AAAA-MM-DD]
Caminho do bundle neste projeto: [ex.: `.claude/skills/`; ou `sdd/`]
</meta_sdd>
