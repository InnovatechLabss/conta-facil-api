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

| # | Spec | Epic Jira | Status |
|---|------|-----------|--------|
| 001 | Turno, mesas e comandas | CF-5 | Rascunho |
| 002 | Cardápio | CF-6 | Rascunho |
| 003 | Pedidos e impressão (ticket de produção) | CF-7 | A fazer |
| 004 | Fechamento, pagamento e Pix | CF-8 | A fazer |
| 005 | Hub Local, modo offline e sincronização | CF-9 | A fazer |
| 006 | KDS (Cozinha / Bar) | CF-10 | A fazer |
| 007 | Planos, módulos, licenciamento e white label | CF-11 | A fazer |
| 008 | Estabelecimento, equipe e permissões | CF-12 | A fazer |

## Gestão no Jira

Projeto **CF** em [innovatechlabs.atlassian.net](https://innovatechlabs.atlassian.net). A spec no repositório é a fonte da verdade; o Jira acompanha a execução.

| SDD (repositório) | Jira |
|---|---|
| Spec `NNN-...` | **Epic** (label `spec-NNN`) |
| História (H1, H2...) com critérios de aceite | **História** filha do Epic |
| Itens do `tasks.md` | **Subtarefas** da História |

- Branches: `CF-<n>-descricao-curta` (ex.: `CF-13-abrir-turno`)
- Commits e PRs citam a chave da issue (ex.: `CF-13: abre turno com fundo de troco`)

## Stack

Node.js + NestJS (TypeScript) · PostgreSQL (nuvem) · SQLite (Hub Local). Detalhes em [`specs/constitution.md`](specs/constitution.md).
