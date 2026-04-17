# Failure Analysis: Document 02

## 1. What Went Wrong
The model interpreted the `tax` field as an empty string `""` instead of `null` when no tax information was present in the document. This violates the `invoice_schema.md` which explicitly states that absent tax values must be represented by `null`. This causes a type validation failure when the output is parsed by a strict downstream system.

## 2. Source Document Text
```text
INVOICE
From: Globex Corporation
Inv#: INV-2321
Date: 2024-02-14

Items:
- 10x Hardware @ 1200.0 USD
- 1x Consulting @ 150.0 USD

Sub: 12150.0
TOTAL: 12150.0 USD
```

## 3. Expected JSON
```json
{
  "vendor": "Globex Corporation",
  "invoice_number": "INV-2321",
  "date": "2024-02-14",
  "due_date": null,
  "currency": "USD",
  "subtotal": 12150.0,
  "tax": null,
  "total": 12150.0,
  "line_items": [
    {
      "description": "Hardware",
      "quantity": 10,
      "unit_price": 1200.0
    },
    {
      "description": "Consulting",
      "quantity": 1,
      "unit_price": 150.0
    }
  ]
}
```

## 4. Actual Model Output
```json
{
  "vendor": "Globex Corporation",
  "invoice_number": "INV-2321",
  "date": "2024-02-14",
  "due_date": null,
  "currency": "USD",
  "subtotal": 12150.0,
  "tax": "",
  "total": 12150.0,
  "line_items": [
    {
      "description": "Hardware",
      "quantity": 10,
      "unit_price": 1200.0
    },
    {
      "description": "Consulting",
      "quantity": 1,
      "unit_price": 150.0
    }
  ]
}
```

## 5. Why It Likely Failed
While the training set included examples with absent taxes (using `null`), Llama 3.2 has a strong pre-trained bias toward using empty strings `""` or `"N/A"` for missing data points in JSON generations. The Loss signal during training was apparently not strong enough to completely suppress the `"N/A"` or `""` behavior entirely, leading to occasional reversions during inference.

## 6. Proposed Fix (Data Revision)
Increase the proportion of "missing field" examples in the training dataset. Currently, only about 20% of the training instances feature a missing tax or due date field. Increasing this to 35-40% of the dataset will provide a stronger, more consistent supervised signal that `null` is the *only* acceptable token for an undefined numerical field.
