## ADDED Requirements

### Requirement: Order currency

orders-api SHALL accept an optional `currency` in `POST /orders`, either `EUR` or `GBP`,
defaulting to `EUR`. Prices and the total are in minor units of that currency. When the
order is created, orders-api SHALL fix its EUR exchange rate (EUR per one unit of the
currency), which is `1` for EUR.

#### Scenario: Order in pounds

- **WHEN** a client sends `POST /orders` with `currency: "GBP"`
- **THEN** the order is created in GBP with the current GBP exchange rate

#### Scenario: Currency not given

- **WHEN** a client sends `POST /orders` without `currency`
- **THEN** the order is created in EUR with an exchange rate of 1

#### Scenario: Unsupported currency

- **WHEN** a client sends `POST /orders` with `currency: "USD"`
- **THEN** the response is `400` and no order is created

## MODIFIED Requirements

### Requirement: Publish orders.created

orders-api SHALL publish `orders.created` once for every order it creates, with the
payload `{orderId, customerEmail, totalCents, currency, eurExchangeRate, createdAt}`,
where `totalCents` is in minor units of `currency`.

#### Scenario: Event after a new order

- **WHEN** an order is created
- **THEN** `orders.created` is published with its id, the customer's email, the total in minor units, the currency, the EUR exchange rate and the creation time

#### Scenario: No event for a rejected order

- **WHEN** `POST /orders` is rejected
- **THEN** nothing is published
