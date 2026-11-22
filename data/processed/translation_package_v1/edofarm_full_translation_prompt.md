# EdoFarm AI — Full-Dataset English to Edo (Bini) Machine Translation Prompt

You are preparing provisional machine-generated translation drafts for EdoFarm AI, a crop-production advisory chatbot project at the University of Benin, Nigeria.

You are given two CSV files:

1. `edofarm_full_translation_input.csv` — the complete source dataset. This is the source of truth for English text and identifiers.
2. `edofarm_translation_output_template.csv` — the required output structure. Fill this template and return it as a downloadable CSV file.

## 1. Task

Translate all 306 records from English into **Edo (Bini)**. Do not substitute Esan, Urhobo, Nigerian Pidgin, or another language. The target is specifically Edo/Bini as used by the Edo people of Benin City and surrounding Bini-speaking communities.

This is a translation task only. Do not generate new agricultural advice, improve the source answer, fact-check it by changing it, or silently rewrite the meaning.

## 2. How to handle each record type

### `bilingual_qa` (231 records)

Translate both fields:
- `english_question` → `edo_question_machine`
- `english_answer` → `edo_answer_machine`

Keep the full meaning and all relevant details. Do not summarize long answers simply to save space.

### `question_only_classifier` (75 records)

Translate only:
- `english_question` → `edo_question_machine`

Leave `edo_answer_machine` empty and set `machine_answer_status` to `not_applicable_classifier_only`. Do not create or translate an answer for these records. They are out-of-scope classifier examples, not crop-advisory content.

## 3. Edo/Bini translation requirements

- Use natural, comprehensible Edo/Bini rather than word-for-word English syntax where that would sound unnatural.
- Preserve the meaning, intent, politeness, and level of certainty of the original.
- Do not invent Edo vocabulary or claim a technical term has a standard Edo equivalent if uncertain.
- When an agricultural or scientific term has no confidently known equivalent, retain the English technical term where appropriate and explain the uncertainty briefly in `translation_notes`.
- Preserve names of crops, pests, diseases, varieties, places, products, and people accurately. Do not translate proper names.
- Preserve every number, quantity, percentage, dosage, interval, measurement, unit, date, and range exactly. Do not convert units.
- Preserve negations, warnings, restrictions, conditional language, and uncertainty. For example, do not turn “may”, “might”, “could”, or “consult an expert” into a definite instruction.
- Preserve safety warnings and pre-harvest or application intervals exactly as written. Do not add, remove, or alter them.
- Do not add recommendations or contextual statements that are absent from the English source.
- Do not omit sections, numbered steps, or material details from a long answer.
- Keep markdown-like lists readable in the translated text where present.

## 4. Be honest about language uncertainty

Edo/Bini is a lower-resource language for many AI systems. Do not fabricate fluent-looking text when you are unsure.

For each question:
- Use `machine_draft_generated` if you can provide a meaningful draft.
- Use `translation_uncertain` if important wording, terminology, or meaning could not be translated confidently. Provide the best cautious draft only where possible, and explain the uncertainty in `translation_notes`.
- Use `translation_failed` if you cannot produce a meaningful translation.

For each answer in `bilingual_qa`, use the same status rules independently.

A machine draft is not a human-verified translation. Never describe a machine draft as native-speaker reviewed, human-approved, or validated.

## 5. Output requirements — exact schema

Return a CSV containing exactly these eight columns, in this exact order:

```text
translation_group_id,record_id,translation_unit_type,edo_question_machine,edo_answer_machine,machine_question_status,machine_answer_status,translation_notes
```

Use the supplied output template. Do not add columns or remove columns.

## 6. Data integrity — mandatory

- Return exactly 306 rows, one for each input record.
- Preserve all `translation_group_id` values exactly.
- Preserve all `record_id` values exactly.
- Preserve each `translation_unit_type` exactly.
- Preserve the original row order from the input/template.
- Do not create, delete, duplicate, rename, or reorder records.
- Do not modify the IDs to make them easier to translate.
- Do not return only examples or a sample.
- Ensure all 231 `bilingual_qa` records have a question and answer translation status.
- Ensure all 75 `question_only_classifier` records have an empty answer translation and `machine_answer_status=not_applicable_classifier_only`.
- Use valid CSV quoting for commas, quotation marks, line breaks, and non-ASCII Edo characters. Save as UTF-8 (UTF-8 with BOM is acceptable).
- Do not put Markdown fences, explanations, or commentary inside the CSV.

## 7. Output file

Return the completed file with this exact filename:

`edofarm_machine_translations_v1.csv`

If your environment cannot return a downloadable file, output the CSV in clearly identified chunks without omitting any records, and state the exact IDs in each chunk. Do not claim completion until all 306 records are returned and reconciled.

## 8. Final self-audit before returning

Verify:
1. 306 data rows, plus one header row.
2. 306 unique record IDs.
3. 306 unique translation group IDs.
4. The IDs and order exactly match the input/template.
5. 231 `bilingual_qa` rows have both translation fields populated or an explicit uncertainty/failure status.
6. 75 `question_only_classifier` rows have no answer translation.
7. No English source fields, IDs, or translation types were changed.
8. All statuses use the permitted values.
9. All numeric values, measurements, warnings, and negations were preserved.

Report honestly if any row could not be translated confidently. Do not fabricate missing rows or claim full completion when the checks fail.

Remember: this output is a machine-generated draft. Human Edo/Bini review is still required, and agricultural accuracy remains a separate verification process.
