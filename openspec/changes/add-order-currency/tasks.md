## 1. invoicing (deployed first)

- [x] 1.1 (invoicing) Failing test: an event without `currency` still gets an EUR invoice, exactly as today; then make the event type accept the new optional fields
- [x] 1.2 (invoicing) Failing test: a GBP event gets a GBP invoice with net and 21% VAT in GBP; then implement
- [x] 1.3 (invoicing) Failing test: a non-EUR invoice also states the VAT in EUR using the event's exchange rate, and an EUR invoice does not; then implement and print it
- [x] 1.4 (invoicing) Run tests, lint and build

## 2. notifier (deployed second)

- [x] 2.1 (notifier) Failing test: `formatMoney` uses the currency's symbol (€, £); then implement
- [x] 2.2 (notifier) Failing test: the email shows the total in the event's currency, and in EUR when the event has none; then implement
- [x] 2.3 (notifier) Run tests, lint and build

## 3. orders-api (deployed last)

- [ ] 3.1 (orders-api) Failing test: `POST /orders` accepts `currency` EUR or GBP, defaults to EUR and rejects anything else with 400; then implement
- [ ] 3.2 (orders-api) Failing test: the order stores the EUR exchange rate at creation (1 for EUR); then implement
- [ ] 3.3 (orders-api) Failing test: `orders.created` carries `currency` and `eurExchangeRate`; then implement
- [ ] 3.4 (orders-api) Run tests, lint and build
