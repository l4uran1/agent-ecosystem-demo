# order-confirmation Specification

## Purpose

The email notifier sends to confirm an order to the customer.

## Requirements

### Requirement: Send one confirmation per order

notifier SHALL send one confirmation email to the customer for each `orders.created`
event, and SHALL NOT send a second one if the same event is delivered again.

#### Scenario: New order

- **WHEN** notifier receives `orders.created` for an order
- **THEN** it sends an email to the order's customer with the subject "Your Brewly order <short id> is confirmed"

#### Scenario: Duplicate delivery

- **WHEN** notifier receives the same `orders.created` event twice
- **THEN** only one email is sent

### Requirement: Show the order total

The confirmation email SHALL show the order total formatted as euros with two decimals.

#### Scenario: Total in euros

- **WHEN** the order total is 2999 cents
- **THEN** the email shows "Total: €29.99"
