# Spec 003 — Pedidos e Impressão

- **Status**: Rascunho
- **Regras cobertas**: RN-PED-01..18, RN-IMP-01..04, RN-IMP-06..08, RN-IMP-10..11
- **Epic Jira**: CF-7
- **Depende de**: 001 (turno, mesas e comandas), 002 (cardápio), 005 (Hub Local), 008 (papéis)

## Objetivo

Permitir que a equipe lance pedidos para comandas individuais ou compartilhadas, com divisão igual ou personalizada, e que os pedidos cheguem à produção na hora, por impressora térmica, sem depender de internet. É aqui que nasce a conta separada de cada pessoa.

## Fora de escopo

- Painel de produção (KDS): spec 006. Aqui só se define que o pedido é enviado ao KDS quando houver.
- Pré-conta, pagamento e comprovante: spec 004
- "Segurar" itens para enviar depois (D-08)
- Adicionais e variações com preço (RN-CAR-06)

---

## Histórias de usuário

### H1 — Lançar pedido para comandas individuais `CF-31`
**Como** garçom, **quero** lançar vários itens de uma vez, cada um na comanda de quem pediu, **para** atender a mesa rápido sem misturar as contas.

| # | Critério de aceite |
|---|--------------------|
| CA1.1 | **Dado** mesa 5 com comandas "Ana" e "Bruno" `ABERTA`, **quando** o garçom lança 2 "Chopp 300ml" para Ana e 1 "Picanha" para Bruno, **então** é criado um pedido com as duas linhas, cada uma na sua comanda, com status `PENDENTE`. |
| CA1.2 | **Dado** "Chopp 300ml" a R$ 12,90, **quando** o garçom lança 2 para Ana, **então** a linha tem preço unitário 1290 congelado e total 2580. |
| CA1.3 | **Dado** um item com observação "sem cebola", **quando** é lançado, **então** a observação fica gravada no item e aparece no ticket de produção. |
| CA1.4 | **Dado** "Refrigerante lata" com destino `NENHUM`, **quando** é lançado, **então** nasce `ENTREGUE` e não gera ticket de produção. |
| CA1.5 | **Dado** quantidade 0 ou negativa, **quando** o garçom lança, **então** é rejeitado com `QUANTIDADE_INVALIDA`. |
| CA1.6 | **Dado** "Picanha" indisponível, **quando** o garçom tenta lançá-la, **então** é rejeitado com `PRODUTO_INDISPONIVEL`. |
| CA1.7 | **Dado** comanda "Ana" em `FECHAMENTO_SOLICITADO`, **quando** o garçom tenta lançar nela, **então** é rejeitado com `COMANDA_NAO_ABERTA`. |
| CA1.8 | **Dado** que não há turno aberto, **quando** alguém tenta lançar, **então** é rejeitado com `TURNO_FECHADO`. |
| CA1.9 | **Dado** um usuário com papel Cozinha/Bar, **quando** ele tenta lançar pedido, **então** é rejeitado com `SEM_PERMISSAO`. |

### H2 — Lançar item compartilhado `CF-32`
**Como** garçom, **quero** lançar um item dividido entre várias pessoas, em partes iguais ou não, **para** que cada um pague exatamente a sua parte.

| # | Critério de aceite |
|---|--------------------|
| CA2.1 | **Dado** "Porção de batata" a R$ 10,00, **quando** o garçom a divide igualmente entre Ana, Bruno e Carla (nessa ordem), **então** as cotas são Ana R$ 3,34, Bruno R$ 3,33 e Carla R$ 3,33. |
| CA2.2 | **Dado** "Vinho" a R$ 100,00, **quando** o garçom divide com Ana 2 partes e Bruno 1 parte, **então** as cotas são Ana R$ 66,67 e Bruno R$ 33,33. |
| CA2.3 | **Dado** "Tábua de frios" a R$ 45,90, **quando** o garçom divide com Ana 1, Bruno 1 e Carla 2 partes (nessa ordem), **então** as cotas são Ana R$ 11,48, Bruno R$ 11,47 e Carla R$ 22,95. |
| CA2.4 | **Dado** qualquer divisão, **então** a soma das cotas é exatamente igual ao total do item. |
| CA2.5 | **Dado** 2 "Chopp" compartilhados entre Ana e Bruno, **então** o total dividido é o da linha (2 × preço unitário). |
| CA2.6 | **Dado** uma divisão com só 1 comanda, partes iguais a 0 ou comandas de outra mesa, **quando** o garçom lança, **então** é rejeitado com `DIVISAO_INVALIDA`. |
| CA2.7 | **Dado** que "Bruno" está em `FECHAMENTO_SOLICITADO`, **quando** o garçom tenta incluí-lo numa divisão, **então** é rejeitado com `COMANDA_NAO_ABERTA`. |

### H3 — Transferir item e alterar divisão `CF-33`
**Como** garçom, **quero** corrigir de quem é um item ou como ele é dividido, **para** resolver o "essa cerveja era minha".

