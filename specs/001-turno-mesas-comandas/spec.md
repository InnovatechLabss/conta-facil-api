# Spec 001 — Turno, Mesas e Comandas

- **Status**: Rascunho
- **Regras cobertas**: RN-TUR-01..08, RN-MES-01..07, RN-COM-01..09
- **Depende de**: 008 (Estabelecimento, equipe e permissões) para autenticação e papéis

## Objetivo

Permitir que a equipe abra o turno de operação, ocupe mesas e crie uma comanda individual para cada pessoa, mantendo o controle de quem está em cada mesa até que todas as contas sejam encerradas — com ou sem internet.

## Fora de escopo

- Lançamento de pedidos (spec 003)
- Pagamento e cálculo de total (spec 004)
- Mecânica de sincronização (spec 005) — aqui apenas se exige que tudo funcione offline
- Juntar mesas (RN-MES-07)

---

## Histórias de usuário

### H1 — Abrir turno
**Como** caixa, **quero** abrir o turno informando o fundo de troco, **para** iniciar a operação do dia.

| # | Critério de aceite |
|---|--------------------|
| CA1.1 | **Dado** que não há turno aberto, **quando** o caixa abre o turno com fundo de troco R$ 150,00, **então** o turno fica `ABERTO` com responsável, data/hora e fundo registrados. |
| CA1.2 | **Dado** que já existe turno aberto, **quando** alguém tenta abrir outro, **então** a operação é rejeitada com o erro `TURNO_JA_ABERTO`. |
| CA1.3 | **Dado** que o fundo de troco não é informado, **quando** o turno é aberto, **então** o fundo é registrado como R$ 0,00. |
| CA1.4 | **Dado** que o Hub está sem internet, **quando** o caixa abre o turno, **então** o turno é aberto normalmente e o evento fica pendente de sincronização. |
| CA1.5 | **Dado** que a licença em cache do Hub expirou (RN-OFF-09), **quando** o caixa tenta abrir turno, **então** a operação é rejeitada com `LICENCA_EXPIRADA`. |

### H2 — Abrir mesa com comandas
**Como** garçom, **quero** abrir uma mesa e criar as comandas das pessoas sentadas, **para** começar o atendimento.

| # | Critério de aceite |
|---|--------------------|
| CA2.1 | **Dado** turno aberto e mesa 5 `LIVRE`, **quando** o garçom abre a mesa 5 com as comandas "Ana" e "Bruno", **então** a mesa fica `OCUPADA`, uma sessão de mesa é criada e as duas comandas ficam `ABERTA`. |
| CA2.2 | **Dado** que não há turno aberto, **quando** o garçom tenta abrir uma mesa, **então** é rejeitado com `TURNO_FECHADO`. |
| CA2.3 | **Dado** que a mesa 5 está `OCUPADA`, **quando** outro garçom tenta abri-la, **então** é rejeitado com `MESA_OCUPADA`. |
| CA2.4 | **Dado** turno aberto, **quando** o garçom tenta abrir uma mesa sem nenhuma comanda, **então** é rejeitado com `COMANDA_OBRIGATORIA`. |
| CA2.5 | **Dado** que a mesa está desativada, **quando** o garçom tenta abri-la, **então** é rejeitado com `MESA_INATIVA`. |
| CA2.6 | **Dado** que dois garçons abrem a mesma mesa quase ao mesmo tempo, um deles com o aparelho offline, **quando** os eventos chegam ao Hub, **então** o primeiro aceito vence e o segundo garçom recebe `MESA_OCUPADA` (RN-OFF-06). |

### H3 — Adicionar comanda a uma mesa ocupada
**Como** garçom, **quero** adicionar uma comanda quando chega mais alguém, **para** que essa pessoa tenha sua própria conta.

| # | Critério de aceite |
|---|--------------------|
| CA3.1 | **Dado** mesa 5 `OCUPADA` com "Ana", **quando** o garçom adiciona "Carla", **então** "Carla" é criada `ABERTA` na mesma sessão. |
| CA3.2 | **Dado** mesa 5 com "Ana", **quando** o garçom tenta adicionar outra "Ana", **então** é rejeitado com `NOME_COMANDA_DUPLICADO`. |
| CA3.3 | **Dado** mesa 5 `EM_FECHAMENTO`, **quando** o garçom adiciona uma comanda, **então** a comanda é criada e a mesa volta a `OCUPADA`. |

### H4 — Solicitar fechamento de comanda
**Como** garçom, **quero** marcar que uma pessoa pediu a conta, **para** impedir novos lançamentos nela.

| # | Critério de aceite |
|---|--------------------|
| CA4.1 | **Dado** comanda "Ana" `ABERTA`, **quando** o garçom solicita o fechamento, **então** ela passa a `FECHAMENTO_SOLICITADO`. |
| CA4.2 | **Dado** comanda em `FECHAMENTO_SOLICITADO` sem pagamento registrado, **quando** o garçom a reabre, **então** ela volta a `ABERTA`. |
| CA4.3 | **Dado** comanda em `FECHAMENTO_SOLICITADO` com pagamento parcial registrado, **quando** o garçom tenta reabrir, **então** é rejeitado com `COMANDA_COM_PAGAMENTO`. |
| CA4.4 | **Dado** que a pré-conta consolidada da mesa é solicitada, **então** a mesa passa a `EM_FECHAMENTO`. |

