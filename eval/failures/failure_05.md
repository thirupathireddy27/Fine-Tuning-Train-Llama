# Failure Analysis: Document 05

## 1. What Went Wrong
The model mistakenly parsed the date "NOV-15-2023" into the incorrect YYYY-MM-DD string `"2023-11-20"` and returned the `"currency"` as `null` when a £ (Pound sterling) symbol was visually present beside the total amount.

## 2. Source Document Text
```text
PURCHASE ORDER
Buyer: Epsilon Ltd
Supplier: Wayne Ent
Date: NOV-15-2023

Requisition:
Cyber Security Hardware - qty 2 unit 500.0

Grand Total: £1000.00
```

## 3. Expected JSON
```json
{
  "buyer": "Epsilon Ltd",
  "supplier": "Wayne Ent",
  "po_number": "",
  "date": "2023-11-15",
  "delivery_date": null,
  "currency": "GBP",
  "total": 1000.0,
  "items": [
    {
      "item_name": "Cyber Security Hardware",
      "quantity": 2,
      "unit_price": 500.0
    }
  ]
}
```

## 4. Actual Model Output
```json
{
  "buyer": "Epsilon Ltd",
  "supplier": "Wayne Ent",
  "po_number": "",
  "date": "2023-11-20",
  "delivery_date": null,
  "currency": null,
  "total": 1000.0,
  ...
}
```

## 5. Why It Likely Failed
The date hallucination ("20" instead of "15") is a common tokenization artifact in LLMs dealing with alphanumeric strings lacking spaces. The currency failure is a direct result of relying solely on the ISO string (USD, GBP) in the training dataset rather than including examples where the currency must be inferred strictly from symbols like £ or €.

## 6. Proposed Fix (Data Revision)
Incorporate a wider variety of date formatting strings (e.g., "DD/MM/YY", "Mon DD, YYYY", "MM-DD-YYYY") in the training data mapped to the strict "YYYY-MM-DD" JSON output. Additionally, include 5-10 examples where currency is exclusively denoted by symbols ($, £, €, ¥, ₹) so the model learns the necessary mapping to their respective ISO codes (USD, GBP, EUR, JPY, INR).
