# Sinais de exclusão

> **Status: pendente.** Rascunho tirado da especificação. Falta confirmar cada sinal e a classificação.

Usado pelo `/th-hunt` para decidir se uma task pode ser feita sem VDI ou outras restrições.

- **Substituível:** dá para contornar localmente (mock, docker, dado local). A task continua candidata, e o contorno vai na coluna "Restrições substituíveis" do ranking.
- **Impeditivo:** a task vai para "Excluídas", com o motivo.

| Sinal | Como reconhecer | Classificação |
| --- | --- | --- |
| Menção a VDI | "VDI", "máquina virtual do cliente" na descrição ou nos comentários | impeditivo (a confirmar) |
| Credenciais ou acesso ao ambiente do cliente | pede acesso, senha, VPN ou ambiente do cliente | impeditivo (a confirmar) |
| Link "is blocked by" aberto | issue link do tipo bloqueio com a outra task não concluída | impeditivo (a confirmar) |
| Dependência de outro time ou de contrato de API externo | menciona outro time ou uma API que não é nossa | a definir |
| Hardware ou ambiente físico | dispositivo, servidor físico, laboratório | impeditivo (a confirmar) |
| Banco ou serviço que roda local | depende de banco, fila ou serviço com imagem docker | substituível (a confirmar) |
