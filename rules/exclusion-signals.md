# Sinais de exclusão

> **Status: definido** (19/11/2026).

Usado pelo `/th-hunt` para decidir se uma task pode ser feita sem VDI ou outras restrições.

- **Substituível:** dá para contornar localmente. A task continua candidata, e o contorno vai na coluna "Restrições substituíveis" do ranking.
- **Impeditivo:** a task vai para "Excluídas", com o motivo.

| Sinal | Como reconhecer | Classificação |
| --- | --- | --- |
| Menção a VDI | "VDI", "máquina virtual do cliente" na descrição ou nos comentários | impeditivo |
| Credenciais ou acesso ao ambiente do cliente | pede acesso, senha, VPN ou ambiente do cliente | impeditivo |
| Bloqueio declarado | link "is blocked by" com a outra task não concluída, ou label `blocked-by-*` (ex.: `blocked-by-okta`) | impeditivo |
| Dependência de outro time ou de contrato de API externo | menciona outro time ou uma API que não é nossa | impeditivo |
| Hardware ou ambiente físico | dispositivo, servidor físico, laboratório | impeditivo |
| Banco ou serviço que roda local | depende de banco, fila ou serviço que sobe com docker | **substituível** |

## Uso do docker

O Docker fica rodando na máquina do José, e o agente pode usá-lo livremente para subir dependências **locais** (banco, fila, cache) e rodar testes contra elas.

O bloqueio de "comandos contra bancos ou ambientes reais" continua valendo: o docker serve para substituir o ambiente real, nunca para se conectar a ele.
