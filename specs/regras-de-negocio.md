# Regras de Negócio — Conta Fácil (MVP)

Catálogo único das regras de negócio. Specs e planos referenciam regras pelo ID (ex.: `RN-COM-03`). Uma regra alterada mantém o ID; uma regra removida é marcada como **[REVOGADA]**, nunca reaproveitada.

## Glossário

| Termo | Código | Definição |
|-------|--------|-----------|
| Estabelecimento | `Tenant` | Bar/restaurante cliente do SaaS |
| Turno | `Shift` | Período de operação/caixa (ex.: jantar de sexta) |
| Mesa | `Table` | Local físico cadastrado |
| Sessão de mesa | `TableSession` | Uma ocupação da mesa, do início ao fim |
| Comanda | `Tab` | Conta individual de uma pessoa dentro de uma sessão de mesa |
| Item de pedido | `OrderItem` | Produto lançado em uma ou mais comandas |
| Hub Local | `Hub` | Servidor local do restaurante |
| Pré-conta | `Bill` | Documento não fiscal com o valor a pagar |

## Atores

| Ator | Responsabilidade |
|------|------------------|
| **Admin** | Configura estabelecimento, cardápio, mesas, equipe, impressoras e taxa de serviço |
| **Caixa** | Abre/fecha turno, registra pagamentos |
| **Garçom** | Abre mesas, cria comandas, lança pedidos, solicita fechamento |
| **Cozinha/Bar** | Atualiza status de preparo (KDS) |
| **Cliente** | Paga sua comanda (não opera o sistema no MVP) |

---

## RN-EST — Estabelecimento

- **RN-EST-01**: Cada estabelecimento é um tenant isolado. Nenhum dado é visível entre tenants.
- **RN-EST-02**: O estabelecimento configura a taxa de serviço (padrão 10%, pode ser 0%).
- **RN-EST-03**: Cada estabelecimento possui exatamente um Hub Local ativo.

## RN-TUR — Turno (caixa)

- **RN-TUR-01**: Só pode existir **um turno aberto** por estabelecimento.
- **RN-TUR-02**: Mesas só podem ser abertas com turno aberto.
- **RN-TUR-03**: A abertura de turno registra responsável, data/hora e **fundo de troco** (valor inicial em dinheiro, pode ser zero).
- **RN-TUR-04**: O turno só pode ser fechado quando **não houver comandas abertas** (todas pagas ou canceladas).
- **RN-TUR-05**: No fechamento, o sistema apresenta o total por forma de pagamento e o **dinheiro esperado** (fundo + recebimentos em dinheiro). O responsável informa o valor contado; a diferença é registrada.
- **RN-TUR-06**: Pagamentos Pix `AGUARDANDO_CONFIRMACAO` não impedem o fechamento, mas são listados como pendências do turno.
- **RN-TUR-07**: Turno pode atravessar a meia-noite; ele pertence à data de abertura.
- **RN-TUR-08**: Abertura e fechamento de turno funcionam offline.
- **RN-TUR-09**: Somente **Caixa** e **Admin** podem abrir e fechar turno.

## RN-MES — Mesa

- **RN-MES-01**: Cada mesa tem identificador único (número/nome) dentro do estabelecimento.
- **RN-MES-02**: Estados da mesa: `LIVRE → OCUPADA → EM_FECHAMENTO → LIVRE`.
- **RN-MES-03**: Ao abrir uma mesa, cria-se uma **sessão de mesa**. Uma mesa ocupada tem exatamente uma sessão ativa.
- **RN-MES-04**: A mesa passa a `EM_FECHAMENTO` quando ao menos uma pré-conta da mesa (consolidada) é solicitada.
- **RN-MES-05**: A mesa volta a `LIVRE` (sessão encerrada) somente quando todas as comandas da sessão estão `PAGA` ou `CANCELADA`.
- **RN-MES-06**: Uma mesa desativada pelo admin não pode ser aberta, mas mantém histórico.
- **RN-MES-07**: Juntar mesas está **fora do MVP**.
- **RN-MES-08**: A **capacidade** (número de lugares) é um campo **opcional**, configurado pelo estabelecimento por mesa. Quando informada, é exibida no mapa de mesas junto com a quantidade de comandas. Exceder a capacidade **não bloqueia** a abertura de comandas; apenas sinaliza a mesa como acima da capacidade.

