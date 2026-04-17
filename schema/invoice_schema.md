# Invoice JSON Schema Definitions

The expected LLM output for the invoice extraction task must strictly adhere to the following structure. Every data instance must be a valid JSON object matching these requirements.

## Schema Specification

- **`vendor`**: (string) The name of the company or individual issuing the invoice. If the vendor name is completely illegible or absent, this should be an empty string `""`.
- **`invoice_number`**: (string) The strict alphanumeric identifier for the invoice. If absent, use `""`.
- **`date`**: (string) The date the invoice was issued, strictly formatted as "YYYY-MM-DD". If absent, use `""`.
- **`due_date`**: (string or null) The date the payment is due, strictly formatted as "YYYY-MM-DD". If the document contains no due date, the value must be `null` (not a string).
- **`currency`**: (string) The 3-letter ISO currency code (e.g., USD, GBP, EUR, INR). Fall back to "USD" if the `$` symbol is used, or `null` if entirely indiscernible.
- **`subtotal`**: (float) The total amount before taxes and discounts. Must be a numeric float, not a string (e.g., `100.50`). If absent, use `null`.
- **`tax`**: (float or null) The tax amount applied. Must be a numeric float. If no tax is mentioned, the value must be `null`.
- **`total`**: (float) The final grand total of the invoice. Must be a numeric float. If absent or illegible, use `null`. 
- **`line_items`**: (array of objects) A strictly ordered array representing each billable item. If no items exist, this must be an empty array `[]`. Each object must contain:
  - **`description`**: (string) Details of the item.
  - **`quantity`**: (int) The number of units. Use `1` as a default if a plain description implies a single unit without explicitly stating the number.
  - **`unit_price`**: (float) The cost per unit as a numeric float.

### Example Valid JSON
```json
{
  "vendor": "Acme Widgets",
  "invoice_number": "INV-10023",
  "date": "2024-03-15",
  "due_date": null,
  "currency": "USD",
  "subtotal": 150.00,
  "tax": 15.00,
  "total": 165.00,
  "line_items": [
    {
      "description": "Widget A",
      "quantity": 10,
      "unit_price": 10.00
    },
    {
      "description": "Widget B",
      "quantity": 5,
      "unit_price": 10.00
    }
  ]
}
```
