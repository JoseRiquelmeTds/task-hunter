# Task-Hunter — Descrição do Projeto

Nov 19, 2026 · @Claudio

## Visão geral

O Task-Hunter é um conjunto de skills do Claude Code que encontra, prioriza e executa tasks de backend do Jira IDD no repositório edgebr/dell-ddm, e entrega cada uma como PR em draft. O objetivo é que o meu trabalho vire **revisão**, sobrando mais tempo para estudar o projeto.

- **Contexto:** uso do Claude nas tasks já é recomendado. A mudança aqui é o grau de automação, não o uso em si.
- **Escopo:** tasks de backend do catálogo, não atribuídas e pendentes. A label "backend" nem sempre existe no IDD, então a classificação é feita pelo conteúdo.
- **Princípio:** o agente decide sozinho durante a execução, mas registra cada decisão em disco e nunca entrega nada além de um draft.

## Comandos

Todos os comandos são skills com prefixo `th-`, para não colidir com os comandos nativos `/resume` e `/clear` do Claude Code.

| Comando | O que faz | Escreve em |
| --- | --- | --- |
| `/th-hunt` | Busca tasks no Jira, filtra pelos critérios, julga se cada uma pode ser feita sem VDI e afins, rankeia e pergunta se quero iniciar e quantas | `.task-hunter/ranking.md` |
| `/th-work [n]` | Inicia as N primeiras do ranking, cada uma num subagente com worktree próprio. Atribui a task a mim, muda o status para "Em desenvolvimento", implementa, registra decisões e abre o PR em draft | `.task-hunter/<IDD-xxx>/` + Jira + GitHub |
| `/th-resume [IDD-xxx]` | Para cada task (ou uma específica): o que a task pedia, explicação da solução, o que divergiu do plano, decisões tomadas e pontos de menor confiança | só leitura |
| `/th-clear-all` | Apaga todas as pastas de tasks em `.task-hunter/` (pede confirmação) | `.task-hunter/` |
| `/th-clear <IDD-xxx>` | Apaga a pasta de uma task específica | `.task-hunter/<IDD-xxx>/` |

Os clears mexem só nos arquivos locais. Eles não fecham PR, não removem worktree com trabalho não commitado e não alteram o Jira.

## Arquitetura

Um núcleo coordenador (a sessão principal do Claude Code) dispara um subagente por task. Cada subagente trabalha em um git worktree próprio, e todo o estado fica em disco.

&#91;embedded content: fluxo do Task-Hunter · do Jira até a revisão\]

O núcleo atribui a task no Jira antes de disparar os subagentes. Cada subagente termina em um PR em draft, e nada avança sem a minha revisão.

- **Skills:** cada comando é uma skill em `.claude/skills/<nome>/SKILL.md`, com templates e scripts junto quando precisar.
- **Núcleo:** lê o `ranking.md`, cria os worktrees, dispara os subagentes, acompanha o `status` de cada um e consolida o resultado. Só o núcleo dispara subagentes.
- **Worktrees:** um por task, numa branch própria, para que subagentes em paralelo não sobrescrevam o trabalho um do outro.
- **Estado em disco:** o ranking e os registros sobrevivem entre sessões. Os comandos podem rodar em conversas diferentes. A pasta fica no `.gitignore`.

```
.task-hunter/
  ranking.md          # saída do /th-hunt
  IDD-123/
    task.md           # snapshot do ticket no momento do início
    plan.md           # plano escrito antes de codar
    decisions.md      # decisões registradas na hora, não no fim
    status            # pending | running | blocked | draft-opened
```

O `decisions.md` registra, para cada decisão: o contexto, as opções consideradas, a escolha e o motivo. Também registra cada divergência em relação ao `plan.md`. É a principal fonte do `/th-resume`.

## Integração com o Jira

O acesso é pelo MCP da Atlassian. O filtro fixo vai por JQL, e o julgamento de conteúdo é feito pelo modelo lendo a descrição.

1. **Filtro por JQL:** projeto IDD, sem responsável, status "Tarefas pendentes".
2. **Julgamento pelo modelo:** é backend (independente da label)? Pode ser feita sem VDI ou outra restrição? Algumas tasks dependem de algo que dá para substituir (mock, docker, dado local) e outras não. Essa avaliação acontece no `/th-hunt`, e o motivo de cada exclusão vai para o `ranking.md`.
3. **Ao iniciar uma task (`/th-work`):** atribuir a mim e mudar o status de "Tarefas pendentes" para "Em desenvolvimento". Isso acontece antes de o subagente começar a codar, para ninguém do time pegar a mesma task.

O agente não comenta no ticket nem faz outras transições de status.

## Requisitos mandatórios

Nada sai do agente sem revisão humana. O resto desta seção existe para garantir isso.

- **Revisão humana obrigatória:** o agente só abre PR em **draft**. Quem tira do draft, pede review e faz merge sou eu, depois de revisar. A responsabilidade pelo código continua minha.
- **Paralelismo:** no máximo **2** subagentes ao mesmo tempo, igual ao que já uso hoje com duas sessões manuais. O `/th-work` recusa ou enfileira acima disso.
- **Permissões dos subagentes:** liberar só o necessário. Isso inclui git em branches de feature, rodar testes e `gh pr create --draft`.
- **Bloqueios explícitos** (via `permissions.deny` no settings e hooks):
  - push na `main` ou em branches protegidas
  - `git push --force`
  - comandos contra bancos ou ambientes reais
  - `rm -rf` fora da pasta da própria task
  - transições no Jira além de atribuir e mover para "Em desenvolvimento"
- **Transparência no PR:** a descrição do draft aponta para o resumo e as decisões da task.

## Pendências de definição

Quatro definições precisam estar fechadas antes do `/th-work` rodar sem supervisão. Os rascunhos abaixo são pontos de partida.

- [ ] **Lista de sinais de exclusão**

  Rascunho: menção a VDI; necessidade de credenciais ou acesso ao ambiente do cliente; link "is blocked by" aberto; dependência de outro time ou de contrato de API externo; hardware ou ambiente físico. Para cada sinal, definir se é **substituível** (mock, docker, dado local) ou **impeditivo**.
- [ ] **Rubrica de ranking com pesos**

  Rascunho de critérios: prioridade no Jira, se está na sprint atual, se desbloqueia outras tasks, clareza da descrição, tamanho estimado e idade do ticket. Falta definir os pesos e o critério de desempate.
- [ ] **Condições de parada**

  Rascunho: descrição ambígua demais; escopo maior que o limite definido; precisa mudar schema ou migration; mexe em contrato de API com outro time; testes não rodam localmente. Nesses casos o subagente marca `blocked`, explica o motivo no `decisions.md` e não abre PR.
- [ ] **Convenções do repositório** (pendência)

  Levantar e registrar no `CLAUDE.md`: padrão de nome de branch, mensagem de commit com o ID da task, template de PR e como rodar a suíte de testes.

## Ordem de construção

A ideia é ganhar confiança em cada etapa antes de dar mais autonomia ao agente.

1. **`/th-hunt` só leitura.** Validar se o ranking bate com o que eu escolheria à mão. Ainda não atribui nem muda status.
2. **`/th-work` com 1 task, sem paralelismo.** Ajustar o formato do `plan.md` e do `decisions.md` e validar a atribuição e a mudança de status no Jira.
3. **`/th-resume`, `/th-clear` e `/th-clear-all`.** São simples e dependem do formato já estabilizado.
4. **Paralelismo com worktrees.** Até 2 subagentes coordenados pelo núcleo.
