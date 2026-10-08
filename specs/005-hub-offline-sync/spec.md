# Spec 005 — Hub Local, Modo Offline e Sincronização

- **Status**: Rascunho
- **Regras cobertas**: RN-OFF-01..24, RN-EST-03
- **Epic Jira**: CF-9
- **Depende de**: 008 (estabelecimento, usuários e papéis), 007 (licença e limites do plano)

## Objetivo

Garantir que a operação do restaurante **nunca pare** por falta de internet e que **nenhum dado se perca**, nem quando o próprio Hub falha. O Hub Local é a fonte da verdade da operação dentro do restaurante e sincroniza com a Nuvem sempre que há conexão.

## Fora de escopo

- Hub em Linux ou Android (D-10)
- Aparelhos falarem direto com a Nuvem quando o Hub cai (D-11)
- Mais de um Hub por estabelecimento (RN-EST-03)
- Regras de negócio de cada domínio (specs 001 a 004); aqui só o comportamento offline e de sincronização

---

## Histórias de usuário

### H1 — Instalar e ativar o Hub `CF-48`
**Como** admin, **quero** instalar o Hub no computador do caixa e vinculá-lo ao meu estabelecimento, **para** começar a operar.

| # | Critério de aceite |
|---|--------------------|
| CA1.1 | **Dado** um PC com Windows 10 ou 11 (64 bits), **quando** o admin roda o instalador, **então** o Hub é instalado como serviço que inicia junto com o computador. |
| CA1.2 | **Dado** o painel da Nuvem, **quando** o admin gera um código de ativação, **então** recebe um código de uso único, válido por 15 minutos. |
| CA1.3 | **Dado** um código válido, **quando** o admin o informa no Hub, **então** o Hub fica vinculado ao estabelecimento e baixa da Nuvem a configuração (cardápio, mesas, usuários, impressoras, plano). |
| CA1.4 | **Dado** um código expirado ou já usado, **quando** é informado, **então** é rejeitado com `CODIGO_ATIVACAO_INVALIDO`. |
| CA1.5 | **Dado** um Hub já ativo, **quando** o admin ativa um Hub em outro computador, **então** o anterior é desativado, deixa de operar e de sincronizar, e mostra a mensagem `HUB_DESATIVADO`. |
| CA1.6 | **Dado** um PC com sistema não suportado, **quando** o instalador é executado, **então** ele informa que o sistema não é suportado e não instala. |
| CA1.7 | **Dado** o Hub instalado, **então** um ícone na bandeja do Windows mostra o status (online/offline, eventos pendentes, impressoras), e clicar nele abre a administração do Hub no navegador em `http://localhost`. |
| CA1.8 | **Dado** outro computador da rede, **quando** alguém tenta abrir a administração do Hub pelo endereço da rede, **então** o acesso é recusado (só funciona na própria máquina). |

### H2 — Conectar aparelhos ao Hub `CF-49`
**Como** admin, **quero** conectar os celulares e tablets da equipe ao Hub, **para** que trabalhem pela rede local.

| # | Critério de aceite |
|---|--------------------|
| CA2.1 | **Dado** o Hub ativo, **quando** o admin abre a tela de pareamento, **então** o Hub exibe um QR Code com o endereço local, um token temporário e a impressão digital do certificado do Hub. |
| CA2.2 | **Dado** um aparelho com o app instalado, na mesma rede, **quando** lê o QR Code, **então** fica pareado com o Hub e passa a exibir a tela de login. |
| CA2.8 | **Dado** um computador na rede se passando pelo Hub, **quando** um aparelho pareado tenta se conectar a ele, **então** a conexão é recusada, porque o certificado não corresponde à impressão digital do pareamento. |
| CA2.9 | **Dado** que o admin desativou o garçom João na Nuvem com o Hub offline, **quando** João tenta entrar, **então** o login ainda funciona até a próxima sincronização; depois dela, é rejeitado com `CREDENCIAIS_INVALIDAS`. |
| CA2.3 | **Dado** um aparelho pareado, **quando** o garçom entra com usuário e senha sem internet, **então** o login funciona, validado pelo Hub com as credenciais sincronizadas. |
| CA2.4 | **Dado** o limite de 3 aparelhos do plano Básico já atingido, **quando** um 4º aparelho tenta parear, **então** é rejeitado com `LIMITE_APARELHOS`. |
| CA2.5 | **Dado** que o roteador trocou o endereço do Hub, **quando** os aparelhos perdem a conexão, **então** reencontram o Hub automaticamente na rede, sem novo pareamento. |
| CA2.6 | **Dado** um aparelho pareado, **quando** o admin o remove no Hub, **então** ele é desconectado e precisa ser pareado de novo. |
| CA2.7 | **Dado** credenciais erradas, **quando** alguém tenta entrar, **então** é rejeitado com `CREDENCIAIS_INVALIDAS`. |

