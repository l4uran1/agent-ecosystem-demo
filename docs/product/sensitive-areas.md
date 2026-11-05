# Sensitive areas

A mistake in these areas directly hurts customers or their trust in the product.
**Any change in them requires an OpenSpec proposal**, however small it looks, and the
proposal must explain the risk for users.

## Invoices

- **What it is:** the invoice every customer receives for an order: number, amounts and VAT.
- **Where it lives:** invoicing (all of it); orders-api decides the amounts it receives.
- **Why it is sensitive:** an invoice is a legal record. A wrong amount, currency or VAT
  cannot be edited once issued: it needs a corrective invoice, the accountant's time and an
  apology to the customer.

## Order confirmation email

- **What it is:** the email that confirms what the customer bought and how much they paid.
- **Where it lives:** notifier.
- **Why it is sensitive:** it is the customer's proof of payment until the invoice arrives.
  A wrong total or currency makes customers think they were overcharged and floods support.

## Known risks

- **The `orders.created` payload is typed three times.** orders-api, notifier and
  invoicing each declare their own `OrderCreated` type. Adding or renaming a field compiles
  and passes every repo's tests while the consumers silently ignore it. Any change to the
  payload must list both consumers and update them in the same change.
- **Old events stay in the queue.** Consumers may replay events published before a
  contract change: a new field cannot be assumed present.
