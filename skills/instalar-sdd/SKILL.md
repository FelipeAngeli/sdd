---
name: instalar-sdd
description: Copia o bundle SDD deste plugin para o projeto atual e entrega para configurar-sdd. Use ao instalar o SDD pela primeira vez num projeto via plugin do Claude Code — gatilhos como "instala o SDD aqui", "traz o bundle SDD para este projeto".
---

<origem>`$CLAUDE_PLUGIN_ROOT` — a raiz deste plugin, resolvida pelo Claude Code em tempo de execução; contém as 7 pastas do fluxo, `.sdd/`, `qa/` e `VERSION`, no mesmo layout do bundle mestre</origem>

## Persona

Você está automatizando o passo manual que o `README.md` do bundle já descreve: copiar as pastas
certas para o projeto do usuário e entregar para `configurar-sdd`. Você não configura nada
sozinho — só copia e entrega.

<critical>Copie SEMPRE nomeando as pastas, uma por uma. Nunca `cp -a "$CLAUDE_PLUGIN_ROOT"/. destino/` nem `cp -r "$CLAUDE_PLUGIN_ROOT"/* destino/` — os dois arrastariam arquivos que não fazem parte do bundle portátil, como `.claude-plugin/` e esta própria skill (`skills/instalar-sdd/`).</critical>
<critical>NÃO configure nada nesta skill: nenhuma pergunta de stack, comando ou convenção. Ao final, entregue para `configurar-sdd` e pare — configurar é responsabilidade dela, não desta skill.</critical>

## Fluxo

### 1. Perguntar o destino

Pergunte ao usuário onde a cópia deve ficar:

- raiz do projeto, numa pasta `sdd/`
- direto em `.claude/skills/`

### 2. Perguntar sobre QA contínuo

Pergunte se o projeto já sabe que quer a camada opcional de QA contínuo (pasta `qa/`). Se o
usuário não souber, diga que dá para ativar depois — `configurar-sdd` pergunta de novo, e
`atualizar-sdd` traz a pasta quando for a hora — e siga sem copiar `qa/` agora.

<critical>Se `<DESTINO>/.sdd` já existir, PARE. Este projeto já tem uma cópia do SDD — sobrescrevê-la apagaria silenciosamente qualquer skill editada pelo time. Aponte o usuário para `atualizar-sdd`, que compara, mostra o diff e decide arquivo por arquivo, e não copie nada nesta situação.</critical>

### 3. Copiar

Com o destino escolhido em `<DESTINO>`, crie a pasta se não existir e copie, pasta por pasta:

```sh
mkdir -p <DESTINO>
cp -R "$CLAUDE_PLUGIN_ROOT/criar-prd" "$CLAUDE_PLUGIN_ROOT/criar-techspec" "$CLAUDE_PLUGIN_ROOT/criar-tasks" \
      "$CLAUDE_PLUGIN_ROOT/executar-task" "$CLAUDE_PLUGIN_ROOT/executar-qa" "$CLAUDE_PLUGIN_ROOT/executar-bugfix" \
      "$CLAUDE_PLUGIN_ROOT/executar-review" "$CLAUDE_PLUGIN_ROOT/.sdd" "$CLAUDE_PLUGIN_ROOT/VERSION" <DESTINO>/
```

Se o usuário pediu QA contínuo na etapa 2, copie também:

```sh
cp -R "$CLAUDE_PLUGIN_ROOT/qa" <DESTINO>/
```

### 4. Entregar para `configurar-sdd`

Leia `<DESTINO>/.sdd/configurar-sdd/SKILL.md` e siga integralmente as instruções dessa skill para
configurar o SDD neste projeto. A skill `instalar-sdd` termina aqui — tudo que vem depois
(explorar o projeto, perguntar gaps, gravar `sdd.config.md`, atalhos) é responsabilidade dela.

## Checklist de qualidade

- [ ] Destino perguntado e confirmado antes de copiar
- [ ] QA contínuo perguntado antes de copiar, não depois
- [ ] Cópia feita pasta por pasta, nomeada — nunca o diretório inteiro
- [ ] `.claude-plugin/` e `skills/instalar-sdd/` (esta própria skill) nunca copiados para o destino
- [ ] Entregue para `configurar-sdd` ao final, sem configurar nada nesta skill
- [ ] `<DESTINO>/.sdd` conferido antes de copiar — se já existia, a skill parou e apontou para `atualizar-sdd`, sem sobrescrever nada
