# Final Report: Prompting vs. Fine-Tuning for Structured Output

## Core Problem
Getting an LLM to accurately read a text document is relatively easy. Getting it to emit the extraction as rigorously structured JSON, consistently, without preamble or markdown fencing, is surprisingly difficult. This project explored forcing a Llama 3.2 3B model to output strictly formatted JSON for Invoice and Purchase Order extraction.

## Prompting vs. Fine-Tuning Strategy

Through the implementation of the baseline and prompt engineering experiments, it became clear that while few-shot prompting and schema injection (Prompt Iteration 3) drastically improved JSON adherence compared to a zero-shot baseline, they failed to achieve the five-nines reliability required for production automation. 

**Prompt Engineering (The Limitations):**
As prompt complexity grows—adding few-shot examples, JSON schemas, and negative constraints ("DO NOT use markdown")—the prompt context window is consumed by instructions rather than the document itself. The model's attention dilutes. In our case study, maximum-effort prompting on a complex document successfully eliminated markdown fences but resulted in the model hallucinating a line-item value. The cognitive load of balancing formatting restrictions with data extraction was too high for a 3B model.

**Parameter-Efficient Fine-Tuning (The Solution):**
The Supervised Fine-Tuning approach utilizing LoRA (Rank=16, Epochs=3) fundamentally altered the model's default behavior pattern. By training on 80 highly curated input-output JSONL pairs, the instruction to output JSON stopped being something the model needed to be reminded of in the prompt; it became the structural default. 

Our baseline model achieved an ~80% parse success rate (failing largely due to conversational preambles and markdown fencing). The fine-tuned LoRA model reached a **~95% parse success rate**, with complete elimination of conversational text. The few remaining failures were not structural, but rather type-matching edge cases (e.g., using `""` instead of `null` for missing fields).

## Conclusion
For production data pipelines where a single malformed JSON response breaks a downstream system, **fine-tuning is strictly superior to prompt engineering**. Prompting is ideal for exploratory data extraction or when schemas change rapidly. But when the output schema is fixed (like an ERP ingest pipeline), SFT effectively "bakes in" the structural constraints, freeing up the model's context window entirely for the intellectual task of accurate data extraction.
