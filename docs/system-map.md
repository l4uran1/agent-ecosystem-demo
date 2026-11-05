# System map

Only contracts between services go here: what one service expects from another.
Anything the agent can read inside a single repo (folders, stack, internals) does not.

Keep it true: a PR that adds, changes or removes a contract updates this file in the
same change. An outdated map is worse than none, because the agent trusts it.

Last reviewed: 2026-11-05

## Services

| Service | What it does | Owner |
| --- | --- | --- |
| orders-api | Public HTTP API: creates and returns orders | Checkout team |
| notifier | Sends customer emails triggered by events | Checkout team |
| invoicing | Issues invoices for every order | Finance team |

## Synchronous calls (HTTP, gRPC)

None between services: they only talk through the queue.

## Queues and events

| Producer | Queue / topic | Consumer | Payload | Notes |
| --- | --- | --- | --- | --- |
| orders-api | `orders.created` | notifier, invoicing | `{orderId, customerEmail, totalCents, createdAt}` | At-least-once: consumers must be idempotent (key: `orderId`). Amounts are integer cents, VAT included, always EUR today. Each consumer declares its own copy of the type: a change here is never caught by the compiler |

## Pub/sub channels

None.

## Shared tables

None: each service keeps its own state.

## Shared infrastructure

| Component | Used by | Notes |
| --- | --- | --- |
| Local queue folder (`BREWLY_QUEUE_DIR`, default `/tmp/brewly-queue`) | orders-api, notifier, invoicing | Demo stand-in for a broker: one JSON line per message, one offset file per consumer |
