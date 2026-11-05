# agent-ecosystem · demo

**A working example of [agent-ecosystem](https://github.com/l4uran1/agent-ecosystem): three
services in three repos, and one change an AI coding agent gets wrong when it can only
see one of them.**

[![CI](https://github.com/l4uran1/agent-ecosystem-demo/actions/workflows/ci.yml/badge.svg)](https://github.com/l4uran1/agent-ecosystem-demo/actions/workflows/ci.yml)

## The story

Brewly, a fictional online coffee shop, runs on three services, each in its own repo:

```
             POST /orders
                  │
          ┌───────▼───────┐   orders.created    ┌──────────────┐
          │  orders-api   ├────────┬───────────►│   notifier   │  confirmation email
          └───────────────┘        │            └──────────────┘
                                   │            ┌──────────────┐
                                   └───────────►│  invoicing   │  invoice with VAT
                                                └──────────────┘
```

Brewly starts selling to the UK. The ticket says: *"let customers pay in pounds"*.

**An agent working inside `orders-api` alone** adds a `currency` field to the order and to
the event. It compiles, every test in every repo passes, and it ships. Then a customer
pays **£25.00**, receives an email that says **€25.00** and an invoice in euros. Each repo
types the event on its own, so nothing ever failed.

**The same agent, started from this ecosystem repo,** reads the
[system map](docs/system-map.md) and sees that `orders.created` has two consumers. It reads
the [sensitive areas](docs/product/sensitive-areas.md) (invoices, the confirmation email)
and the [constraints](docs/product/constraints.md): the accountant requires the VAT in EUR
on any non-EUR invoice. So its proposal changes all three repos, puts the exchange rate in
the contract so the invoice matches what was charged, and deploys the consumers first.

## See the change

The change `add-order-currency` is in flight, exactly as the workflow leaves it before the
final review:

- **Proposal PR** (`spec: add-order-currency`): [`openspec/changes/add-order-currency/`](https://github.com/l4uran1/agent-ecosystem-demo/pulls)
  on branch `spec/add-order-currency`: proposal, design, delta specs, and tasks tagged
  per repo and ticked as they were done. It also updates the system map.
- **One code PR per repo**, linked to the proposal, built test-first (each `test:` commit
  fails before its `feat:` commit):
  [orders-api](https://github.com/l4uran1/brewly-orders-api/pulls) ·
  [notifier](https://github.com/l4uran1/brewly-notifier/pulls) ·
  [invoicing](https://github.com/l4uran1/brewly-invoicing/pulls)

The branches were created by `./bootstrap.sh branch add-order-currency`, which reads the
`(repo)` tags in the published `tasks.md` and refuses to create anything until the
proposal is published.

## Run it

Requires git, Node 22.18 or later and make. No `npm install` needed to run it.

```bash
git clone https://github.com/l4uran1/agent-ecosystem-demo.git brewly && cd brewly
./bootstrap.sh            # clones the three service repos side by side
make up                   # runs the three services (Ctrl+C stops them)
```

In another terminal:

```bash
make order CURRENCY=GBP   # main branches: currency does not exist yet, the order is in EUR
```

To see the change working, stop `make up`, switch every repo to the change's branches
and try again:

```bash
git switch spec/add-order-currency
for repo in orders-api notifier invoicing; do git -C $repo switch feature/add-order-currency; done
make up                   # other terminal: make order CURRENCY=GBP → £ in the email, GBP invoice with VAT in EUR
```

**Reproduce the bug** an agent working in one repo would ship: keep orders-api on
`feature/add-order-currency` and put the consumers back on main.

```bash
git -C notifier switch main && git -C invoicing switch main
make up                   # make order CURRENCY=GBP → £25.00 charged, €25.00 in the email
```

## What's in here

| | |
| --- | --- |
| [`AGENTS.md`](AGENTS.md) | Shared rules for every agent: workflow, TDD, git |
| [`docs/system-map.md`](docs/system-map.md) | The contract between the three services |
| [`docs/product/`](docs/product) | Sensitive areas, accounting constraints, glossary, decisions |
| [`openspec/`](openspec) | Specs of current behaviour and the in-flight change |
| [`repos.txt`](repos.txt) · [`bootstrap.sh`](bootstrap.sh) | The service repos and the script that clones, updates and branches them |
| `Makefile` | `make up`, `make order` and shortcuts for `bootstrap.sh` |

The template's own documentation is in [`SETUP.md`](SETUP.md) and in the
[template repo](https://github.com/l4uran1/agent-ecosystem).

## How it was built

The ecosystem repo was created from the template and filled in. The services and the
change were written with Claude Code following `AGENTS.md`: proposal first, one failing
test per behaviour, then the code. The queue is a folder of JSON-lines files, so the demo
needs no broker.