## RN-COM — Comanda

- **RN-COM-01**: Cada pessoa da mesa tem uma **comanda individual**, sempre vinculada a uma sessão de mesa.
- **RN-COM-02**: Somente a equipe (garçom/caixa/admin) cria comandas, informando nome ou apelido da pessoa.
- **RN-COM-03**: O nome da comanda é único dentro da sessão de mesa (ex.: não pode haver dois "João"; usar "João 2").
- **RN-COM-04**: Ao abrir uma mesa, é obrigatório criar ao menos uma comanda.
- **RN-COM-05**: Estados: `ABERTA → FECHAMENTO_SOLICITADO → PAGA`, ou `CANCELADA`.
- **RN-COM-06**: Comanda em `FECHAMENTO_SOLICITADO` não recebe novos itens; pode ser reaberta (volta a `ABERTA`) enquanto não houver pagamento registrado.
- **RN-COM-07**: Comanda só pode ser cancelada se **não tiver itens ativos** (todos cancelados ou transferidos).
- **RN-COM-08**: Uma comanda pode ser **transferida para outra mesa** ocupada (a pessoa trocou de lugar), levando todos os itens. Itens compartilhados com pessoas da mesa de origem permanecem divididos.
- **RN-COM-09**: Cada comanda é fechada e paga **independentemente**; uma pessoa pode ir embora sem que a mesa feche.

## RN-CAR — Cardápio

- **RN-CAR-01**: Cardápio organizado em **categorias** e **produtos**.
- **RN-CAR-02**: Produto possui nome, descrição, preço (centavos), foto (opcional) e **destino de produção** (`COZINHA`, `BAR` ou `NENHUM` — ex.: item pronto como refrigerante em lata).
- **RN-CAR-03**: Produto pode ser marcado como **indisponível** temporariamente, sem exclusão.
- **RN-CAR-04**: Produtos não são excluídos fisicamente se já foram pedidos; são arquivados.
- **RN-CAR-05**: O cardápio é editado na Nuvem e replicado ao Hub quando houver conexão.
- **RN-CAR-06**: Adicionais/variações com preço estão **fora do MVP**; somente observação livre.
- **RN-CAR-07**: Somente o **Admin** cria e edita categorias e produtos.
- **RN-CAR-08**: **Disponibilidade é dado operacional**: pode ser alterada por Admin, Caixa e Cozinha/Bar, inclusive **offline** (via Hub). Se a mesma disponibilidade for alterada no Hub e na Nuvem enquanto estavam desconectados, prevalece a alteração **mais recente**.
- **RN-CAR-09**: Nome do produto é único dentro do estabelecimento (entre produtos não arquivados). Nome da categoria também.
- **RN-CAR-10**: Produto pode ter um **código curto** opcional (ex.: `105`), único no estabelecimento, para busca rápida pelo garçom.
- **RN-CAR-11**: Categorias e produtos têm **ordem de exibição** definida pelo Admin.
- **RN-CAR-12**: Categoria **desativada** oculta seus produtos no lançamento de pedidos. Categoria só pode ser **excluída** se não tiver produtos (ativos ou arquivados).
- **RN-CAR-13**: Preço em centavos, **maior ou igual a zero** (zero permite cortesias). Toda alteração de preço fica registrada no **histórico de preços** (valor anterior, novo, autor, data/hora).
- **RN-CAR-14**: Produto nunca pedido pode ser **excluído**; produto já pedido só pode ser **arquivado** (RN-CAR-04). Produto arquivado não aparece no lançamento, mas pode ser restaurado.
- **RN-CAR-15**: Foto opcional (JPG, PNG ou WebP, até 5 MB), redimensionada pela Nuvem. Fotos **não são necessárias para operar**: o Hub guarda apenas miniaturas, e a falta delas não impede lançamentos.
- **RN-CAR-16**: O estabelecimento tem um **cardápio digital** público, **somente leitura**, acessado pelo cliente via QR Code ou link, sem login. Disponível em **todos os planos**. O cliente **não faz pedidos** por ele (RN-PED-01).
- **RN-CAR-17**: O cardápio digital exibe categorias ativas e produtos não arquivados, na ordem do admin, com nome, descrição, preço e foto. Produtos indisponíveis aparecem marcados como **"indisponível no momento"**. Código curto e destino de produção não são exibidos.
- **RN-CAR-18**: O cardápio digital é servido pela **Nuvem** (o cliente usa a própria internet) e reflete a última sincronização com o Hub. Com o Hub offline, a disponibilidade exibida pode estar desatualizada.
- **RN-CAR-19**: Há **um QR Code por estabelecimento**, que o Admin pode baixar para impressão. A identidade visual segue RN-WL (marca Conta Fácil no Básico e no Pro; white label no Premium).
- **RN-CAR-20**: O Admin pode **desativar** o cardápio digital; o link passa a exibir uma página de indisponível.

