# Model Comparison Report — Week 4

**Name:** Andrew Skoblov
**Date:** March 22, 2026
**Capstone Project:** Financial Fraud Detection System
**My Component:** Case Management & Dashboard

## Test Setup

**Input dataset:** 5 fraud investigation text samples covering:
- 2 clearly concerning/high-severity records (Records 1, 4)
- 1 ambiguous/edge case record (Record 3)
- 2 routine/benign records (Records 2, 5)

**Models tested:**
1. distilbert-base-uncased-finetuned-sst-2-english (sentiment)
2. facebook/bart-large-mnli (zero-shot classification)
3. dslim/bert-large-NER (named entity recognition)
4. Groq Llama 3.1 8B Instant (LLM classification)

**Evaluation criteria:** label accuracy, confidence score, speed, ease of integration in n8n

## Results Summary

| Record | Input (abbreviated) | Sentiment | Zero-Shot | NER Entities | Groq |
|--------|-------------------|-----------|-----------|--------------|------|
| 1 | Unauthorized login from IP... | NEGATIVE (0.9994) | possible anomaly (0.90) | {} | HIGH: Multiple failed login attempts detected |
| 2 | Routine firewall rule update... | NEGATIVE (0.9986) | routine activity (1.00) | {} | INFORMATIONAL: This is a scheduled maintenance event |
| 3 | Phishing email detected... | NEGATIVE (0.9968) | possible anomaly (0.80) | {} | HIGH: This phishing attempt targets sensitive systems |
| 4 | Multiple failed SSH attempts... | NEGATIVE (0.9993) | possible anomaly (0.90) | {"entity_group":"MISC",...} | CRITICAL: Multiple failed SSH attempts indicate active attack |
| 5 | System resource utilization normal... | NEGATIVE (0.9880) | routine activity (1.00) | {} | INFORMATIONAL: This status report shows normal operations |

## Analysis

**Where models agreed:** Records 2 and 5 were both correctly identified as routine/benign by Zero-Shot (scoring 1.00 confidence) and by Groq (INFORMATIONAL). These two models aligned well on the clearly non-threatening inputs. Record 4 was correctly flagged as the most severe by both Zero-Shot ("possible anomaly") and Groq ("CRITICAL").

**Where models disagreed:** The Sentiment model labeled every single record as NEGATIVE with very high confidence (0.988–0.999), including the two routine records (2 and 5). This is a significant failure — it cannot distinguish between a benign status report and an active attack. Zero-Shot and Groq disagreed on severity for Record 1: Zero-Shot called it a "possible anomaly" while Groq classified it HIGH, suggesting the LLM captures threat context better than the classification model.

**Most accurate model overall:** Groq Llama 3.1 8B was the most accurate. It correctly assigned CRITICAL to Record 4, HIGH to Records 1 and 3, and INFORMATIONAL to Records 2 and 5 — a sensible and actionable severity ladder that maps directly to a fraud investigation queue.

**Fastest/most practical:** Zero-Shot (bart-large-mnli) was fast and reliable for bulk classification, though Groq was only marginally slower and produced far more useful output for case prioritization.

## Recommended Models for My Capstone Component

**Component:** Case Management & Dashboard

**Primary model:** Groq Llama 3.1 8B — it produces structured severity labels (CRITICAL / HIGH / INFORMATIONAL) with a one-sentence explanation that can be displayed directly in the Streamlit dashboard alert feed, giving analysts immediate context without clicking into a case.

**Secondary model:** facebook/bart-large-mnli (Zero-Shot) — useful as a lightweight pre-filter to bucket transactions into "fraudulent activity / routine transaction / requires review" before the full Groq call, reducing API usage at scale.

**Rejected models and why:**
- distilbert-sst-2 (Sentiment): Labeled all 5 records NEGATIVE regardless of actual risk level, making it useless for prioritizing an investigation queue. A dashboard sorted by sentiment score would surface benign records alongside critical threats.
- dslim/bert-large-NER: Returned empty entity arrays for 4 out of 5 records. While it could extract named entities from richer transaction narratives, the short alert-style inputs used in this system do not contain enough proper nouns to make NER useful at this stage.

## Failure Cases and Limitations

The most surprising failure was the Sentiment model classifying Record 5 — "System resource utilization normal across all monitored hosts — no anomalies detected" — as NEGATIVE with 0.9880 confidence. This is a completely benign status message. The model appears to be reacting to domain-specific vocabulary like "anomalies" and "monitored" as negative signals, even though the sentence explicitly states everything is normal. In production, using sentiment as a risk signal for fraud alerts would generate a constant stream of false positives, overwhelming analysts and undermining trust in the dashboard. This demonstrates that general-purpose sentiment models trained on review or social media data are not appropriate for technical security and fraud text without domain-specific fine-tuning.

## Reflection

The biggest surprise was how much better Groq performed compared to the Hugging Face models for this specific use case. I expected Zero-Shot to be the most flexible model since you can define your own labels, but Groq's free-form reasoning gave more actionable output — not just a label but a sentence explaining *why*, which is exactly what a fraud analyst reviewing a case queue needs. The NER results were also surprising in how empty they were; I expected it to pick up IP addresses or organization names, but the model only flagged one MISC entity across all five records. It made me realize how important it is to actually test models on your real data rather than assuming benchmark performance translates to your domain.

## Next Steps

With more time, I would test the workflow against the full 50-transaction dataset planned for the capstone and measure what percentage of known fraudulent transactions each model flags correctly. I would also experiment with more specific zero-shot labels like "wire fraud," "account takeover," and "money mule" to see if narrower categories improve precision. A 5th model worth testing is `unitary/toxic-bert` — not for toxicity per se, but to see if hostility signals in transaction descriptions correlate with fraud risk in any meaningful way.
