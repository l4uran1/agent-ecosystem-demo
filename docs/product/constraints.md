# External constraints

Limits that do not depend on us. A proposal that clashes with them is not viable, even if
it looks technically right.

## Accounting (Brewly's accountant)

- Brewly is a Spanish company and every order ships from Spain: the VAT rate is the Spanish
  standard rate, 21%, whatever the customer's currency.
- An invoice is issued in the currency the customer paid in.
- When that currency is not EUR, the invoice must also state the VAT amount in EUR, using
  the exchange rate applied when the customer paid.
- Issued invoices are never edited or deleted: mistakes are fixed with a corrective invoice.

## Payments provider

- Brewly can only charge in EUR and GBP.

## Privacy

- Customer emails must never appear in logs outside the demo console output.