### H5 — Cancelar comanda
**Como** garçom, **quero** cancelar uma comanda criada por engano, **para** manter a mesa correta.

| # | Critério de aceite |
|---|--------------------|
| CA5.1 | **Dado** comanda "Bruno" sem itens ativos, **quando** o garçom a cancela, **então** ela passa a `CANCELADA` com autor e data/hora registrados. |
| CA5.2 | **Dado** comanda "Bruno" com itens ativos, **quando** o garçom tenta cancelar, **então** é rejeitado com `COMANDA_COM_ITENS`. |

### H6 — Transferir comanda para outra mesa
**Como** garçom, **quero** mover a comanda de uma pessoa que trocou de mesa, **para** que a conta a acompanhe.

| # | Critério de aceite |
|---|--------------------|
| CA6.1 | **Dado** "Ana" na mesa 5 e mesa 8 `OCUPADA`, **quando** o garçom transfere "Ana" para a mesa 8, **então** "Ana" e seus itens passam para a sessão da mesa 8 e a transferência é registrada no histórico. |
| CA6.2 | **Dado** mesa 8 `LIVRE`, **quando** o garçom transfere "Ana" para ela, **então** a mesa 8 é aberta (nova sessão) e "Ana" passa a pertencer a ela. |
| CA6.3 | **Dado** que a mesa 8 já tem uma comanda "Ana", **quando** a transferência é feita, **então** é rejeitada com `NOME_COMANDA_DUPLICADO`. |
| CA6.4 | **Dado** que "Ana" era a única comanda não encerrada da mesa 5, **quando** é transferida, **então** a sessão da mesa 5 é encerrada e a mesa volta a `LIVRE`. |
| CA6.5 | **Dado** que "Ana" divide um item com "Bruno" (mesa 5), **quando** "Ana" é transferida, **então** a divisão do item é mantida. |

### H7 — Liberar mesa automaticamente
| # | Critério de aceite |
|---|--------------------|
| CA7.1 | **Dado** mesa 5 com "Ana" `PAGA` e "Bruno" `FECHAMENTO_SOLICITADO`, **quando** "Bruno" é paga, **então** a sessão é encerrada e a mesa volta a `LIVRE`. |
| CA7.2 | **Dado** mesa 5 com "Ana" `PAGA` e "Bruno" `CANCELADA`, **então** a mesa volta a `LIVRE`. |

### H8 — Fechar turno
**Como** caixa, **quero** fechar o turno conferindo o dinheiro, **para** encerrar o caixa do dia.

| # | Critério de aceite |
|---|--------------------|
| CA8.1 | **Dado** turno aberto com comanda(s) `ABERTA` ou `FECHAMENTO_SOLICITADO`, **quando** o caixa tenta fechar, **então** é rejeitado com `COMANDAS_ABERTAS`, listando as mesas pendentes. |
| CA8.2 | **Dado** turno sem comandas abertas, fundo de R$ 150,00 e R$ 500,00 recebidos em dinheiro, **quando** o caixa informa R$ 640,00 contados, **então** o turno é fechado registrando esperado R$ 650,00, contado R$ 640,00 e diferença −R$ 10,00. |
| CA8.3 | **Dado** pagamentos Pix `AGUARDANDO_CONFIRMACAO`, **quando** o turno é fechado, **então** o fechamento é permitido e esses pagamentos aparecem como pendências no relatório. |
| CA8.4 | **Dado** turno fechado, **então** o relatório de fechamento é enviado para impressão (RN-IMP-03). |
| CA8.5 | **Dado** turno aberto às 19h do dia 10 e fechado às 2h do dia 11, **então** o turno pertence ao dia 10. |

### H9 — Visualizar mapa de mesas
**Como** garçom, **quero** ver todas as mesas e seus estados, **para** saber onde atender.

| # | Critério de aceite |
|---|--------------------|
| CA9.1 | **Dado** turno aberto, **quando** o garçom abre o mapa, **então** vê cada mesa ativa com estado (`LIVRE`, `OCUPADA`, `EM_FECHAMENTO`), quantidade de comandas e tempo de ocupação. |
| CA9.2 | **Dado** alteração em uma mesa feita por outro aparelho, **então** o mapa é atualizado em tempo real via Hub. |

---

## Requisitos não funcionais

- **RNF1**: Todas as histórias funcionam sem internet (apenas rede local com o Hub).
- **RNF2**: Toda ação gera evento imutável com autor, aparelho e data/hora (RN-OFF-02).
- **RNF3**: Operações respondem em até 300ms na rede local.

## Erros de domínio

`TURNO_JA_ABERTO`, `TURNO_FECHADO`, `LICENCA_EXPIRADA`, `MESA_OCUPADA`, `MESA_INATIVA`, `COMANDA_OBRIGATORIA`, `NOME_COMANDA_DUPLICADO`, `COMANDA_COM_PAGAMENTO`, `COMANDA_COM_ITENS`, `COMANDAS_ABERTAS`

## Perguntas em aberto

- Garçom pode fechar turno ou apenas caixa/admin? (proposta: apenas caixa/admin)
- Mesa deve ter capacidade (nº de lugares) cadastrada? (proposta: opcional, apenas informativa)
