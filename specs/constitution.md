# Constitution — Conta Fácil

Princípios que guiam todas as specs, planos e implementações. Alterar este documento exige decisão explícita e registro no histórico de commits.

## 1. Princípios de produto

1. **Comanda individual é o núcleo.** Toda funcionalidade deve preservar a rastreabilidade de "quem consumiu o quê".
2. **A operação nunca para.** Falta de internet, de impressora ou de integração externa jamais impede lançar pedidos ou fechar contas.
3. **Só a equipe opera no MVP.** O cliente não faz pedidos nem cria comandas; interage apenas para pagar.
4. **Dinheiro é exato.** Valores monetários em centavos (inteiros). Nenhum arredondamento implícito; regras de divisão são explícitas.
5. **Histórico é imutável.** Nada operacional é apagado: cancelamentos, estornos e transferências são novos registros com autor e motivo.

## 2. Arquitetura: Hub Local + Nuvem

```
☁️  NUVEM (NestJS + PostgreSQL)
    cadastro, planos/assinatura, cardápio (fonte da verdade),
    relatórios, backup, integração Pix (webhooks)
              ▲
              │ sincronização assíncrona por eventos
              ▼
🖥️  HUB LOCAL (NestJS + SQLite) — na rede do restaurante
    fonte da verdade da OPERAÇÃO: turno, mesas, comandas,
    pedidos, pagamentos, fila de impressão
       ▲            ▲            ▲             ▲
   📱 Garçom    📱 Garçom    📺 KDS      🖨️ Impressoras térmicas
```

- O Hub Local roda em **equipamento do próprio restaurante**. No MVP, um PC com **Windows 10/11** (em geral o do caixa), como serviço do sistema.
- Hub e Nuvem compartilham o mesmo código de domínio; o "modo" (local/nuvem) é configuração de deploy.
- Aparelhos da equipe falam **somente com o Hub** durante a operação.
- O modo offline está disponível em **todos os planos**.

### Aplicações

| Aplicação | Forma | Por quê |
|-----------|-------|---------|
| **Hub Local** | **Serviço do Windows** (NestJS + SQLite empacotados), sem janela, com ícone de status na bandeja | Precisa rodar em segundo plano, iniciar com o Windows, receber conexões da rede local, falar com impressoras e gravar dados de forma durável — nada disso é possível no navegador |
| **Administração do Hub** (ativação, pareamento, impressoras, status) | **Página web local** em `http://localhost`, aberta pelo ícone da bandeja e acessível só na própria máquina | Evita construir interface nativa de Windows; mesma tecnologia web do resto |
| **Aparelhos da equipe** (garçom, caixa) e **KDS** | **App instalado** (Android e iOS; Capacitor ou React Native, a definir no plano técnico) | Fila offline durável (o navegador pode apagar dados), descoberta do Hub na rede local e conexão segura com o Hub sem certificado público |
| **Painel do Admin** | **Web**, servido pela Nuvem | Configuração e relatórios; não precisa funcionar offline |
| **Cardápio digital do cliente** | **Web**, servido pela Nuvem | Sem instalar nada; só leitura |

### Fontes da verdade

| Dado | Fonte da verdade |
|------|------------------|
| Turno, mesas, comandas, pedidos, pagamentos registrados | Hub Local |
| Cardápio, preços, equipe, configurações, plano | Nuvem |
| Confirmação de Pix | Provedor de pagamento (via Nuvem) |

## 3. Princípios de offline e sincronização

1. **IDs gerados na origem** (UUIDv7). Nenhuma criação depende da nuvem.
2. **Event log / outbox**: toda alteração operacional gera um evento imutável, persistido localmente antes de ser confirmado ao usuário.
3. **Sincronização idempotente**: reenviar um evento nunca duplica efeitos.
4. **Ordem causal por entidade**: eventos de uma mesma entidade são aplicados na ordem em que o Hub os aceitou.
5. **Conflitos**: vence o primeiro evento aceito pelo Hub; o segundo é rejeitado com motivo e o aparelho é notificado.
6. **Aparelhos têm fila local**: se o Wi-Fi cair, ações ficam enfileiradas no aparelho e são enviadas ao Hub ao reconectar.
7. **Status de conexão visível** em todos os aparelhos (online / offline / N eventos pendentes).
8. **Nada se perde se o Hub morrer**: cada aparelho guarda seus eventos até a Nuvem confirmar o recebimento; um novo Hub se reconstrói a partir da Nuvem e dos aparelhos.

## 4. Stack e convenções técnicas

- **Linguagem**: TypeScript (strict).
- **Framework**: NestJS, organizado em módulos por domínio (DDD leve: `domain`, `application`, `infrastructure`).
- **Banco**: PostgreSQL (nuvem), SQLite (Hub). Acesso via ORM compatível com ambos; regras de negócio não dependem do banco.
- **Multi-tenant**: todo registro carrega `tenantId`; isolamento obrigatório em toda consulta.
- **Integrações externas atrás de interfaces (ports)**: `PaymentProvider` (Pix), `PrinterDriver` (ESC/POS). Provedores concretos são plugáveis.
- **Feature flags por plano**: módulos habilitados por tenant, validados também no Hub via licença em cache.
- **Testes**: todo critério de aceite de uma spec vira ao menos um teste automatizado.
- **Idioma**: documentação e termos de domínio em português; código em inglês com glossário (ver `regras-de-negocio.md`).

## 5. Fluxo SDD

1. `spec.md` — histórias e critérios de aceite (Dado/Quando/Então). Sem detalhes técnicos.
2. `plan.md` — modelo de dados, endpoints, eventos, decisões técnicas. Referencia IDs de regras.
3. `tasks.md` — tarefas pequenas, cada uma ligada a critérios de aceite.
4. Implementação + testes. PRs referenciam a spec e os IDs das regras atendidas.