## RN-PED — Pedidos

- **RN-PED-01**: Somente a equipe lança pedidos. O cliente não pede pelo sistema.
- **RN-PED-02**: Todo item pertence a **uma comanda**, ou é **compartilhado** entre N comandas da mesma sessão de mesa.
- **RN-PED-03**: Item compartilhado: valor dividido igualmente entre as comandas; os centavos restantes são atribuídos, um a um, às primeiras comandas da divisão (ordem de seleção).
- **RN-PED-04**: O **preço é congelado** no momento do lançamento.
- **RN-PED-05**: Estados do item: `PENDENTE → EM_PREPARO → PRONTO → ENTREGUE`, ou `CANCELADO`. Itens com destino `NENHUM` nascem `ENTREGUE`.
- **RN-PED-06**: Item pode ser cancelado por garçom enquanto `PENDENTE`. Após isso, somente caixa/admin, com **motivo obrigatório**.
- **RN-PED-07**: Item pode ser **transferido** entre comandas da mesma sessão, ou ter sua divisão alterada, enquanto as comandas envolvidas estiverem `ABERTA`.
- **RN-PED-08**: Cada item aceita **observação livre** (ex.: "sem cebola").
- **RN-PED-09**: Lançar um pedido gera **ticket de produção** (impressão) e/ou envio ao **KDS**, conforme plano e configuração, agrupado por destino.
- **RN-PED-10**: Produto indisponível não pode ser lançado.

## RN-PAG — Fechamento e pagamento

- **RN-PAG-01**: Total da comanda = itens individuais + cotas de itens compartilhados + taxa de serviço (se aceita).
- **RN-PAG-02**: A taxa de serviço é **opcional** e pode ser removida por comanda, a pedido do cliente.
- **RN-PAG-03**: Formas de pagamento no MVP: dinheiro, cartão (registrado manualmente — maquininha externa) e Pix.
- **RN-PAG-04**: Uma comanda aceita **múltiplos pagamentos** (ex.: parte em dinheiro, parte no Pix) até quitar o total.
- **RN-PAG-05**: Pagamento em dinheiro registra valor recebido e troco.
- **RN-PAG-06**: Uma pessoa pode pagar a comanda de outra; o pagamento registra a comanda pagadora (quando houver).
- **RN-PAG-07**: Pix é acessado via interface abstrata `PaymentProvider`; o provedor concreto será definido depois.
- **RN-PAG-08**: **Online**: gera Pix dinâmico (cobrança com `txid`); confirmação automática via webhook.
- **RN-PAG-09**: **Offline**: o Hub gera **Pix estático** (chave + valor + identificador da comanda). O pagamento fica `AGUARDANDO_CONFIRMACAO`.
- **RN-PAG-10**: Com pagamento `AGUARDANDO_CONFIRMACAO`, o caixa/garçom pode **liberar a comanda** com base no comprovante do cliente; a conciliação ocorre quando a conexão voltar.
- **RN-PAG-11**: Pix offline não conciliado em **24h** (configurável) gera alerta de divergência para o admin.
- **RN-PAG-12**: Estorno de pagamento: somente admin, com motivo; reabre a comanda se o turno ainda estiver aberto.

