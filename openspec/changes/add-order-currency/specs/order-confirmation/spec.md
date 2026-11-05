## MODIFIED Requirements

### Requirement: Show the order total

The confirmation email SHALL show the order total in the order's currency, with its
symbol and two decimals. An event without a currency SHALL be treated as EUR.

#### Scenario: Total in euros

- **WHEN** the order total is 2999 cents in EUR
- **THEN** the email shows "Total: €29.99"

#### Scenario: Total in pounds

- **WHEN** the order total is 2500 pence in GBP
- **THEN** the email shows "Total: £25.00"

#### Scenario: Event published before currencies existed

- **WHEN** the event has no currency and a total of 2999
- **THEN** the email shows "Total: €29.99"
