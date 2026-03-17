# PARROT-errors

Error injection pipeline for the PARROT multilingual radiology report dataset, as described in:

> **Detecting Errors in Radiology Reports with Large Language Models: A Multilingual Study**
> Le Guellec et al., 2026

This repository contains the scripts used to inject three types of errors into PARROT reports (French, German, Greek, Italian) and their English translations, as well as the resulting annotated dataset.

---

## Pipeline

```
parrot_proofread.jsonl
       │
       │
       ├── introduce_laterality_errors.py  → reports_with_laterality_errors.jsonl
       ├── introduce_negation_errors.py    → reports_with_negation_errors.jsonl
       └── introduce_lexical_errors.py     → reports_dmeta.jsonl
                                                    │
                                                    ▼
                                            merge_reports.py → merged_reports.jsonl
```

---

## Scripts


### `introduce_laterality_errors.py`
Uses `gpt-4o-2024-08-06` to introduce laterality contradictions (e.g., "left" in Findings vs. "right" in Impression) consistently across each original-language report and its English translation.

```bash
python introduce_laterality_errors.py \
  --input parrot_proofread.jsonl \
  --output reports_with_laterality_errors.jsonl \
  --api-key $OPENAI_API_KEY
```

### `introduce_negation_errors.py`
Uses `gpt-4o-2024-08-06` to introduce negation contradictions (e.g., "no evidence of fracture" → "evidence of fracture") consistently across each report pair.

```bash
python introduce_negation_errors.py \
  --input parrot_proofread.jsonl \
  --output reports_with_negation_errors.jsonl \
  --api-key $OPENAI_API_KEY
```

### `introduce_lexical_errors.py`
Double Metaphone-based lexical error injection. Replaces one word per report with a phonetically similar but semantically different word, using language-specific frequency lists (`wordfreq`).

```bash
python introduce_lexical_errors.py \
  --input parrot_proofread.jsonl \
  --report-col report_proofread \
  --lang-col language \
  --translation-col translation_proofread \
  --out reports_dmeta.jsonl \
  --seed 42
```

Dependencies: `pip install Metaphone wordfreq`

### `merge_reports.py`
Merges the three error files into a single dataset, renaming fields for clarity (`report_with_error` → `report_with_laterality_error`, etc.).

```bash
python merge_reports.py
```

---

## Data

| File | Description |
|------|-------------|
| `parrot_proofread.jsonl` | LLM-proofread reports (pipeline input) |
| `reports_with_laterality_errors.jsonl` | Reports with injected laterality errors |
| `reports_with_negation_errors.jsonl` | Reports with injected negation errors |
| `reports_dmeta.jsonl` | Reports with injected lexical errors |

---

## Requirements

```
openai
pandas
metaphone
wordfreq
```

---

## Languages

French, German, Greek, Italian — with paired English translations for all reports.
