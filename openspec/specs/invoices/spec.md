# invoices Specification

## Purpose

The invoice invoicing issues for every order: its number, amounts and VAT.

## Requirements

### Requirement: Issue one invoice per order

invoicing SHALL issue exactly one invoice for each `orders.created` event, numbered
`INV-<year>-<sequence>` with a four-digit consecutive sequence, and SHALL NOT issue a
second invoice if the same event is delivered again.

#### Scenario: Consecutive numbers

- **WHEN** two different orders are created in 2026
- **THEN** their invoices are numbered INV-2026-0001 and INV-2026-0002

#### Scenario: Duplicate delivery

- **WHEN** invoicing receives the same `orders.created` event twice
- **THEN** only one invoice exists for that order

### Requirement: VAT breakdown

The invoice SHALL be issued in EUR and SHALL split the VAT-inclusive total into net
amount and 21% VAT, rounding the VAT to the cent so that net plus VAT equals the total.

#### Scenario: Exact split

- **WHEN** the order total is 12100 cents
- **THEN** the invoice shows net 10000, VAT 2100 and total 12100, in EUR

#### Scenario: Rounding

- **WHEN** the order total is 2999 cents
- **THEN** the VAT is 520 cents and the net 2479 cents