## RN-IMP — Impressão (impressoras térmicas)

- **RN-IMP-01**: Impressoras térmicas compatíveis com **ESC/POS**, conectadas ao Hub via rede (TCP/9100) ou USB. Larguras suportadas: 58mm e 80mm.
- **RN-IMP-02**: Impressão é feita pelo Hub e **funciona offline**.
- **RN-IMP-03**: Documentos: ticket de produção, pré-conta individual, pré-conta da mesa (consolidada, com subtotal por pessoa), comprovante de pagamento e relatório de fechamento de turno.
- **RN-IMP-04**: Cada destino de produção (`COZINHA`, `BAR`) é mapeado para uma impressora. No plano Básico (sem KDS), o ticket de produção é o meio de envio do pedido à produção.
- **RN-IMP-05**: Pré-conta inclui QR Code do Pix (dinâmico se online, estático se offline) quando o módulo Pix estiver habilitado.
- **RN-IMP-06**: Falha de impressão **nunca bloqueia** a operação: o documento entra em fila com novas tentativas e alerta visível; reimpressão manual disponível.
- **RN-IMP-07**: Reimpressões são marcadas como **"2ª VIA"**.
- **RN-IMP-08**: Todos os documentos impressos contêm **"NÃO É DOCUMENTO FISCAL"**.
- **RN-IMP-09**: Integração com maquininhas Smart POS (Stone, PagBank, Getnet etc.) está **fora do MVP**.

## RN-OFF — Modo offline e sincronização

- **RN-OFF-01**: Toda entidade operacional recebe ID gerado na origem (UUIDv7).
- **RN-OFF-02**: Toda alteração operacional gera um evento imutável em fila local (outbox).
- **RN-OFF-03**: Sincronização Hub ↔ Nuvem é idempotente; reenvio não duplica efeitos.
- **RN-OFF-04**: Fonte da verdade: Hub para operação; Nuvem para configuração (ver constitution).
- **RN-OFF-05**: Alterações de cardápio/configuração chegam ao Hub ao reconectar; pedidos já lançados mantêm preço congelado (RN-PED-04).
- **RN-OFF-06**: Conflito na mesma entidade: vence o primeiro evento aceito pelo Hub; o segundo é rejeitado e o aparelho é notificado com o motivo.
- **RN-OFF-07**: Aparelhos mantêm fila local se perderem conexão com o Hub e reenviam ao reconectar.
- **RN-OFF-08**: Todos os aparelhos exibem status de conexão e quantidade de eventos pendentes.
- **RN-OFF-09**: O Hub mantém **licença em cache** válida por **7 dias** (configurável) sem contato com a Nuvem. Após o prazo, entra em **modo restrito** (não abre novos turnos), mas **nunca interrompe um turno em andamento**.

## RN-PLA — Planos e módulos

- **RN-PLA-01**: Assinatura mensal por estabelecimento; cada plano habilita um conjunto de **módulos**.
- **RN-PLA-02**: Matriz de módulos:

