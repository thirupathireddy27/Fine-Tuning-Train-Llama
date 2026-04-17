# Failure Analysis: Document 01

## 1. What Went Wrong
The fine-tuned model completely missed the `po_number` key. The schema explicitly mandates `po_number` (string) for purchase orders. The model failed to include this key in the final JSON, causing parsing validation to fail (`has_all_required_keys = False`).

## 2. Source Document Text
```text
PURCHASE ORDER
Company: Zeta Group
Supplier: Acme Corp
Date: 2023-11-20
Req. Delivery: 2023-11-25

Identifier: PO-98129 (Urgent)

Requisition:
Widget - qty 5 unit 10.5 USD

Grand Total: 52.5 USD
```

## 3. Expected JSON
```json
{
  "buyer": "Zeta Group",
  "supplier": "Acme Corp",
  "po_number": "PO-98129",
  "date": "2023-11-20",
  "delivery_date": "2023-11-25",
  "currency": "USD",
  "total": 52.5,
  "items": [
    {
      "item_name": "Widget",
      "quantity": 5,
      "unit_price": 10.5
    }
  ]
}
```

## 4. Actual Model Output
```json
{
  "buyer": "Zeta Group",
  "supplier": "Acme Corp",
  "date": "2023-11-20",
  "delivery_date": "2023-11-25",
  "currency": "USD",
  "total": 52.5,
  "items": [
    {
      "item_name": "Widget",
      "quantity": 5,
      "unit_price": 10.5
    }
  ]
}
```

## 5. Why It Likely Failed
The model relies on lexical cues like "PO#:", "PO Number:", or "Order No:" to map the PO identifier field. In this document, the PO number was prefixed with the word "Identifier:". This terminology was completely absent from the 80 training examples. As a result, the attention mechanism didn't heavily weight the alphanumeric string next to it, and the model silently omitted the field rather than hallucinating.

## 6. Proposed Fix (Data Revision)
We must expand the training diversity for key-value pair terminologies. 
Add 3-5 new examples to `curated_train.jsonl` where the purchase order number is labeled with unusual terms such as "Identifier", "Reference Number", "Doc#", or "Tracking Code". Fine-tuning failures are data coverage problems; exposing the model to these lexical variants during SFT will correct the mapping behavior.
