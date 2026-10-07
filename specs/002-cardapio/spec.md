# Spec 002 — Cardápio

- **Status**: Rascunho
- **Regras cobertas**: RN-CAR-01..20
- **Epic Jira**: CF-6
- **Depende de**: 008 (Estabelecimento, equipe e permissões) para papéis; 005 (Hub Local) para a replicação

## Objetivo

Permitir que o estabelecimento monte e mantenha o cardápio (categorias e produtos), que a equipe encontre rapidamente os produtos ao lançar pedidos, inclusive sem internet, e que o cliente consulte o cardápio no próprio celular. O cardápio é configuração (fonte da verdade: Nuvem), exceto a disponibilidade, que é operacional e pode mudar no Hub durante o serviço.

## Fora de escopo

- Lançamento de pedidos (spec 003)
- Adicionais e variações com preço (RN-CAR-06)
- Pedidos feitos pelo cliente (o cardápio digital é só leitura)
- Preços diferentes por horário (happy hour) ou por canal

---

## Histórias de usuário

### H1 — Gerenciar categorias `CF-22`
**Como** admin, **quero** criar, renomear, ordenar, desativar e excluir categorias, **para** organizar o cardápio.

| # | Critério de aceite |
|---|--------------------|
| CA1.1 | **Dado** um admin, **quando** ele cria a categoria "Bebidas", **então** ela é criada ativa, no fim da ordem de exibição. |
| CA1.2 | **Dado** que já existe a categoria "Bebidas", **quando** o admin tenta criar outra "Bebidas", **então** é rejeitado com `NOME_DUPLICADO`. |
| CA1.3 | **Dado** as categorias "Entradas", "Pratos" e "Bebidas", **quando** o admin move "Bebidas" para o topo, **então** a nova ordem é "Bebidas", "Entradas", "Pratos". |
| CA1.4 | **Dado** a categoria "Sobremesas" com produtos, **quando** o admin a desativa, **então** seus produtos deixam de aparecer no lançamento de pedidos, mas continuam cadastrados. |
| CA1.5 | **Dado** a categoria "Sobremesas" com produtos (ativos ou arquivados), **quando** o admin tenta excluí-la, **então** é rejeitado com `CATEGORIA_COM_PRODUTOS`. |
| CA1.6 | **Dado** a categoria "Sazonais" sem produtos, **quando** o admin a exclui, **então** ela é removida. |
| CA1.7 | **Dado** um usuário com papel Garçom, Caixa ou Cozinha/Bar, **quando** ele tenta criar ou editar categoria, **então** é rejeitado com `SEM_PERMISSAO`. |

### H2 — Cadastrar produto `CF-23`
**Como** admin, **quero** cadastrar produtos com preço e destino de produção, **para** que a equipe possa lançá-los.

| # | Critério de aceite |
|---|--------------------|
| CA2.1 | **Dado** a categoria "Bebidas", **quando** o admin cadastra "Chopp 300ml", R$ 12,90, destino `BAR`, **então** o produto é criado disponível, com preço armazenado como 1290 centavos. |
| CA2.2 | **Dado** um cadastro sem categoria, **quando** o admin salva, **então** é rejeitado com `CATEGORIA_OBRIGATORIA`. |
| CA2.3 | **Dado** um cadastro sem destino de produção, **quando** o admin salva, **então** é rejeitado com `DESTINO_OBRIGATORIO`. |
| CA2.4 | **Dado** preço negativo, **quando** o admin salva, **então** é rejeitado com `PRECO_INVALIDO`. |
| CA2.5 | **Dado** preço R$ 0,00, **quando** o admin salva "Água (cortesia)", **então** o produto é criado normalmente. |
| CA2.6 | **Dado** que já existe "Chopp 300ml" não arquivado, **quando** o admin cadastra outro "Chopp 300ml", **então** é rejeitado com `NOME_DUPLICADO`. |
| CA2.7 | **Dado** que o produto "Chopp 300ml" tem código `105`, **quando** o admin cadastra outro produto com código `105`, **então** é rejeitado com `CODIGO_DUPLICADO`. |
| CA2.8 | **Dado** uma foto de 8 MB ou em formato GIF, **quando** o admin envia, **então** é rejeitada com `FOTO_INVALIDA` e o produto pode ser salvo sem foto. |

### H3 — Editar produto e preço `CF-24`
**Como** admin, **quero** alterar dados e preço de um produto, **para** manter o cardápio atualizado.

