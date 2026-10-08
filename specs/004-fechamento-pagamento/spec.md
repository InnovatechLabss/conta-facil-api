# Spec 004 — Fechamento, Pagamento e Pix

- **Status**: Rascunho
- **Regras cobertas**: RN-PAG-01..23, RN-IMP-03, RN-IMP-05, RN-TUR-05..06
- **Epic Jira**: CF-8
- **Depende de**: 001 (comandas), 003 (itens e cotas), 005 (Hub Local), 007 (módulo Pix), 008 (papéis)

## Objetivo

Permitir que cada pessoa feche e pague a própria comanda, de forma independente, com dinheiro, cartão ou Pix, inclusive sem internet, e que o caixa feche o turno com tudo conferido: descontos, taxas, estornos e pendências.

## Fora de escopo

- Emissão de NFC-e (D-01)
- Integração com maquininhas de cartão (D-05); o cartão é só registrado
- Escolha do provedor Pix concreto (D-02); aqui se usa a interface `PaymentProvider`
- Divisão por valor em reais (D-09)

## Exemplos usados nesta spec

Mesa 5, taxa de serviço 10%:
- **Ana**: 2 Chopp (R$ 25,80) + 1/3 da Porção de batata (R$ 3,34) = subtotal **R$ 29,14**; taxa R$ 2,91; total **R$ 32,05**.
- **Bruno**: Picanha (R$ 89,90) + 1/3 da Porção de batata (R$ 3,33) = subtotal **R$ 93,23**; taxa R$ 9,32; total **R$ 102,55**.

---

## Histórias de usuário

### H1 — Ver o total da comanda `CF-39`
**Como** garçom ou caixa, **quero** ver o total de cada comanda com tudo discriminado, **para** cobrar o valor certo de cada pessoa.

| # | Critério de aceite |
|---|--------------------|
| CA1.1 | **Dado** a comanda de Ana, **quando** o garçom abre o fechamento, **então** vê subtotal R$ 29,14, taxa R$ 2,91, total R$ 32,05, já pago R$ 0,00 e saldo R$ 32,05. |
| CA1.2 | **Dado** subtotal R$ 10,05 e taxa de 10%, **então** a taxa é R$ 1,01 (R$ 1,005 arredondado com meio centavo para cima). |
| CA1.3 | **Dado** que a taxa era 10% quando a comanda de Ana foi aberta, **quando** o admin muda a taxa para 12%, **então** a comanda de Ana continua com 10% e só novas comandas usam 12%. |
| CA1.4 | **Dado** itens cancelados na comanda, **então** eles não entram no subtotal. |
| CA1.5 | **Dado** uma comanda com item compartilhado, **então** o fechamento mostra a cota como "1/3 Porção de batata — R$ 3,34". |

### H2 — Remover ou restaurar a taxa de serviço `CF-40`
**Como** garçom, **quero** retirar a taxa quando o cliente não quiser pagá-la, **para** respeitar o direito dele.

| # | Critério de aceite |
|---|--------------------|
| CA2.1 | **Dado** a comanda de Ana com taxa, **quando** o garçom remove a taxa, **então** o total passa a R$ 29,14 e fica registrado quem removeu e quando. |
| CA2.2 | **Dado** a taxa removida, **quando** o garçom a restaura, **então** o total volta a R$ 32,05. |
| CA2.3 | **Dado** uma comanda `PAGA`, **quando** alguém tenta remover a taxa, **então** é rejeitado com `COMANDA_PAGA`. |
| CA2.4 | **Dado** a comanda de Ana com R$ 30,00 já pagos, **quando** o garçom tenta remover a taxa (o total cairia para R$ 29,14), **então** é rejeitado com `TOTAL_MENOR_QUE_PAGO`. |

### H3 — Aplicar desconto `CF-41`
**Como** caixa, **quero** dar desconto em uma comanda, **para** resolver cortesias e reclamações com registro.

