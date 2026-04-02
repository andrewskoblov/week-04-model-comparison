# Week 4: Model Comparison

Tested 4 AI models on 5 fraud investigation text samples to evaluate their suitability for the **Case Management & Dashboard** component of our **Financial Fraud Detection System**.

## Models Tested

- HF Sentiment (distilbert-sst-2)
- HF Zero-Shot (bart-large-mnli)
- HF NER (bert-large-NER)
- Groq Llama 3.1 8B Instant

## Finding

Recommended **Groq Llama 3.1 8B** for the Case Management & Dashboard component because it produces structured severity labels (CRITICAL / HIGH / INFORMATIONAL) with a plain-English explanation that can be surfaced directly in the analyst alert queue - something no other tested model provides out of the box.

See `report.md` for full analysis and `results/comparison-table.csv` for raw model outputs.
