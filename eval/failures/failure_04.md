# Failure Analysis: Document 04

## 1. What Went Wrong
The model hallucinated a "tax" value of 0.0 instead of `null` when no tax was specified. Similarly, it hallucinated the string `"None"` for the `due_date` instead of returning `null`.

## 2. Source Document Text
```text
INVOICE
From: Massive Dynamic
Inv#: INV-102
Date: 2024-01-10
Due: On Receipt

Items:
- 10x Widget @ 10.0 USD

Sub: 100.0
TOTAL: 100.0 USD
```

## 3. Expected JSON
```json
{
  "vendor": "Massive Dynamic",
  "invoice_number": "INV-102",
  "date": "2024-01-10",
  "due_date": null,
  "currency": "USD",
  "subtotal": 100.0,
  "tax": null,
  "total": 100.0,
  "line_items": [
    {
      "description": "Widget",
      "quantity": 10,
      "unit_price": 10.0
    }
  ]
}
```

## 4. Actual Model Output
```json
{
  "vendor": "Massive Dynamic",
  "invoice_number": "INV-102",
  "date": "2024-01-10",
  "due_date": "None",
  "currency": "USD",
  "subtotal": 100.0,
  "tax": 0.0,
  "total": 100.0,
  ...
}
```

## 5. Why It Likely Failed
The prompt text included "Due: On Receipt". The model tried to parse "On Receipt" as a date string, failed, and defaulted to "None" rather than `null`. For the tax, it inferred that since Subtotal equals Total, mathematically Tax must equal 0.0. This shows the LLM doing "smart" reasoning, which violates the strict extraction paradigm.

## 6. Proposed Fix (Data Revision)
Add examples where the "Due" field exists but contains non-date text like "On Receipt", "Immediately", or "Net 30", mapping explicitly to `null` in the target JSON since the schema requires a strict YYYY-MM-DD format. Furthermore, add more examples emphasizing that implicit zero concepts map to `null`, not `0.0`.