| # | Critério de aceite |
|---|--------------------|
| CA3.1 | **Dado** a comanda de Ana, **quando** o caixa aplica 10% de desconto com motivo "cliente frequente", **então** o desconto é R$ 2,91, a taxa passa a R$ 2,62 e o total a R$ 28,85. |
| CA3.2 | **Dado** a comanda de Ana, **quando** o caixa aplica R$ 5,00 de desconto, **então** a taxa passa a R$ 2,41 e o total a R$ 26,55. |
| CA3.3 | **Dado** um desconto sem motivo, **quando** o caixa aplica, **então** é rejeitado com `MOTIVO_OBRIGATORIO`. |
| CA3.4 | **Dado** um desconto de R$ 50,00 na comanda de Ana (subtotal R$ 29,14), **quando** o caixa aplica, **então** é rejeitado com `DESCONTO_INVALIDO`. |
| CA3.5 | **Dado** a comanda de Ana com R$ 30,00 já pagos, **quando** o caixa aplica um desconto que deixaria o total abaixo de R$ 30,00, **então** é rejeitado com `TOTAL_MENOR_QUE_PAGO`. |
| CA3.6 | **Dado** um desconto de 10% já aplicado, **quando** o caixa aplica R$ 5,00, **então** o novo desconto substitui o anterior. |
| CA3.7 | **Dado** um usuário com papel Garçom, **quando** ele tenta aplicar desconto, **então** é rejeitado com `SEM_PERMISSAO`. |
| CA3.8 | **Dado** descontos aplicados no turno, **então** o relatório de fechamento lista cada desconto com comanda, valor, motivo e autor. |

### H4 — Imprimir pré-conta `CF-42`
**Como** garçom, **quero** imprimir a conta de uma pessoa ou da mesa inteira, **para** o cliente conferir antes de pagar.

| # | Critério de aceite |
|---|--------------------|
| CA4.1 | **Dado** a comanda de Ana, **quando** o garçom imprime a pré-conta individual, **então** ela lista itens, cotas, subtotal, desconto, taxa (com o aviso "taxa de serviço opcional"), total, já pago e saldo, e a comanda passa a `FECHAMENTO_SOLICITADO`. |
| CA4.2 | **Dado** a mesa 5, **quando** o garçom imprime a pré-conta da mesa, **então** ela mostra o subtotal e o total de cada pessoa e o total da mesa, e a mesa passa a `EM_FECHAMENTO`. |
| CA4.3 | **Dado** um estabelecimento com o módulo Pix, **quando** a pré-conta individual é impressa, **então** traz o QR Code Pix do saldo: dinâmico se o Hub estiver online, estático se offline. |
| CA4.4 | **Dado** um estabelecimento sem o módulo Pix, **então** a pré-conta não traz QR Code. |
| CA4.5 | **Dado** uma pré-conta já impressa, **quando** é reimpressa, **então** sai como "2ª VIA" e com valores atualizados. |
| CA4.6 | **Dado** qualquer pré-conta, **então** ela contém "NÃO É DOCUMENTO FISCAL". |

### H5 — Registrar pagamento em dinheiro ou cartão `CF-43`
**Como** caixa ou garçom, **quero** registrar como a pessoa pagou, **para** quitar a comanda e fechar o caixa certo.

| # | Critério de aceite |
|---|--------------------|
| CA5.1 | **Dado** saldo de R$ 32,05, **quando** o caixa registra dinheiro recebido R$ 50,00, **então** o valor aplicado é R$ 32,05, o troco R$ 17,95 e a comanda passa a `PAGA`. |
| CA5.2 | **Dado** saldo de R$ 32,05, **quando** o garçom registra cartão de débito de R$ 32,05, **então** a comanda passa a `PAGA`. |
| CA5.3 | **Dado** saldo de R$ 32,05, **quando** alguém registra cartão de R$ 40,00, **então** é rejeitado com `VALOR_EXCEDE_SALDO`. |
| CA5.4 | **Dado** saldo de R$ 32,05, **quando** são registrados cartão de R$ 20,00 e depois dinheiro recebido R$ 20,00, **então** o dinheiro aplicado é R$ 12,05, o troco R$ 7,95 e a comanda passa a `PAGA`. |
| CA5.5 | **Dado** uma comanda `ABERTA`, **quando** o primeiro pagamento parcial é registrado, **então** ela passa a `FECHAMENTO_SOLICITADO` e não aceita novos itens. |
| CA5.6 | **Dado** um usuário com papel Garçom, **quando** ele tenta registrar pagamento em dinheiro, **então** é rejeitado com `SEM_PERMISSAO`. |
| CA5.7 | **Dado** um estabelecimento sem o módulo Pix, **quando** o garçom registra "Pix" de R$ 32,05, **então** o pagamento é registrado manualmente, sem QR Code e sem conciliação. |
| CA5.8 | **Dado** que o Hub está sem internet, **quando** pagamentos em dinheiro ou cartão são registrados, **então** funcionam normalmente. |
| CA5.9 | **Dado** Ana e Bruno pagos e Carla ainda `ABERTA` na mesa 5, **então** a mesa continua `OCUPADA` até Carla pagar (RN-COM-09). |
| CA5.10 | **Dado** um pagamento registrado, **quando** alguém pede o comprovante, **então** ele é impresso com forma, valor aplicado, troco e saldo restante. |

