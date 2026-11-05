## Context

Three services share one event, `orders.created`, and each declares its own TypeScript
type for it. Amounts are integer cents with VAT included. Every order ships from Spain, so
the VAT rate is 21% regardless of currency (docs/product/constraints.md), and the
accountant requires the VAT in EUR on any invoice issued in another currency.

## Goals / Non-Goals

**Goals:**
- Customers can pay in EUR or GBP, and every customer-facing amount shows the currency
  they paid in.
- Invoices meet the accountant's rule for non-EUR currencies.
- Deploying the services in any order never produces a wrong email or invoice.

**Non-Goals:**
- Live exchange rates. A fixed table in orders-api is enough until we sell in volume.
- Prices per currency in the catalogue: the client sends the unit prices, as today.
- Any tax rule other than Spanish VAT at 21%.

## Decisions

**1. Additive contract, `totalCents` keeps its name.** Renaming it to something like
`totalMinor` would be clearer but breaks both consumers on deploy. Instead we add
`currency` and `eurExchangeRate` and redefine `totalCents` as minor units of `currency`.
For EUR orders nothing changes.

**2. The exchange rate travels in the event.** invoicing needs the rate used at checkout
to state the VAT in EUR. Alternatives considered:
- *invoicing looks up its own rate:* the invoice could disagree with what was charged.
- *invoicing calls orders-api:* adds a synchronous dependency where today there is none.
Fixing it in orders-api and sending it in the event keeps the services decoupled and
the numbers consistent. `eurExchangeRate` is EUR per one unit of `currency`; `1` for EUR.

**3. Missing fields mean EUR.** Old events can be replayed (at-least-once). Consumers
read `currency ?? "EUR"` and `eurExchangeRate ?? 1`.

**4. Rounding.** VAT is still split in the order's currency and rounded once to the cent.
The VAT in EUR is `round(vatCents × eurExchangeRate)`, rounded once more. Net plus VAT
equals the total in the order's currency; the EUR figure is informative, as required.

## Risks / Trade-offs

- [The static GBP rate drifts from the real one] → acceptable for the pilot; the rate is
  in one constant in orders-api and is stamped on each order, so changing it never alters
  past invoices.
- [A consumer is deployed after orders-api] → it would show or invoice GBP orders as EUR.
  Mitigated by the deployment order in the proposal.

## Migration Plan

1. Merge and deploy invoicing, then notifier (both backward compatible).
2. Merge and deploy orders-api.
3. Rollback: in the reverse order (see proposal).

## Open Questions

- Should the GBP rate come from the payment provider instead of a constant? Left for a
  follow-up once we know the provider's API (see "Decisions for the reviewer" in the PR).
