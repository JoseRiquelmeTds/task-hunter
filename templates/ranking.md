# Ranking do Task-Hunter

Gerado em: {{data}} · JQL: `project = IDD AND assignee IS EMPTY AND status = 10023 ORDER BY created ASC` (10023 = "Tarefas pendentes")
Total retornado: {{n}} · Candidatas: {{n}} · Excluídas: {{n}} · Não backend: {{n}}
Rubrica: `rules/ranking-rubric.md` · Sinais de exclusão: `rules/exclusion-signals.md`

## Candidatas

| # | Task | Título | Nota | Restrições substituíveis | Motivo da posição |
| --- | --- | --- | --- | --- | --- |
| 1 | {{IDD-xxx}} | {{título}} | {{nota}} | {{ex.: banco via docker}} | {{resumo dos critérios}} |

## Excluídas

| Task | Título | Sinal de exclusão | Detalhe |
| --- | --- | --- | --- |
| {{IDD-xxx}} | {{título}} | {{ex.: VDI (impeditivo)}} | {{trecho ou motivo}} |

## Não são backend

| Task | Título | Por quê |
| --- | --- | --- |
| {{IDD-xxx}} | {{título}} | {{motivo}} |
