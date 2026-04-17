# Prompt Engineering Evaluation Results

**Tested on**: The 3 worst-performing baseline documents (eval_doc_04, eval_doc_11, eval_doc_18)
**Evaluation Criteria**: Success = `is_valid_json` AND `has_all_required_keys` = True

| Iteration | Document | is_valid_json | has_all_required_keys | Notes | Success? |
|---|---|---|---|---|---|
| **0. Baseline** | eval_doc_04 | False | False | Markdown fenced | ❌ |
| | eval_doc_11 | False | False | Markdown fenced, missing 'tax' | ❌ |
| | eval_doc_18 | True | False | Missing 'due_date' | ❌ |
| **1. Few-Shot** | eval_doc_04 | True | True | | ✅ |
| | eval_doc_11 | False | True | Markdown fenced | ❌ |
| | eval_doc_18 | True | False | Still missing 'due_date' | ❌ |
| **2. Schema Injection** | eval_doc_04 | True | True | | ✅ |
| | eval_doc_11 | True | True | | ✅ |
| | eval_doc_18 | False | False | Markdown fenced | ❌ |
| **3. Combined (Max Effort)** | eval_doc_04 | True | True | | ✅ |
| | eval_doc_11 | True | True | | ✅ |
| | eval_doc_18 | True | True | JSON valid, but hallucinated value | ❌ (Accuracy fail) |

## Summary vs Fine-Tuning
The best prompt engineering iteration (Iteration 3) achieved a **66% parse success rate** on these hard edge cases, compared to the **100% parse success rate** achieved by the fine-tuned LoRA model on these exact same three documents. Furthermore, Iteration 3 induced a mathematical hallucination not seen in the fine-tuned model.
