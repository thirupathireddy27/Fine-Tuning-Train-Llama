# Structured Output Fine-Tuning: Llama 3.2

## Overview
This repository contains the full execution, datasets, configuration, and analysis for fine-tuning a Large Language Model (Llama 3.2 3B) to strictly output machine-parseable JSON from unstructured business documents (invoices and purchase orders).

The core thesis of this project is demonstrating that **parameter-efficient fine-tuning (LoRA)** on a highly curated synthetic dataset drastically outperforms traditional **prompt engineering** when rigorous schema adherence and JSON parsing success are required.

## Methodology & Project Structure

The project strictly follows a structured sequence:

1. **Schema Design (`schema/`)**: 
   - We defined rigid JSON structures for `invoice_schema.md` and `po_schema.md` dictating specific field names, data types, and null-handling rules.

2. **Data Curation (`data/`)**: 
   - A dataset of 80 synthetic examples (50 Invoices, 30 POs) was generated representing realistic OCR variations.
   - Outputs conform 100% to the designed schemas.
   - Handled via `curated_train.jsonl` and meticulously documented in `curation_log.md`.

3. **Evaluation (Baselines & Fine-Tuned) (`eval/`)**: 
   - 20 held-out documents were processed zero-shot using the Base Instruct model resulting in frequent Markdown fences and conversational JSON wrapper texts (`baseline_responses.md`).
   - The same documents were re-evaluated after LoRA fine-tuning (`finetuned_responses.md`), demonstrating near-perfect parsing and schema compliance.
   - Includes full tabular CSV scoring matrices and before-vs-after summaries.

4. **Failure Analysis (`eval/failures/`)**: 
   - Deep-dives into 5 specific complex instances where the LLM still failed either semantically or structurally, isolating root causes in the training data's diversity distribution rather than the model itself.

5. **Prompt Engineering vs SFT (`prompts/` and `report.md`)**:
   - Tested robust few-shot and schema-injection prompts on the hardest baseline failures. Demonstrated that while prompt engineering can mitigate errors, it reduces contextual focus compared to Supervised Fine-Tuning which makes formatting a hard constraint natively.

6. **Training Configuration (`training_config.md` & `screenshots/`)**:
   - Justifies all LlamaFactory Hyperparameters (Rank 16, Alpha 32, Learning Rate 2e-4, 3 Epochs).

## Findings
Fine-Tuning effectively forces structural compliance onto LLMs. It elevated the baseline model's Parse Success Rate from ~80% (marred by markdown fencing and conversational padding) to ~95%, demonstrating its necessity in production environments executing automated data extraction pipelines.
