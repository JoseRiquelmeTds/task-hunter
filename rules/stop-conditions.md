# Condições de parada

> **Status: pendente.** Rascunho tirado da especificação.

Usado pelo subagente do `/th-work`. Se qualquer condição abaixo for atingida, o subagente:

1. grava `blocked` no arquivo `status` da task;
2. registra uma entrada do tipo `BLOQUEIO` no `decisions.md`, com o motivo;
3. **não** abre PR.

| Condição | Como reconhecer |
| --- | --- |
| Descrição ambígua demais | não dá para escrever critérios de aceite no `plan.md` sem inventar requisito |
| Escopo acima do limite | limite a definir (ex.: número de arquivos ou linhas alteradas) |
| Mudança de schema ou migration | o plano exige alterar schema, migration ou dados persistidos |
| Contrato de API com outro time | muda request ou response de uma API consumida por outro time |
| Testes não rodam localmente | a suíte não sobe ou depende de ambiente real |
