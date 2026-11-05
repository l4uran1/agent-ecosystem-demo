## MODIFIED Requirements

### Requirement: VAT breakdown

The invoice SHALL be issued in the order's currency (EUR when the event has none) and
SHALL split the VAT-inclusive total into net amount and 21% VAT, rounding the VAT to the
cent so that net plus VAT equals the total.

#### Scenario: Exact split

- **WHEN** the order total is 12100 cents in EUR
- **THEN** the invoice shows net 10000, VAT 2100 and total 12100, in EUR

#### Scenario: Rounding

- **WHEN** the order total is 2999 cents
- **THEN** the VAT is 520 cents and the net 2479 cents

#### Scenario: Invoice in pounds

- **WHEN** the order total is 12100 pence in GBP
- **THEN** the invoice shows net 10000, VAT 2100 and total 12100, in GBP

#### Scenario: Event published before currencies existed

- **WHEN** the event has no currency
- **THEN** the invoice is issued in EUR, exactly as before

## ADDED Requirements

### Requirement: VAT in EUR for other currencies

An invoice in a currency other than EUR SHALL also state the VAT amount in EUR, computed
with the exchange rate fixed when the order was created and rounded to the cent. An EUR
invoice SHALL NOT carry it.

#### Scenario: GBP invoice

- **WHEN** a GBP order with 2100 pence of VAT was created at 1.17 EUR per GBP
- **THEN** the invoice also states VAT in EUR of 2457 cents and the rate 1.17

#### Scenario: EUR invoice

- **WHEN** an EUR order is invoiced
- **THEN** the invoice has no separate VAT in EUR
