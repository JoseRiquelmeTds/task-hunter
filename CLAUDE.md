# Task-Hunter

Conjunto de skills do Claude Code que encontra, prioriza e executa tasks de backend do Jira **IDD** no repositório **edgebr/dell-ddm**, e entrega cada uma como **PR em draft**. O objetivo é que o trabalho do José vire revisão.

Este repositório guarda as skills, os templates e as regras do Task-Hunter. A especificação completa e viva está no doc "Task-Hunter — Descrição do Projeto". Este arquivo é a versão que o Claude Code lê ao trabalhar aqui.

## Contexto

- O uso do Claude nas tasks já é recomendado no trabalho. O que muda é o grau de automação.
- Escopo: tasks de backend do catálogo, **não atribuídas** e com status **"Tarefas pendentes"**.
- No IDD, a label "backend" nem sempre existe. Classificar pelo conteúdo da task, não só pela tag.
- O agente decide sozinho durante a execução, mas registra cada decisão em disco. Ele nunca entrega nada além de um draft.

## Comandos

Todos são skills com prefixo `th-`, para não colidir com `/resume` e `/clear`, que são nativos do Claude Code.

| Comando | O que faz | Escreve em |
| --- | --- | --- |
| `/th-hunt` | Busca no Jira, filtra, julga se a task pode ser feita sem VDI e afins, rankeia e pergunta se quero iniciar e quantas | `.task-hunter/ranking.md` |
| `/th-work [n]` | Inicia as N primeiras do ranking, cada uma num subagente com worktree próprio. Atribui a task a mim, muda para "Em desenvolvimento", implementa, registra decisões e abre o PR em draft | `.task-hunter/<IDD-xxx>/`, Jira, GitHub |
| `/th-resume [IDD-xxx]` | Para cada task (ou uma específica): o que a task pedia, explicação da solução, o que divergiu do plano, decisões tomadas e pontos de menor confiança | só leitura |
| `/th-clear-all` | Apaga todas as pastas de task em `.task-hunter/` (pede confirmação) | `.task-hunter/` |
| `/th-clear <IDD-xxx>` | Apaga a pasta de uma task | `.task-hunter/<IDD-xxx>/` |

Os clears mexem só nos arquivos locais. Eles não fecham PR, não removem worktree com trabalho não commitado e não alteram o Jira.

## Estrutura deste repositório

```
task-hunter/
  CLAUDE.md
  .claude/
    settings.json            # permissions.deny e hooks (ver "Requisitos mandatórios")
    skills/
      th-hunt/SKILL.md
      th-work/SKILL.md
      th-resume/SKILL.md
      th-clear-all/SKILL.md
      th-clear/SKILL.md
  templates/
    ranking.md
    plan.md
    decisions.md
  rules/
    exclusion-signals.md     # pendente
    ranking-rubric.md        # pendente
    stop-conditions.md       # pendente
```

As skills rodam dentro do repositório dell-ddm. Por isso, elas precisam ser instaladas como skills de usuário (por exemplo, com symlink de `.claude/skills/th-*` para `~/.claude/skills/`) ou copiadas para o `.claude/` do dell-ddm. **Pendente:** escolher o método.

## Arquitetura

- **Núcleo:** a sessão principal do Claude Code. Lê o `ranking.md`, cria os worktrees, dispara os subagentes, acompanha o `status` de cada um e consolida o resultado. Só o núcleo dispara subagentes.
- **Subagentes:** um por task, cada um em um **git worktree** próprio e numa branch própria, para que dois subagentes não sobrescrevam o trabalho um do outro.
- **Estado em disco:** tudo fica em `.task-hunter/`, no dell-ddm. Essa pasta vai no `.gitignore`. Os comandos podem rodar em sessões diferentes.

```
.task-hunter/
  ranking.md          # saída do /th-hunt, incluindo o motivo de cada exclusão
  IDD-123/
    task.md           # snapshot do ticket no momento do início
    plan.md           # plano escrito antes de codar
    decisions.md      # decisões registradas na hora, não no fim
    status            # pending | running | blocked | draft-opened
```