### H6 — Pagar com Pix (online) `CF-44`
**Como** cliente, **quero** pagar com Pix lendo um QR Code, **para** pagar rápido sem depender da maquininha.

| # | Critério de aceite |
|---|--------------------|
| CA6.1 | **Dado** o módulo Pix e o Hub online, **quando** o garçom gera o Pix do saldo de Ana (R$ 32,05), **então** é criada uma cobrança dinâmica com `txid`, e o QR Code aparece no aparelho e pode ser impresso. |
| CA6.2 | **Dado** a cobrança gerada, **quando** o provedor confirma o pagamento (webhook), **então** o pagamento é registrado como confirmado, a comanda passa a `PAGA` e o aparelho do garçom é avisado. |
| CA6.3 | **Dado** uma cobrança pendente há mais de 15 minutos, **então** ela expira, não conta como pagamento e o garçom pode gerar outra. |
| CA6.4 | **Dado** que o webhook de confirmação chega duas vezes, **então** o pagamento é registrado uma vez só. |
| CA6.5 | **Dado** uma cobrança gerada para R$ 32,05, **quando** o cliente paga um valor menor, **então** o pagamento é registrado pelo valor efetivamente recebido e o saldo restante continua em aberto. |
| CA6.6 | **Dado** uma cobrança de R$ 32,05, **quando** o cliente paga R$ 35,00, **então** é aplicado R$ 32,05, a comanda passa a `PAGA` e os R$ 2,95 excedentes viram divergência para o admin resolver, sem crédito automático (RN-PAG-23). |

### H7 — Pagar com Pix sem internet e conciliar depois `CF-45`
**Como** caixa, **quero** receber Pix mesmo com a internet fora do ar, **para** o cliente não ficar preso na mesa.

| # | Critério de aceite |
|---|--------------------|
| CA7.1 | **Dado** o módulo Pix e o Hub sem internet, **quando** o garçom gera o Pix de Ana, **então** o Hub gera um Pix estático com chave, valor R$ 32,05 e identificador da comanda, e o pagamento fica `AGUARDANDO_CONFIRMACAO`. |
| CA7.2 | **Dado** um Pix `AGUARDANDO_CONFIRMACAO`, **quando** o garçom confere o comprovante no celular do cliente e libera a comanda, **então** a comanda passa a `PAGA`, com registro de quem liberou. |
| CA7.3 | **Dado** que a internet voltou, **quando** o Hub sincroniza, **então** os Pix aguardando são conciliados com os recebimentos do provedor pelo identificador e valor, e passam a confirmados. |
| CA7.4 | **Dado** um Pix offline não conciliado em 24 horas, **então** o admin recebe um alerta de divergência com comanda, valor e quem liberou. |
| CA7.5 | **Dado** Pix aguardando confirmação no fechamento do turno, **então** o turno pode ser fechado, e eles aparecem como pendências no relatório (RN-TUR-06). |

### H8 — Pagar a comanda de outra pessoa ou várias de uma vez `CF-46`
**Como** cliente, **quero** pagar a conta de outra pessoa ou de todo mundo, **para** resolver a conta do grupo de uma vez.

