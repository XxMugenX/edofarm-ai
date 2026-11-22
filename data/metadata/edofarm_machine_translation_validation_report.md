# EdoFarm AI — Machine Translation Validation Report

**Input:** `edofarm_machine_translations_v1.csv`  
**Reference:** `edofarm_full_translation_input.csv` and `edofarm_translation_queue_v1.csv`  
**Records expected:** 306  
**Records received:** 306

## 1. Structural checks

| Check | Result |
|---|---|
| Input translation rows | 306 |
| Machine output rows | 306 |
| Translation queue rows | 306 |
| Unique record IDs in machine output | 306 |
| Record IDs and original row order match | PASS |
| Translation group IDs match input | PASS |
| Translation unit types match input | PASS |
| Missing machine question drafts | 0 |
| Missing machine answer drafts among Q&A rows | 0 |
| Unexpected answers among classifier-only rows | 0 |
| Human translation columns overwritten | NO |

## 2. Translation unit counts

- Agricultural Q&A records: 231
- Out-of-scope question-only records: 75
- Total: 306

## 3. Machine translation status

Question status:
- `machine_draft_generated`: 306

Answer status:
- `machine_draft_generated`: 230
- `translation_uncertain`: 1
- `not_applicable_classifier_only`: 75

## 4. Selection behavior

Question final-source selection:
- `machine`: 306

Answer final-source selection:
- `machine`: 230
- `missing`: 1
- `not_applicable_classifier_only`: 75

Human translation fields remain blank and human review statuses remain `not_started`. All machine drafts are marked `pending_human_review`; meaning preservation remains `not_assessed`.

## 5. Specific issue found

- `AQA_00689`: the machine answer is exactly identical to the English source answer. It was retained in the machine column for traceability, but marked `translation_uncertain` and excluded from the selected final Edo answer. A human translation is required for this answer.

## 6. Language-quality limitation

These checks establish file integrity, completeness and status consistency. They do **not** establish that the generated text is natural, idiomatic or semantically correct Edo (Bini). The machine drafts contain some English technical terms, crop names and framework terminology. Their suitability must be checked by a competent Edo (Bini) speaker, especially for agricultural vocabulary, measurements, instructions and negation.

No translation is marked human-approved or native-speaker-verified.

## 7. Output

Merged translation queue: `edofarm_translation_queue_v2_machine_filled.csv`

This file preserves the original queue fields, adds machine translation statuses, fills machine columns, uses the machine as a provisional fallback where a draft is available, and leaves human fields ready for later entry. Human-approved translations will override machine fallbacks when the selection function is rerun.