| Módulo | Básico | Pro | Premium |
|--------|:------:|:---:|:-------:|
| Mesas e comandas individuais | ✅ | ✅ | ✅ |
| Modo offline (Hub Local) | ✅ | ✅ | ✅ |
| Impressão térmica | ✅ | ✅ | ✅ |
| Cardápio digital (QR, só leitura) | ✅ | ✅ | ✅ |
| Pagamento manual (dinheiro/cartão) | ✅ | ✅ | ✅ |
| Pix integrado | ❌ | ✅ | ✅ |
| KDS Cozinha | ❌ | ✅ | ✅ |
| KDS Bar (separado) | ❌ | ❌ | ✅ |
| Usuários/aparelhos | 3 | 10 | ilimitado |
| Relatórios | básico | completo | completo + exportação |
| White label (logo, cores, sem marca do sistema) | ❌ | ❌ | ✅ |
| Domínio próprio | ❌ | ❌ | ✅ |

- **RN-PLA-03**: Módulos são validados na Nuvem e no Hub (via licença em cache).
- **RN-PLA-04**: Mudança de plano (upgrade ou downgrade) é aplicada a partir do **próximo turno**, nunca durante um turno aberto.

## RN-WL — Identidade visual e white label

- **RN-WL-01**: White label é um **módulo** de plano (RN-PLA-02), resolvido em tempo de execução por tenant. Não há build nem deploy separado por cliente.
- **RN-WL-02**: Em **todos os planos**, o nome do estabelecimento aparece nos aparelhos e nos documentos impressos. Isso é identificação, não white label.
- **RN-WL-03**: Sem o módulo white label (Básico e Pro), aplica-se o **tema padrão Conta Fácil** e a marca do sistema fica visível: logo nos aparelhos e rodapé "Powered by Conta Fácil" nos documentos impressos.
- **RN-WL-04**: Com o módulo white label (Premium), o estabelecimento configura **logo, cor primária, cor secundária e nome de exibição**, e a marca Conta Fácil é removida de aparelhos e documentos impressos.
- **RN-WL-05**: Com o módulo de domínio próprio (Premium), o painel web do estabelecimento pode ser acessado por um domínio do cliente (ex.: `gestao.restaurantex.com.br`), com certificado HTTPS emitido pelo sistema. Sem o módulo, o acesso é por subdomínio padrão (ex.: `restaurantex.contafacil.app`).
- **RN-WL-06**: A identidade visual é **configuração** (fonte da verdade: Nuvem) e é replicada ao Hub com os arquivos (logo) em cache local, para funcionar offline.
- **RN-WL-07**: Para impressão térmica, o logo é convertido automaticamente em **bitmap monocromático** na largura da impressora (58mm/80mm).
- **RN-WL-08**: Logo aceito: PNG ou SVG, até 1 MB. Cores com contraste insuficiente para leitura geram **alerta** ao admin, sem bloquear.
- **RN-WL-09**: Informações obrigatórias **não são afetadas** pelo tema: "NÃO É DOCUMENTO FISCAL" (RN-IMP-08), status de conexão (RN-OFF-08) e textos legais.
- **RN-WL-10**: No downgrade, o tema padrão volta a valer a partir do próximo turno (RN-PLA-04). A configuração personalizada fica guardada por **90 dias** para ser restaurada em caso de novo upgrade.

---

## Decisões em aberto

| # | Tema | Situação |
|---|------|----------|
| D-01 | Emissão de **NFC-e** (nota fiscal) | Fora do MVP. Restaurante emite em sistema próprio. Avaliar como módulo futuro. |
| D-02 | Provedor Pix concreto | Interface abstrata; escolher depois (Efí, Asaas, Mercado Pago...). |
| D-03 | Juntar mesas | Fora do MVP. |
| D-04 | Cliente acompanhar a própria comanda pelo celular (QR) | Fora do MVP. O cardápio digital (só leitura) está no MVP (RN-CAR-16). |
| D-05 | Integração com Smart POS | Fora do MVP. |
| D-06 | App nativo com marca própria nas lojas (App Store/Play Store) por cliente | Fora do MVP. White label no MVP é só visual, em tempo de execução. |
| D-07 | White label para revendas (agência revende o sistema com a própria marca para vários restaurantes) | **Descartado.** O cliente do SaaS é sempre o estabelecimento; não haverá nível de revenda. |
