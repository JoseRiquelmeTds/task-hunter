---
name: th-hunt
description: Busca tasks de backend não atribuídas no Jira IDD, filtra as que exigem VDI ou outras restrições, rankeia e grava em .task-hunter/ranking.md. Só leitura no Jira.
disable-model-invocation: true
---

# /th-hunt

Encontra as tasks do IDD que o Task-Hunter pode executar e gera um ranking. **Etapa 1: só leitura.** Esta skill não atribui, não muda status e não comenta no Jira. O único arquivo que ela escreve é `.task-hunter/ranking.md`.

## Referências

- Raiz do Task-Hunter: `C:/dev/task-hunter` (enquanto o método de instalação não for decidido, os caminhos abaixo são relativos a ela).
- Regras: `rules/exclusion-signals.md` e `rules/ranking-rubric.md`.
- Template: `templates/ranking.md`.
- Jira: site `edgebr.atlassian.net` (use como `cloudId`), projeto `IDD`.
- Status "Tarefas pendentes" = **id `10023`**. Na JQL, filtre pelo id: o nome traduzido não é aceito pela JQL.

## Passos

1. **Conferir o repositório.** Rode `git remote -v` no diretório atual. Se não aparecer `edgebr/dell-ddm`, pare e avise que o `/th-hunt` deve rodar dentro do dell-ddm.
   - Se `.task-hunter/` não estiver no `.gitignore` do dell-ddm, **avise** o usuário, mas não edite o `.gitignore`.

2. **Ler as regras.** Leia `rules/exclusion-signals.md`, `rules/ranking-rubric.md` e `templates/ranking.md` inteiros antes de consultar o Jira. As notas e exclusões seguem esses arquivos, não critérios próprios.

3. **Buscar as candidatas.** Use `searchJiraIssuesUsingJql` do MCP da Atlassian:
   - JQL: `project = IDD AND assignee IS EMPTY AND status = 10023 ORDER BY created ASC`
   - `view: "evidence"` (traz descrição e issue links), `maxResults: 50`.
   - Pagine com `nextPageToken` até acabar. Anote o total de tasks retornadas.

4. **Buscar quais estão na sprint ativa.** Rode a mesma JQL com `AND sprint in openSprints()` e guarde as chaves. Todas as outras ficam com nota 0 no critério "Sprint atual".

5. **Descartar o que não é task executável.** Tipo `Épico` vai direto para "Não são backend", com o motivo "épico, não é executável".

6. **Classificar backend.** Para cada task restante, julgue pelo **conteúdo** (resumo, descrição, épico pai), não pela label, se o trabalho é de backend. Exemplos de backend: API, serviço, integração, banco, testes de backend, autenticação no servidor. Exemplos de não backend: tela, componente de frontend, documentação pura, estudo ou levantamento sem código. As que não forem vão para "Não são backend", com uma frase de motivo.

7. **Aplicar os sinais de exclusão.** Para cada task de backend, verifique cada sinal de `rules/exclusion-signals.md`:
   - Leia descrição, labels e issue links. Se a task tiver comentários, leia-os com `listJiraIssueComments` (ou `getJiraIssue`) antes de decidir.
   - Sinal **impeditivo** → "Excluídas", com o sinal e o trecho ou fato que o disparou.
   - Sinal **substituível** → continua candidata e o contorno vai na coluna "Restrições substituíveis".
   - Na dúvida entre substituível e impeditivo, trate como impeditivo e diga no detalhe que a dúvida existiu.

8. **Pontuar as candidatas.** Calcule a nota de cada critério com as escalas de `rules/ranking-rubric.md` e a nota final (soma de `nota × peso`). Ordene pela nota final, decrescente, e desempate pela data de criação mais antiga.

9. **Gravar o ranking.** Crie `.task-hunter/` se não existir e escreva `.task-hunter/ranking.md` seguindo `templates/ranking.md`:
   - Na coluna "Motivo da posição", mostre a nota de cada critério (ex.: `sprint 0 · prio 3 · desbl 3 · clar 5 · tam 3 · idade 4`).
   - Toda task retornada pela JQL aparece em exatamente uma das três seções. Confira: candidatas + excluídas + não backend = total do passo 3.
   - Se já existir um `ranking.md`, sobrescreva. Ele é sempre o retrato da última busca.

10. **Responder ao usuário.** Mostre:
    - o total encontrado e quantas ficaram em cada seção;
    - as 5 primeiras candidatas, com chave, título, nota e uma linha de motivo;
    - os sinais de exclusão que mais apareceram;
    - o caminho do `ranking.md`.

    Na etapa 1, termine dizendo que o `/th-work` ainda não está disponível. **Não** pergunte se quer iniciar tasks.

## Proibido nesta skill

- Qualquer escrita no Jira: atribuir, transicionar, comentar, editar campos ou links.
- Escrever qualquer arquivo além de `.task-hunter/ranking.md`.
- Criar branch, worktree ou subagente.