| # | Critério de aceite |
|---|--------------------|
| CA3.1 | **Dado** um "Chopp" na comanda de Ana, **quando** o garçom o transfere para Bruno, **então** o item passa para Bruno com o mesmo preço congelado, e a transferência fica no histórico com autor e data/hora. |
| CA3.2 | **Dado** 1 de 3 "Chopp" de Ana, **quando** o garçom transfere só 1 unidade para Bruno, **então** Ana fica com 2 e Bruno com 1. |
| CA3.3 | **Dado** "Vinho" dividido Ana 1 e Bruno 1, **quando** o garçom altera para Ana 2 e Bruno 1, **então** as cotas são recalculadas pela regra de divisão. |
| CA3.4 | **Dado** um item individual de Ana, **quando** o garçom o transforma em compartilhado com Bruno, **então** passa a ser dividido conforme as partes informadas. |
| CA3.5 | **Dado** que alguma comanda envolvida não está `ABERTA`, **quando** o garçom tenta transferir ou redividir, **então** é rejeitado com `COMANDA_NAO_ABERTA`. |
| CA3.6 | **Dado** uma comanda de outra mesa, **quando** o garçom tenta transferir um item para ela, **então** é rejeitado com `DIVISAO_INVALIDA` (para mudar a pessoa de mesa, usa-se a transferência de comanda, spec 001). |

### H4 — Cancelar item `CF-34`
**Como** garçom, caixa ou admin, **quero** cancelar itens lançados, **para** corrigir erros e registrar perdas.

| # | Critério de aceite |
|---|--------------------|
| CA4.1 | **Dado** um item `PENDENTE`, **quando** o garçom o cancela, **então** ele passa a `CANCELADO` e sai do total da comanda. |
| CA4.2 | **Dado** um item `EM_PREPARO`, **quando** o garçom tenta cancelá-lo, **então** é rejeitado com `SEM_PERMISSAO`. |
| CA4.3 | **Dado** um item `EM_PREPARO`, **quando** o caixa o cancela sem motivo, **então** é rejeitado com `MOTIVO_OBRIGATORIO`. |
| CA4.4 | **Dado** uma "Picanha" `PRONTO` que voltou da mesa, **quando** o caixa a cancela com motivo "ponto errado" e classificação `PERDA`, **então** o item é cancelado e entra como perda no relatório do turno, pelo preço de venda. |
| CA4.5 | **Dado** um item `EM_PREPARO` lançado por engano e ainda não produzido, **quando** o caixa o cancela como `SEM_PERDA`, **então** ele não entra no relatório de perdas. |
| CA4.6 | **Dado** 3 "Chopp" na mesma linha, **quando** o garçom cancela 1 enquanto `PENDENTE`, **então** a linha fica com 2 e o cancelamento de 1 unidade fica registrado. |
| CA4.7 | **Dado** um item já enviado à produção, **quando** ele é cancelado, **então** é impresso um ticket de **CANCELAMENTO** no destino (e/ou o aviso aparece no KDS). |
| CA4.8 | **Dado** um item compartilhado, **quando** ele é cancelado, **então** a cota de todas as comandas envolvidas é removida. |
| CA4.9 | **Dado** um item já `CANCELADO`, **quando** alguém tenta cancelá-lo de novo, **então** é rejeitado com `ITEM_JA_CANCELADO`. |

### H5 — Acompanhar status e marcar entrega `CF-35`
**Como** garçom, **quero** saber o que já está sendo preparado e marcar o que entreguei, **para** controlar o atendimento.

| # | Critério de aceite |
|---|--------------------|
| CA5.1 | **Dado** um estabelecimento sem KDS, **quando** o ticket de produção do item é impresso com sucesso, **então** o item passa a `EM_PREPARO`. |
| CA5.2 | **Dado** um item `EM_PREPARO` ou `PRONTO`, **quando** o garçom o marca como entregue, **então** ele passa a `ENTREGUE`. |
| CA5.3 | **Dado** a mesa 5, **quando** o garçom abre o detalhe da mesa, **então** vê cada item, de quem é (ou "compartilhado"), quantidade e status. |
| CA5.4 | **Dado** que a impressão do ticket falhou, **então** o item continua `PENDENTE` e a mesa mostra um alerta de item não enviado à produção. |

### H6 — Lançar pedido sem conexão `CF-36`
**Como** garçom, **quero** continuar lançando pedidos mesmo com a rede instável, **para** não parar o atendimento.

| # | Critério de aceite |
|---|--------------------|
| CA6.1 | **Dado** que o Hub está sem internet, **quando** o garçom lança um pedido, **então** o pedido é registrado e o ticket sai normalmente na impressora local. |
| CA6.2 | **Dado** que o aparelho do garçom perdeu a conexão com o Hub (Wi-Fi), **quando** ele lança um pedido, **então** o pedido fica na fila do aparelho, marcado como "pendente de envio", e é enviado quando a conexão voltar. |
| CA6.3 | **Dado** um pedido na fila do aparelho, **então** nenhum ticket é impresso até o pedido chegar ao Hub. |
| CA6.4 | **Dado** um pedido na fila para a comanda "Ana", que outro garçom colocou em `FECHAMENTO_SOLICITADO`, **quando** o pedido chega ao Hub, **então** é rejeitado com `COMANDA_NAO_ABERTA` e o garçom é avisado (RN-OFF-06). |
| CA6.5 | **Dado** um pedido na fila com "Picanha", que a cozinha marcou indisponível nesse meio-tempo, **quando** o pedido chega ao Hub, **então** a linha da Picanha é rejeitada com `PRODUTO_INDISPONIVEL`, as demais são aceitas e o garçom é avisado. |
| CA6.6 | **Dado** que o aparelho reenvia o mesmo pedido após uma falha de rede, **então** o Hub não o duplica (RN-OFF-03). |

