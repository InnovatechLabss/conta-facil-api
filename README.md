# Conta Fácil — API

SaaS de cardápio e gestão de mesas para bares e restaurantes, com **comandas individuais por pessoa**: cada cliente da mesa tem a sua própria conta, e o fechamento já sai separado.

## Proposta de valor

- Fim da confusão na divisão da conta: cada item pertence a uma pessoa (ou é dividido entre pessoas específicas).
- **Funciona sem internet**: a operação roda num Hub Local dentro do restaurante e sincroniza com a nuvem quando a conexão volta.
- Pagamento via Pix integrado, impressão em impressoras térmicas e KDS (painel de cozinha/bar) conforme o plano.

## Metodologia: Spec-Driven Development (SDD)

Nenhum código é escrito sem uma spec aprovada. Estrutura:

```
specs/
  constitution.md              ← princípios imutáveis (stack, arquitetura, convenções)
  regras-de-negocio.md         ← catálogo único de regras de negócio (IDs rastreáveis)
  NNN-nome-da-feature/
    spec.md                    ← O QUÊ e POR QUÊ: histórias + critérios de aceite
    plan.md                    ← COMO: modelo de dados, endpoints, eventos
    tasks.md                   ← passos de implementação
```

## Roadmap de specs do MVP

| # | Spec | Status |
|---|------|--------|
| 001 | Turno, mesas e comandas | Rascunho |
| 002 | Cardápio | A fazer |
| 003 | Pedidos e impressão (ticket de produção) | A fazer |
| 004 | Fechamento, pagamento e Pix | A fazer |
| 005 | Hub Local, modo offline e sincronização | A fazer |
| 006 | KDS (Cozinha / Bar) | A fazer |
| 007 | Planos, módulos, licenciamento e white label | A fazer |
| 008 | Estabelecimento, equipe e permissões | A fazer |

## Stack

Node.js + NestJS (TypeScript) · PostgreSQL (nuvem) · SQLite (Hub Local). Detalhes em [`specs/constitution.md`](specs/constitution.md).
