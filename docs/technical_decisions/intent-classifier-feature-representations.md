# ADR-001: Intent Classifier Feature Representations

- **Project:** EdoFarm AI
- **Status:** Accepted for baseline experimentation
- **Decision date:** 21 November 2026
- **Revision:** 2 — BPE deferred to future implementation
- **Scope:** Text representation for the first crop-advisory intent classifier

## 1. Context

The current labelled classifier dataset contains 306 underlying questions across eight intent classes. Each underlying question has an English version and a provisional machine-generated Edo (Bini) version, producing 612 language-specific examples.

The original group-aware split is retained:

| Split | Underlying questions | Language-specific examples |
|---|---:|---:|
| Training | 196 | 392 |
| Validation | 49 | 98 |
| Test | 61 | 122 |
| **Total** | **306** | **612** |

English and Edo versions of the same question remain in the same split. The test split is reserved for final evaluation and must not be used for feature selection, tokenizer training, or hyperparameter tuning.

Edo translations have not yet received human language review. Results on Edo examples must therefore be described as results on a provisional machine-translated dataset, not as proof of reliable real-world Bini understanding.

## 2. Decision

Use TF-IDF features with Logistic Regression as the first interpretable baseline. Compare **three** feature representations under **two** training conditions, giving six trained model configurations.

### Feature representations included in the current baseline

| ID | Representation | Configuration | Reason for inclusion |
|---|---|---|---|
| F1 | Word n-grams | Word unigrams and bigrams | Conventional lexical baseline; captures individual terms and short word sequences. |
| F2 | Character n-grams | `char_wb`, character n-grams 2–5 | Captures recurring spelling fragments and subword patterns without requiring a language dictionary or morphological analyser. |
| F3 | Word + character n-grams | Feature union of F1 and F2 | Tests whether lexical and character-level signals complement one another. |

### Training conditions

- **English-only:** Train on the 196 English training questions.
- **Bilingual:** Train on all 392 English and Edo training examples.

Every trained model is evaluated separately on English validation questions and Edo validation questions. This creates **12 validation measurements** (six trained models × two evaluation languages).

The English-only model evaluated on Edo is a cross-language transfer probe. The bilingual model evaluated on Edo measures performance when provisional Edo examples were available during training.

## 3. BPE decision: defer to future implementation

Do **not** include BPE in the current baseline experiment. Defer it to a future implementation phase when the project has access to a substantially larger, quality-controlled Edo (Bini) text corpus and, if available, Edo speakers who can help review language data and conventions.

The current classifier training set is too small to justify training a project-specific BPE vocabulary. Learning subword units from a few hundred short questions could produce a vocabulary dominated by accidental fragments and the particular machine-translation style in the current data. Running BPE now would add complexity without giving a fair test of an Edo-specific tokenizer.

### Future BPE implementation requirements

Before introducing BPE, the project should:

1. Build a larger Edo/Bini text corpus from approved, appropriately sourced material. It may include questions, answers, educational text, and other relevant language material; it need not consist only of labelled intent-classification examples.
2. Obtain speaker input where possible to review spelling conventions, word boundaries, common variants, dialect considerations, and normalization rules. Speaker participation should be recorded accurately; it must not be claimed before it occurs.
3. Preserve source provenance, consent/licensing conditions, and data quality labels for corpus material.
4. Compare a project-trained Edo tokenizer with a suitable existing multilingual tokenizer, where one has adequate Edo coverage.
5. Train and evaluate tokenizer variants using training-corpus data only. Keep validation and test text out of tokenizer vocabulary learning and model fitting.
6. Evaluate tokenization quality and downstream intent classification using independently authored or speaker-reviewed Edo questions, not only machine-translated questions.
7. Document vocabulary size, normalization, pre-tokenization, special tokens, corpus composition, and tokenizer version so inference uses exactly the same tokenizer.

BPE is a statistical subword method; it does not automatically learn linguistically correct Edo morphemes. Edo speaker input can improve corpus quality and language-convention review, but does not by itself make the resulting tokenizer linguistically valid. Any claimed Edo-specific benefit must be demonstrated empirically.

## 4. Lemmatization decision

Do not apply lemmatization in this baseline.

A standard English lemmatizer is not appropriate for Edo text. An Edo-specific lemmatizer would require a validated morphological analyser, dictionary, or expert-reviewed rules. No such resource has yet been established for this project. Incorrect lemmatization could remove or alter meaningful distinctions.

Lemmatization may be considered in a later experiment if a reliable Edo language resource becomes available and its output can be reviewed. English and Edo preprocessing should be documented separately rather than applying an English-only tool to both languages.

## 5. Shared classifier and evaluation protocol

Use Logistic Regression with:

- `class_weight="balanced"`
- `C=1.0`
- `max_iter=3000`
- fixed `random_state=42`

Use the same classifier settings across all feature configurations to keep the initial comparison controlled. Fit TF-IDF vocabularies and inverse-document-frequency weights on the relevant training subset only. Transform validation examples using the fitted training representation.

Primary comparison metric: **macro F1**, because it weights the eight intent classes equally. Also report accuracy, macro precision, macro recall, and weighted F1. Report per-class results for selected candidates.

The validation set may be used to compare feature representations and select a candidate. The test set remains untouched until the feature configuration and model-selection decision are recorded.

## 6. Reproducibility and leakage controls

1. Reuse the existing group-aware train, validation, and test files; do not create a new split.
2. Keep English and Edo versions of each underlying question in the same split.
3. Train vectorizers and IDF weights using the relevant training subset only.
4. Do not include validation or test text in any future BPE tokenizer training corpus.
5. Do not use answers as intent-classifier input; the classifier predicts intent from the question.
6. Keep the 306 underlying questions distinct from the 612 language-specific examples when reporting dataset size.
7. Do not describe provisional machine translations as human-reviewed or native-speaker-validated.
8. Do not use validation or test performance as evidence of agronomic correctness.

## 7. Current status and next steps

- Dataset preparation: completed; 306 underlying questions and 612 language-specific examples.
- Group-aware split validation: passed.
- Current feature experiments: word n-grams, character n-grams, and combined word + character n-grams.
- Training strategies: English-only and bilingual.
- Model comparison: pending validation results.
- Final model selection: pending.
- Test-set evaluation: pending and must happen only after model selection.
- Human review of Edo translations: pending.
- BPE tokenizer: deferred to a future phase pending a larger quality-controlled Edo corpus and possible speaker input.
- Edo-specific lemmatization: deferred until a reliable resource is identified.

## 8. Interpretation warning

This is a small, preliminary prototype dataset. The experiment compares feature representations under the current data conditions; it does not establish generalization to all Edo speakers, dialect variation, spelling conventions, or field conditions. A strong score on machine-generated translations may reflect patterns in the translation output rather than natural Edo usage. Human review and testing on independently authored Edo questions are needed before making stronger language-performance claims.