### H7 — Configurar impressoras e destinos `CF-37`
**Como** admin, **quero** cadastrar as impressoras e definir para onde vai cada destino, **para** que os pedidos saiam no lugar certo.

| # | Critério de aceite |
|---|--------------------|
| CA7.1 | **Dado** uma impressora de rede em `192.168.0.50`, 80 mm, **quando** o admin a cadastra como "Cozinha", **então** ela fica disponível para mapeamento. |
| CA7.2 | **Dado** uma impressora cadastrada, **quando** o admin pede uma impressão de teste, **então** sai uma página de teste com nome da impressora, largura e data/hora. |
| CA7.3 | **Dado** as impressoras "Cozinha" e "Balcão", **quando** o admin mapeia `COZINHA` → "Cozinha" e `BAR` → "Balcão", **então** os tickets passam a sair nessas impressoras. |
| CA7.4 | **Dado** uma única impressora, **quando** o admin a mapeia para `COZINHA` e `BAR`, **então** ela recebe um ticket separado para cada destino. |
| CA7.5 | **Dado** o destino `BAR` sem impressora e sem KDS, **quando** um item de bar é lançado, **então** o lançamento é aceito, o item fica `PENDENTE` e o caixa e o admin veem um alerta (RN-PED-18). |
| CA7.6 | **Dado** um usuário que não é Admin, **quando** ele tenta cadastrar impressora ou alterar o mapeamento, **então** é rejeitado com `SEM_PERMISSAO`. |

### H8 — Imprimir ticket de produção `CF-38`
**Como** cozinha ou bar, **quero** receber um ticket claro de cada pedido, **para** preparar o que foi pedido e saber para quem é.

| # | Critério de aceite |
|---|--------------------|
| CA8.1 | **Dado** um pedido com itens de cozinha e de bar, **quando** é lançado, **então** sai um ticket na impressora da cozinha e outro na do bar, cada um só com seus itens. |
| CA8.2 | **Dado** um ticket de produção, **então** ele contém destino, mesa, garçom, data/hora, número do pedido e, por item, quantidade, produto, observação e nome da comanda (ou "compartilhado"). |
| CA8.3 | **Dado** que a impressora está desligada, **quando** um ticket é enviado, **então** ele entra na fila com novas tentativas automáticas, e o caixa vê um alerta. O lançamento não é bloqueado. |
| CA8.4 | **Dado** que a impressora volta a funcionar, **então** os tickets da fila são impressos na ordem em que foram lançados. |
| CA8.5 | **Dado** um ticket já impresso, **quando** o garçom ou o caixa pede reimpressão, **então** ele sai marcado como "2ª VIA". |
| CA8.6 | **Dado** qualquer ticket, **então** ele contém "NÃO É DOCUMENTO FISCAL". |

---

## Requisitos não funcionais

- **RNF1**: Do lançamento à impressão do ticket, no máximo 3 s na rede local.
- **RNF2**: Funciona integralmente sem internet (RN-IMP-02).
- **RNF3**: Toda ação (lançar, transferir, redividir, cancelar, entregar) gera evento imutável com autor, aparelho e data/hora.
- **RNF4**: A regra de divisão (RN-PED-03) tem testes de propriedade: para qualquer total e quaisquer partes, a soma das cotas é igual ao total e nenhuma cota difere do valor exato em mais de 1 centavo.

## Erros de domínio

`SEM_PERMISSAO`, `TURNO_FECHADO`, `COMANDA_NAO_ABERTA`, `PRODUTO_INDISPONIVEL`, `QUANTIDADE_INVALIDA`, `DIVISAO_INVALIDA`, `MOTIVO_OBRIGATORIO`, `ITEM_JA_CANCELADO`

## Decisões registradas

- Divisão de item compartilhado pode ser igual ou personalizada, por partes inteiras (RN-PED-03).
- Todo item vai à produção na hora; "segurar" pedido fica fora do MVP por risco de erro do usuário (RN-PED-17, D-08).
- Cancelamento após o preparo exige motivo e classificação `PERDA`/`SEM_PERDA`; perdas entram no relatório do turno (RN-PED-14).

## Perguntas em aberto

1. **Divisão por valor:** além de partes (2/3 e 1/3), permitir dividir por valor em reais (ex.: Ana paga R$ 70 do vinho e Bruno o resto)? (proposta: não no MVP; partes cobrem a maioria dos casos)
2. **Quem marca entregue:** além do garçom, o caixa pode marcar itens como entregues? (proposta: sim, Garçom, Caixa e Admin)