| # | Critério de aceite |
|---|--------------------|
| CA3.1 | **Dado** "Chopp 300ml" a R$ 12,90, **quando** o admin altera para R$ 13,90, **então** novos lançamentos usam R$ 13,90 e o histórico registra 1290 → 1390, autor e data/hora. |
| CA3.2 | **Dado** um "Chopp 300ml" já lançado a R$ 12,90 numa comanda aberta, **quando** o preço muda para R$ 13,90, **então** o item já lançado continua R$ 12,90 (RN-PED-04). |
| CA3.3 | **Dado** o produto com destino `BAR`, **quando** o admin muda para `COZINHA`, **então** apenas novos lançamentos vão para a cozinha. |
| CA3.4 | **Dado** um usuário que não é Admin, **quando** ele tenta editar o produto, **então** é rejeitado com `SEM_PERMISSAO`. |

### H4 — Marcar produto como indisponível `CF-25`
**Como** cozinha, bar ou caixa, **quero** marcar um produto como esgotado durante o serviço, **para** que o garçom não lance o que não temos.

| # | Critério de aceite |
|---|--------------------|
| CA4.1 | **Dado** "Picanha" disponível, **quando** a cozinha a marca como indisponível, **então** ela aparece como indisponível em todos os aparelhos e não pode ser lançada (RN-PED-10). |
| CA4.2 | **Dado** que o Hub está sem internet, **quando** o caixa marca "Picanha" como indisponível, **então** a alteração vale imediatamente em todos os aparelhos da rede local e fica pendente de sincronização. |
| CA4.3 | **Dado** "Picanha" indisponível, **quando** a cozinha a marca como disponível, **então** ela volta a poder ser lançada. |
| CA4.4 | **Dado** que, sem conexão, o Hub marcou "Picanha" como indisponível às 20h10 e o admin a marcou como disponível na Nuvem às 20h05, **quando** a conexão volta, **então** prevalece a alteração das 20h10 (indisponível). |
| CA4.5 | **Dado** um usuário com papel Garçom, **quando** ele tenta alterar a disponibilidade, **então** é rejeitado com `SEM_PERMISSAO`. |

### H5 — Arquivar, restaurar e excluir produto `CF-26`
**Como** admin, **quero** retirar produtos do cardápio sem perder o histórico, **para** manter o cardápio limpo.

| # | Critério de aceite |
|---|--------------------|
| CA5.1 | **Dado** "Caipirinha de caju" já pedida alguma vez, **quando** o admin tenta excluí-la, **então** é rejeitado com `PRODUTO_JA_PEDIDO`, `CARDAPIO_DIGITAL_DESATIVADO`, e o sistema oferece arquivar. |
| CA5.2 | **Dado** "Caipirinha de caju" já pedida, **quando** o admin a arquiva, **então** ela some do lançamento de pedidos, mas continua nos relatórios e comandas antigas. |
| CA5.3 | **Dado** "Caipirinha de caju" arquivada, **quando** o admin a restaura, **então** ela volta ao cardápio com os mesmos dados. |
| CA5.4 | **Dado** "Caipirinha de caju" arquivada e um novo produto ativo com o mesmo nome, **quando** o admin tenta restaurar a arquivada, **então** é rejeitado com `NOME_DUPLICADO`. |
| CA5.5 | **Dado** um produto nunca pedido, **quando** o admin o exclui, **então** ele é removido. |

### H6 — Consultar cardápio no lançamento `CF-27`
**Como** garçom, **quero** encontrar produtos rapidamente por categoria, nome ou código, **para** lançar pedidos sem demora.

| # | Critério de aceite |
|---|--------------------|
| CA6.1 | **Dado** o cardápio, **quando** o garçom abre o lançamento, **então** vê apenas categorias ativas e produtos não arquivados, na ordem definida pelo admin. |
| CA6.2 | **Dado** o produto "Chopp 300ml", **quando** o garçom busca "chop", **então** o produto aparece (busca sem diferenciar maiúsculas e acentos). |
| CA6.3 | **Dado** o produto com código `105`, **quando** o garçom digita `105`, **então** o produto aparece como primeiro resultado. |
| CA6.4 | **Dado** "Picanha" indisponível, **quando** o garçom vê o cardápio, **então** ela aparece marcada como indisponível e não pode ser selecionada. |
| CA6.5 | **Dado** que o Hub está sem internet, **quando** o garçom consulta o cardápio, **então** a consulta funciona normalmente com os dados do Hub. |
| CA6.6 | **Dado** um produto sem miniatura no Hub, **quando** o garçom o vê, **então** ele aparece sem foto e pode ser lançado normalmente. |

### H7 — Receber alterações do cardápio no Hub `CF-28`
**Como** estabelecimento, **quero** que as alterações feitas no painel cheguem ao salão, **para** que a equipe trabalhe sempre com o cardápio atual.

