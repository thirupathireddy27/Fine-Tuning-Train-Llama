# Failure Analysis: Document 03

## 1. What Went Wrong
The model extracted the correct line item descriptions and prices but combined two identical line items into a single entry with `quantity: 2`, instead of keeping them as two separate `quantity: 1` objects as written in the raw document layout. While semantically identical in value, it fails strict structural extraction checks.

## 2. Source Document Text
```text
INVOICE
From: Initech
Inv#: INV-0092
Date: 2024-03-01

Items:
- 1x Software License @ 500.0 USD
- 1x Software License @ 500.0 USD
- 2x Consult @ 100.0 USD

Sub: 1200.0
TOTAL: 1200.0 USD
```

## 3. Expected JSON
```json
{
  "vendor": "Initech",
  "invoice_number": "INV-0092",
  "date": "2024-03-01",
  "due_date": null,
  "currency": "USD",
  "subtotal": 1200.0,
  "tax": null,
  "total": 1200.0,
  "line_items": [
    {
      "description": "Software License",
      "quantity": 1,
      "unit_price": 500.0
    },
    {
      "description": "Software License",
      "quantity": 1,
      "unit_price": 500.0
    },
    {
      "description": "Consult",
      "quantity": 2,
      "unit_price": 100.0
    }
  ]
}
```

## 4. Actual Model Output
```json
{
  ...
  "line_items": [
    {
      "description": "Software License",
      "quantity": 2,
      "unit_price": 500.0
    },
    {
      "description": "Consult",
      "quantity": 2,
      "unit_price": 100.0
    }
  ]
}
```

## 5. Why It Likely Failed
Large Language Models tend to act as summarizers by default. When seeing two visually identical rows, the model's pre-trained logic optimization kicks in, internally "compressing" the data into a single, merged JSON object. The fine-tuning dataset did not contain any examples of invoices with identical, non-aggregated line items, so the model had no penalty applied to this summarization behavior during SFT.

## 6. Proposed Fix (Data Revision)
Introduce deliberately repetitive line items into the `curated_train.jsonl` dataset. By including 4-5 examples where exact duplicate rows appear in the input and are mapped strictly 1:1 as separate objects in the target JSON, the model will learn that its task is verbatim extraction rather than intelligent summarization.