### Formato do `decisions.md`

Cada entrada tem: **contexto**, **opções consideradas**, **escolha**, **motivo** e **nível de confiança** (alta, média ou baixa). Toda divergência em relação ao `plan.md` vira uma entrada marcada como `DIVERGÊNCIA`. O `/th-resume` lê principalmente este arquivo.

## Jira (via MCP da Atlassian)

1. **Filtro por JQL:** projeto IDD, sem responsável, status "Tarefas pendentes" (na JQL, use o id: `status = 10023`; o nome não é aceito).
2. **Julgamento pelo modelo:** a task é de backend (independente da label)? Pode ser feita sem VDI ou outra restrição? Algumas restrições são substituíveis (mock, docker, dado local) e outras são impeditivas. Essa avaliação acontece no `/th-hunt` e segue `rules/exclusion-signals.md`.
3. **Ao iniciar uma task:** atribuir ao José e mudar o status de "Tarefas pendentes" para "Em desenvolvimento". Isso é feito pelo núcleo **antes** de disparar o subagente.

O agente não comenta no ticket e não faz nenhuma outra transição de status.

## Requisitos mandatórios

- **Revisão humana obrigatória.** O agente só abre PR em **draft** (`gh pr create --draft`). Nunca tira do draft, nunca pede review e nunca faz merge.
- **No máximo 2 subagentes em paralelo.** Acima disso, o `/th-work` recusa ou enfileira.
- **Permissões mínimas:** git em branches de feature, rodar testes, `gh pr create --draft` e as duas operações permitidas no Jira.
- **Bloqueios explícitos** (`permissions.deny` em `.claude/settings.json` e hooks):
  - push na `main` ou em branches protegidas
  - `git push --force`
  - comandos contra bancos ou ambientes reais
  - `rm -rf` fora da pasta da própria task
  - transições no Jira além de atribuir e mover para "Em desenvolvimento"
- **Condição de parada:** quando uma condição de `rules/stop-conditions.md` for atingida, o subagente marca `blocked`, explica o motivo no `decisions.md` e **não** abre PR.
- **Transparência no PR:** a descrição do draft resume a task e as principais decisões, com referência ao ID da task.

## Pendências de definição

- [x] **Sinais de exclusão** (`rules/exclusion-signals.md`). Todos impeditivos, exceto dependências que sobem localmente com docker (substituível). O Docker fica disponível para o agente, só para recursos locais.
- [x] **Rubrica de ranking** (`rules/ranking-rubric.md`). Pesos: sprint 4, prioridade 4, desbloqueio 4, clareza 3, tamanho 2, idade 4. Desempate pela task mais antiga.
- [ ] **Condições de parada** (`rules/stop-conditions.md`). Rascunho: descrição ambígua demais; escopo acima do limite; mudança de schema ou migration; contrato de API com outro time; testes que não rodam localmente.
- [ ] **Convenções do dell-ddm.** Levantar o padrão de nome de branch, a mensagem de commit com o ID da task, o template de PR e o comando da suíte de testes.
- [ ] **Instalação das skills.** Decidir entre symlink em `~/.claude/skills/` e cópia para o dell-ddm.

## Ordem de construção

1. `/th-hunt` só leitura. Validar se o ranking bate com a escolha manual. Ainda não atribui nem muda status.
2. `/th-work` com 1 task, sem paralelismo. Estabilizar `plan.md` e `decisions.md` e validar as operações no Jira.
3. `/th-resume`, `/th-clear` e `/th-clear-all`.
4. Paralelismo com worktrees, até 2 subagentes.

## Como trabalhar neste repositório

- Escrever skills, templates e regras em português.
- Cada `SKILL.md` deve ter um frontmatter com `name` e `description`, e passos numerados e verificáveis.
- Não implementar uma etapa da ordem de construção antes da anterior estar validada pelo José.
- Quando uma pendência acima for resolvida, atualizar este arquivo e o arquivo de `rules/` correspondente.
