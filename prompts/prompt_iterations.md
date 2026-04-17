# Prompt Engineering Iterations

Target: The 3 worst-performing baseline documents (eval_doc_04.txt, eval_doc_11.txt, eval_doc_18.txt).
Model: Llama-3.2-3B-Instruct (Base)

## Baseline Prompt (Iteration 0)
```text
System: You are an extraction assistant. Extract all invoice or purchase order fields and return ONLY a valid JSON object. No explanation, no markdown, no code fences.
User: <document text>
```
**Result**: High failure rate. Model frequently wrapped JSON in ```json blocks, added conversational preambles, and omitted null keys entirely.

## Iteration 1: Few-Shot Prompting
```text
System: Extract all fields and return ONLY a valid JSON object conforming to the schema. No markdown fences.
User: [Example Document 1]
Assistant: {"vendor": "Acme", "tax": null ...}
User: [Example Document 2]
Assistant: {"buyer": "Zeta", "items": [...] ...}
User: <document text>
```
**Result**: Performance improved significantly for schema key adherence. However, the model still occasionally included markdown code fences, particularly on longer documents.

## Iteration 2: Stricter Formatting Constraints with JSON Schema Injection
```text
System: You are a pure data processing API. You must output raw JSON. Do not use markdown formatting. Do not use code blocks. Here is the strict JSON schema you must emit:
{ "vendor": "string", "due_date": "null or string", ... }
Remember: ONLY output the JSON starting with '{' and ending with '}'.
User: <document text>
```
**Result**: Markdown code fences were reduced but not completely eliminated (1/3 documents still had a code fence). The model successfully mapped `null` values much better than the baseline due to the explicit schema injection.

## Iteration 3: Combined Few-Shot + Schema Injection + Negative Guidance
```text
System: Return ONLY raw, valid JSON. 
RULE 1: ABSOLUTELY NO MARKDOWN CODE FENCES (```).
RULE 2: Output must adhere exactly to this schema: [Schema]
RULE 3: If a field is absent, its value MUST be null.
Example 1:
User: ...
Assistant: { ... }
User: <document text>
Assistant: 
```
**Result**: This prompt achieved valid JSON on 2 out of the 3 targeted failures. However, on the most complex document (eval_doc_18.txt), the model hallucinated a line item price to make the math balance because the prompt size grew very large, shifting its attention away from the document text and heavily onto the schema rules.