### H3 — Operar sem internet `CF-50`
**Como** equipe, **quero** continuar trabalhando normalmente quando a internet cai, **para** não parar o atendimento.

| # | Critério de aceite |
|---|--------------------|
| CA3.1 | **Dado** o Hub sem internet, **quando** a equipe abre mesas, lança pedidos, imprime e registra pagamentos em dinheiro e cartão, **então** tudo funciona como online. |
| CA3.2 | **Dado** o Hub sem internet, **então** todos os aparelhos exibem o indicador "Offline" e a quantidade de eventos pendentes de sincronização. |
| CA3.3 | **Dado** que a internet volta, **então** o indicador muda para "Online" e a quantidade de pendentes cai até zero conforme a sincronização avança. |
| CA3.4 | **Dado** qualquer ação, **então** ela é gravada como evento no Hub antes de ser confirmada ao usuário. |
| CA3.5 | **Dado** o relógio de um aparelho errado em 10 minutos, **quando** ele lança um pedido, **então** o evento usa a data/hora do Hub. |

### H4 — Sincronizar com a Nuvem `CF-51`
**Como** dono do estabelecimento, **quero** que tudo o que acontece no salão chegue à Nuvem, **para** ter relatórios e backup.

| # | Critério de aceite |
|---|--------------------|
| CA4.1 | **Dado** o Hub online, **quando** um evento é gerado, **então** ele é enviado à Nuvem em até 10 segundos. |
| CA4.2 | **Dado** 3 horas offline com 2.000 eventos acumulados, **quando** a internet volta, **então** os eventos são enviados na ordem em que foram aceitos pelo Hub. |
| CA4.3 | **Dado** que um lote foi enviado mas a confirmação se perdeu, **quando** o Hub reenvia, **então** a Nuvem não duplica nada (idempotência). |
| CA4.4 | **Dado** uma falha da Nuvem, **então** o Hub tenta de novo automaticamente, com intervalos crescentes, sem afetar a operação. |
| CA4.5 | **Dado** o painel do admin, **então** ele mostra a data/hora da última sincronização e quantos eventos estão pendentes. |
| CA4.6 | **Dado** turno aberto e o Hub sem sincronizar há mais de 2 horas, **então** o admin recebe um alerta no painel da Nuvem e por e-mail. |
| CA4.7 | **Dado** uma alteração de configuração na Nuvem (ex.: preço), **quando** o Hub está online, **então** ela chega ao Hub e aos aparelhos (RN-OFF-05). |

### H5 — Resolver conflitos `CF-52`
**Como** equipe, **quero** que ações simultâneas sobre a mesma coisa sejam resolvidas de forma previsível, **para** não haver dados inconsistentes.

| # | Critério de aceite |
|---|--------------------|
| CA5.1 | **Dado** dois garçons abrindo a mesa 5 ao mesmo tempo, **quando** os eventos chegam ao Hub, **então** o primeiro aceito vence e o segundo garçom recebe o motivo da rejeição (`MESA_OCUPADA`). |
| CA5.2 | **Dado** dois cancelamentos do mesmo item, **então** o segundo é rejeitado com `ITEM_JA_CANCELADO`. |
| CA5.3 | **Dado** um evento rejeitado, **então** o aparelho mostra ao usuário o que foi rejeitado e por quê, e o remove da fila. |
| CA5.4 | **Dado** a disponibilidade de um produto alterada no Hub e na Nuvem enquanto desconectados, **então** prevalece a alteração mais recente (RN-CAR-08). |

### H6 — Trabalhar com o aparelho sem conexão com o Hub `CF-53`
**Como** garçom, **quero** continuar lançando pedidos quando meu aparelho perde o Wi-Fi, **para** não perder o pedido do cliente.

| # | Critério de aceite |
|---|--------------------|
| CA6.1 | **Dado** o aparelho sem conexão com o Hub, **quando** o garçom lança um pedido, **então** ele fica na fila do aparelho, marcado como "pendente de envio". |
| CA6.2 | **Dado** o aparelho sem conexão com o Hub, **quando** o garçom tenta abrir mesa, registrar pagamento ou fechar turno, **então** a ação é bloqueada com `HUB_INDISPONIVEL`. |
| CA6.3 | **Dado** a conexão restabelecida, **então** a fila é enviada ao Hub na ordem em que foi criada, e cada item recebe aceito ou rejeitado (com motivo). |
| CA6.4 | **Dado** que o aparelho é fechado ou reiniciado com itens na fila, **quando** ele volta, **então** a fila continua lá. |

