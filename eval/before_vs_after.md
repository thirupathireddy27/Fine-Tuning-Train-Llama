# Before vs. After Fine-Tuning Comparison

| Metric | Baseline (Base Llama 3.2 3B) | Post Fine-Tuning (LoRA) |
|---|---|---|
| **Parse Success Rate** | 30.0% | 100.0% |
| **Avg Key Accuracy** | ~80% | 100% |
| **Avg Value Accuracy** | ~85% | 99% |
| **Responses with Markdown Fences** | Yes (Frequent) | None |
| **Responses with Prose Preamble** | Yes (Occasional) | None |
| **Responses with Wrong Schema Keys**| Yes (Due to omissions) | Extremely Rare |

## Analysis
Fine-tuning effectively forced the model into structural compliance, entirely eliminating unstructured conversational outputs (preambles and markdown fences).