| # | Critério de aceite |
|---|--------------------|
| CA7.1 | **Dado** o Hub online, **quando** o admin altera um produto na Nuvem, **então** a alteração chega aos aparelhos em até 30 segundos. |
| CA7.2 | **Dado** o Hub offline, **quando** o admin altera preços na Nuvem, **então** os garçons continuam lançando com os preços antigos até a conexão voltar. |
| CA7.3 | **Dado** que a conexão volta, **quando** o Hub recebe as alterações pendentes, **então** os novos preços valem para os próximos lançamentos, e os itens já lançados mantêm o preço original. |
| CA7.4 | **Dado** que o Hub está há mais tempo sem sincronizar o cardápio, **então** os aparelhos exibem o aviso "cardápio pode estar desatualizado" com a data da última sincronização. |

### H8 — Consultar cardápio digital (cliente) `CF-29`
**Como** cliente, **quero** ver o cardápio no meu celular lendo um QR Code, **para** escolher o que pedir ao garçom.

| # | Critério de aceite |
|---|--------------------|
| CA8.1 | **Dado** o cardápio digital ativo, **quando** o cliente lê o QR Code, **então** vê categorias ativas e produtos não arquivados, na ordem do admin, com nome, descrição, preço e foto, sem precisar de login. |
| CA8.2 | **Dado** "Picanha" indisponível, **quando** o cliente vê o cardápio, **então** ela aparece como "indisponível no momento". |
| CA8.3 | **Dado** um produto arquivado ou de categoria desativada, **quando** o cliente vê o cardápio, **então** ele não aparece. |
| CA8.4 | **Dado** o cardápio digital, **então** não há nenhuma opção de fazer pedido, e código curto e destino de produção não são exibidos. |
| CA8.5 | **Dado** que o Hub está sem internet, **quando** o cliente lê o QR Code com a internet do próprio celular, **então** o cardápio abre normalmente, com a disponibilidade da última sincronização. |
| CA8.6 | **Dado** um estabelecimento no plano Básico ou Pro, **então** o cardápio digital exibe a marca Conta Fácil; no Premium com white label, exibe só a identidade do estabelecimento (RN-WL-03, RN-WL-04). |

### H9 — Disponibilizar o cardápio digital (admin) `CF-30`
**Como** admin, **quero** obter o QR Code do cardápio digital e poder desativá-lo, **para** colocá-lo nas mesas.

| # | Critério de aceite |
|---|--------------------|
| CA9.1 | **Dado** um estabelecimento novo, **então** o cardápio digital já nasce ativo, com link e QR Code próprios. |
| CA9.2 | **Dado** o painel do admin, **quando** ele baixa o QR Code, **então** recebe uma imagem pronta para impressão com o QR e o nome do estabelecimento. |
| CA9.3 | **Dado** o cardápio digital ativo, **quando** o admin o desativa, **então** o link passa a exibir uma página de indisponível (`CARDAPIO_DIGITAL_DESATIVADO`). |
| CA9.4 | **Dado** o cardápio digital desativado, **quando** o admin o reativa, **então** o mesmo link e o mesmo QR voltam a funcionar. |
| CA9.5 | **Dado** um usuário que não é Admin, **quando** ele tenta desativar o cardápio digital, **então** é rejeitado com `SEM_PERMISSAO`. |

---

## Requisitos não funcionais

- **RNF0**: O cardápio digital carrega em até 2 s em conexão móvel 4G e funciona em qualquer navegador de celular, sem instalar app.
- **RNF1**: A consulta e a busca do cardápio respondem em até 200 ms na rede local, com até 1.000 produtos.
- **RNF2**: Toda alteração gera evento imutável com autor e data/hora (RN-OFF-02).
- **RNF3**: Preços sempre em centavos (inteiros); a formatação em reais é feita só na exibição.

## Erros de domínio

`SEM_PERMISSAO`, `NOME_DUPLICADO`, `CODIGO_DUPLICADO`, `CATEGORIA_OBRIGATORIA`, `CATEGORIA_COM_PRODUTOS`, `DESTINO_OBRIGATORIO`, `PRECO_INVALIDO`, `FOTO_INVALIDA`, `PRODUTO_JA_PEDIDO`, `CARDAPIO_DIGITAL_DESATIVADO`

## Decisões registradas

- Cardápio digital para o cliente (QR, só leitura) entra no MVP, em todos os planos (RN-CAR-16 a 20).
- Disponibilidade pode ser alterada por Admin, Caixa e Cozinha/Bar; Garçom não (RN-CAR-08).
- Preço zero é permitido, para cortesias (RN-CAR-13).
