# Rubrica de ranking

> **Status: definido** (19/11/2026). Os pesos e o desempate foram definidos pelo José. As escalas de 0 a 5 são uma proposta, para validar no teste do `/th-hunt` (etapa 1).

Cada critério dá uma nota de 0 a 5. A nota final é a soma de `nota × peso` (máximo 105).

| Critério | Peso | Escala (0 a 5) |
| --- | --- | --- |
| Sprint atual | 4 | na sprint ativa = 5 · fora = 0 |
| Prioridade no Jira | 4 | Highest = 5 · High = 4 · Medium = 3 · Low = 2 · Lowest = 1 |
| Desbloqueia outras tasks | 4 | links "blocks" com task aberta: 0 → 0 · 1 → 3 · 2 → 4 · 3 ou mais → 5 |
| Clareza da descrição | 3 | 5 = critérios de aceite explícitos e escopo fechado · 3 = dá para inferir o que fazer · 1 = vaga |
| Tamanho estimado | 2 | Story Points (customfield_10028), se houver: até 2 = 5 · 3 a 5 = 3 · 8 ou mais = 1. Sem Story Points, estimativa do modelo: pequena = 5 · média = 3 · grande = 1 |
| Idade do ticket | 4 | criada há: mais de 12 meses = 5 · 6 a 12 = 4 · 3 a 6 = 3 · 1 a 3 = 2 · menos de 1 = 1 |

## Desempate

Em caso de nota final igual, fica à frente a task **mais antiga** (data de criação no Jira).

## No ranking

A coluna "Motivo da posição" do `ranking.md` mostra a nota de cada critério, para dar para conferir a conta.
