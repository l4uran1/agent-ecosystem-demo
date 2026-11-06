## Why

Brewly is starting to sell to customers in the UK, and they want to pay in pounds. Today
every amount in the system is implicitly EUR: orders-api has no currency, and the
confirmation email and the invoice both assume euros. Our payment provider already
supports GBP (docs/product/constraints.md).

## What Changes

- `POST /orders` accepts an optional `currency`: `EUR` (default) or `GBP`. Anything else
  is rejected with `400`.
- orders-api fixes the EUR exchange rate when the order is created, so the amount charged
  and the amounts on the invoice come from the same rate.
- **Contract change:** `orders.created` gains two fields, `currency` and
  `eurExchangeRate`. `totalCents` keeps its name and now means minor units of `currency`
  (pence for GBP). The change is additive: no field is renamed or removed.
- notifier shows the total with the order's currency (`£25.00`, `€25.00`).
- invoicing issues the invoice in the order's currency and, when it is not EUR, also
  states the VAT in EUR, as the accountant requires.
- Consumers treat an event without `currency` as EUR with rate 1: events published before
  this change can still be in the queue.

## Capabilities

### New Capabilities

None.

### Modified Capabilities

- `orders`: orders have a currency and an exchange rate; `orders.created` carries both.
- `order-confirmation`: the total is shown in the order's currency.
- `invoices`: invoices are issued in the order's currency, with the VAT also in EUR when needed.

## Impact

**Affected repos:** orders-api, notifier, invoicing.

**Contracts between services:** `orders.created` (orders-api → notifier, invoicing)
gains `currency` and `eurExchangeRate`. `docs/system-map.md` is updated in this change.
This is exactly the contract listed under "Known risks": each repo types the payload
on its own, so a producer-only change would compile and pass every test while both
consumers silently kept showing and invoicing euros.

**Sensitive areas:** both are touched.
- *Invoices:* a GBP order invoiced in EUR, or without the VAT in EUR, is a wrong legal
  record that needs a corrective invoice. Mitigated by deploying invoicing before
  orders-api, so no GBP order exists before invoicing can handle it.
- *Order confirmation email:* a UK customer who paid £25.00 and reads "€25.00" will think
  they were overcharged. Same mitigation: notifier is deployed before orders-api.

**Deployment order:** invoicing and notifier first (they accept both the old and the new
payload), then orders-api.

**Rollback plan:** revert orders-api first: it stops accepting `currency` and publishes
the old payload, which the new consumers still handle. Then, if needed, revert notifier
and invoicing. GBP invoices already issued stay valid and are not touched; if any of them
is wrong, the accountant issues a corrective invoice.
