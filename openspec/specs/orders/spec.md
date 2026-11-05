# orders Specification

## Purpose

How customers place orders through orders-api, and the `orders.created` event that tells
the rest of the system about them.

## Requirements

### Requirement: Create an order

orders-api SHALL create an order from a customer email and one or more items, each with a
SKU, a positive integer quantity and a unit price in integer cents with VAT included. The
order total SHALL be the sum of quantity times unit price.

#### Scenario: Valid order

- **WHEN** a client sends `POST /orders` with a valid email and two units of a 1250-cent item
- **THEN** the response is `201` with the order and a total of 2500 cents

#### Scenario: Invalid order

- **WHEN** a client sends `POST /orders` with an invalid email or no items
- **THEN** the response is `400` with the reason and no order is created

### Requirement: Publish orders.created

orders-api SHALL publish `orders.created` once for every order it creates, with the
payload `{orderId, customerEmail, totalCents, createdAt}`.

#### Scenario: Event after a new order

- **WHEN** an order is created
- **THEN** `orders.created` is published with its id, the customer's email, the total in cents and the creation time

#### Scenario: No event for a rejected order

- **WHEN** `POST /orders` is rejected
- **THEN** nothing is published
