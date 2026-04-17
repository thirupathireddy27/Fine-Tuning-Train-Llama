# Purchase Order JSON Schema Definitions

The expected LLM output for the purchase order (PO) extraction task must strictly adhere to the following structure. Every data instance must be a valid JSON object matching these requirements.

## Schema Specification

- **`buyer`**: (string) The name of the company or individual issuing the purchase order. If illegible or absent, use `""`.
- **`supplier`**: (string) The name of the vendor or supplier fulfilling the order. If absent, use `""`.
- **`po_number`**: (string) The alphanumeric identifier for the purchase order. If absent, use `""`.
- **`date`**: (string) The issue date of the PO, strictly formatted as "YYYY-MM-DD". If absent, use `""`.
- **`delivery_date`**: (string or null) The requested delivery or expected shipping date, strictly formatted as "YYYY-MM-DD". If absent, the value must be `null` (not a string).
- **`currency`**: (string) The 3-letter ISO currency code (e.g., USD, GBP, EUR, INR). If missing but a currency symbol exists, infer it (e.g., £ -> GBP). If entirely absent, use `null`.
- **`total`**: (float) The grand total of the authorized purchase order. Must be a numeric float, not a string (e.g., `1500.00`). If absent, use `null`.
- **`items`**: (array of objects) A list representing the items to be purchased. If none are specified, this must be an empty array `[]`. Each object must contain:
  - **`item_name`**: (string) The name or description of the product/service.
  - **`quantity`**: (int) The requested quantity. If omitted in the document but implied, use `1`.
  - **`unit_price`**: (float) The cost per unit as a numeric float. If omitted or "TBD", use `null`.

### Example Valid JSON
```json
{
  "buyer": "Globex Corporation",
  "supplier": "Stark Industries",
  "po_number": "PO-99120",
  "date": "2024-04-01",
  "delivery_date": "2024-04-10",
  "currency": "USD",
  "total": 55000.00,
  "items": [
    {
      "item_name": "Arc Reactor Core",
      "quantity": 2,
      "unit_price": 25000.00
    },
    {
      "item_name": "Titanium Plating",
      "quantity": 10,
      "unit_price": 500.00
    }
  ]
}
```