### H7 — Recuperar a operação quando o Hub falha `CF-54`
**Como** admin, **quero** colocar o restaurante para funcionar de novo se o computador do Hub quebrar, **para** não perder vendas nem dados.

| # | Critério de aceite |
|---|--------------------|
| CA7.1 | **Dado** que o Hub caiu, **então** todos os aparelhos exibem o alerta "Hub indisponível" e passam a operar como em H6. |
| CA7.2 | **Dado** o Hub religado, **então** ele retoma do ponto em que parou e recebe as filas dos aparelhos. |
| CA7.3 | **Dado** que o computador do Hub quebrou, **quando** o admin instala e ativa o Hub em outro computador, **então** o novo Hub baixa da Nuvem a configuração e o último estado sincronizado, inclusive o turno aberto, as mesas e as comandas. |
| CA7.4 | **Dado** eventos que chegaram ao Hub antigo mas não à Nuvem, **quando** o novo Hub é ativado, **então** os aparelhos reenviam as cópias que guardavam, e nada se perde. |
| CA7.5 | **Dado** um evento confirmado pela Nuvem, **então** o aparelho apaga a cópia local desse evento. |
| CA7.6 | **Dado** os aparelhos reenviando eventos que o novo Hub já recebeu da Nuvem, **então** nada é duplicado. |

### H8 — Manter a licença sem internet `CF-55`
**Como** dono, **quero** que uma internet ruim não bloqueie meu restaurante, **para** que a assinatura nunca atrapalhe um serviço em andamento.

| # | Critério de aceite |
|---|--------------------|
| CA8.1 | **Dado** o Hub online, **então** a licença em cache é renovada a cada sincronização. |
| CA8.2 | **Dado** o Hub 6 dias sem internet, **quando** o caixa abre um turno, **então** a abertura funciona e os aparelhos mostram quantos dias restam até o modo restrito. |
| CA8.3 | **Dado** o Hub mais de 7 dias sem internet, **quando** o caixa tenta abrir um turno, **então** é rejeitado com `LICENCA_EXPIRADA`. |
| CA8.4 | **Dado** um turno aberto quando a licença expira, **então** o turno continua funcionando até ser fechado. |
| CA8.5 | **Dado** o modo restrito, **quando** a internet volta e a assinatura está ativa, **então** o modo restrito é retirado automaticamente. |

### H9 — Manter o Hub leve e atualizado `CF-56`
**Como** admin, **quero** que o Hub não encha o disco e se atualize sozinho, **para** não precisar de manutenção.

| # | Critério de aceite |
|---|--------------------|
| CA9.1 | **Dado** dados operacionais com mais de 30 dias já sincronizados, **então** o Hub os remove, e eles continuam disponíveis na Nuvem. |
| CA9.2 | **Dado** eventos com mais de 30 dias ainda não sincronizados, **então** o Hub não os remove. |
| CA9.3 | **Dado** uma nova versão do Hub disponível, **quando** há turno aberto, **então** a atualização é baixada mas só é aplicada depois do fechamento do turno. |
| CA9.4 | **Dado** o banco de dados do Hub, **então** ele é criptografado no disco, e a comunicação com os aparelhos é criptografada e autenticada. |

---

## Requisitos não funcionais

- **RNF1**: Operações na rede local respondem em até 300 ms (RNF da spec 001).
- **RNF2**: O Hub funciona em um PC com 4 GB de RAM e 10 GB livres em disco.
- **RNF3**: Nenhum evento confirmado ao usuário pode ser perdido, mesmo com queda de energia (gravação durável antes da confirmação).
- **RNF4**: Testes automatizados simulam queda de internet, queda do Wi-Fi do aparelho, queda do Hub e substituição do Hub, verificando que nenhum evento se perde ou duplica.

## Erros de domínio

`CODIGO_ATIVACAO_INVALIDO`, `HUB_DESATIVADO`, `LIMITE_APARELHOS`, `CREDENCIAIS_INVALIDAS`, `HUB_INDISPONIVEL`, `LICENCA_EXPIRADA`

## Decisões registradas

- Hub roda em Windows 10/11 no MVP (RN-OFF-10, D-10).
- Se o Hub cair, os aparelhos enfileiram pedidos e o Hub é recuperado a partir da Nuvem e das cópias dos aparelhos (RN-OFF-14 a 16, D-11).
- O Hub guarda os últimos 30 dias; o histórico completo fica na Nuvem (RN-OFF-17).
- Hub é serviço do Windows com administração via navegador local; aparelhos da equipe e KDS são apps instalados (RN-OFF-23, RN-OFF-24).
- Usuário desativado com o Hub offline ainda entra até a próxima sincronização (RN-OFF-12).
- Alerta de Hub sem sincronizar vai para o painel e por e-mail (RN-OFF-21).