| # | Critério de aceite |
|---|--------------------|
| CA8.1 | **Dado** as comandas de Ana (R$ 32,05) e Bruno (R$ 102,55), **quando** Ana paga as duas com um cartão de R$ 134,60, **então** as duas passam a `PAGA` e a de Bruno registra Ana como pagadora. |
| CA8.2 | **Dado** um pagamento agrupado, **então** o valor é distribuído pelo saldo de cada comanda: R$ 32,05 para Ana e R$ 102,55 para Bruno. |
| CA8.3 | **Dado** um pagamento agrupado em dinheiro de R$ 150,00 para Ana e Bruno, **então** o troco é R$ 15,40. |
| CA8.4 | **Dado** um pagamento agrupado com valor diferente da soma dos saldos (sem ser dinheiro com troco), **então** é rejeitado com `VALOR_DIFERENTE_DO_SALDO`. |
| CA8.5 | **Dado** um pagamento agrupado com Pix integrado, **então** é gerada uma única cobrança com a soma dos saldos. |
| CA8.6 | **Dado** Bruno sem celular e Ana pagando só a dele, **quando** o garçom registra o pagamento na comanda de Bruno indicando Ana como pagadora, **então** a comanda de Bruno fica `PAGA` e a de Ana continua com o próprio saldo. |
| CA8.7 | **Dado** comandas de mesas diferentes no mesmo turno, **quando** um pagamento agrupado as inclui, **então** ele é aceito. |

### H9 — Estornar pagamento `CF-47`
**Como** admin, **quero** estornar um pagamento registrado errado, **para** corrigir o caixa.

| # | Critério de aceite |
|---|--------------------|
| CA9.1 | **Dado** o cartão de R$ 102,55 de Bruno no turno aberto, **quando** o admin estorna com motivo "registrado na comanda errada", **então** a comanda de Bruno volta a `FECHAMENTO_SOLICITADO` com saldo R$ 102,55. |
| CA9.2 | **Dado** que a mesa de Bruno já foi liberada, **quando** o pagamento é estornado, **então** a comanda aparece nas pendências do caixa, a mesa continua `LIVRE` e o turno não pode ser fechado até a comanda ser paga ou cancelada. |
| CA9.3 | **Dado** um pagamento de um turno já fechado, **quando** o admin o estorna, **então** o estorno entra como saída no turno aberto atual e a comanda não é reaberta. |
| CA9.4 | **Dado** um Pix integrado confirmado, **quando** o admin o estorna, **então** o sistema solicita a devolução ao provedor e registra o estorno. |
| CA9.5 | **Dado** um estorno em dinheiro, **então** o dinheiro esperado no fechamento do turno é reduzido pelo valor estornado. |
| CA9.6 | **Dado** um estorno sem motivo ou feito por quem não é Admin, **então** é rejeitado com `MOTIVO_OBRIGATORIO` ou `SEM_PERMISSAO`. |

---

## Requisitos não funcionais

- **RNF1**: Todos os cálculos em centavos (inteiros); arredondamento só onde a regra define (RN-PAG-13).
- **RNF2**: Dinheiro, cartão, Pix manual e Pix estático funcionam sem internet.
- **RNF3**: O webhook do provedor Pix é autenticado e idempotente.
- **RNF4**: Todo pagamento, desconto, remoção de taxa, liberação e estorno gera evento imutável com autor, aparelho e data/hora.
- **RNF5**: Testes de propriedade garantem que, para qualquer combinação de itens, cotas, desconto e taxa, o total pago das comandas de uma mesa é igual à soma dos totais individuais.

## Erros de domínio

`SEM_PERMISSAO`, `COMANDA_PAGA`, `TOTAL_MENOR_QUE_PAGO`, `MOTIVO_OBRIGATORIO`, `DESCONTO_INVALIDO`, `VALOR_EXCEDE_SALDO`, `VALOR_DIFERENTE_DO_SALDO`

## Decisões registradas

- Dinheiro é registrado só por Caixa e Admin; cartão e Pix, por qualquer um da equipe (RN-PAG-16).
- Desconto em % ou em reais, só por Caixa e Admin, com motivo, e aparece no relatório do turno (RN-PAG-14).
- Um pagamento pode quitar várias comandas de uma vez (RN-PAG-06).
- Qualquer um da equipe pode remover a taxa de serviço a pedido do cliente, com registro (RN-PAG-02).
- A taxa de serviço incide sobre o subtotal depois do desconto (RN-PAG-13).
- Pix pago a mais vira divergência para o admin, sem crédito automático (RN-PAG-23).
