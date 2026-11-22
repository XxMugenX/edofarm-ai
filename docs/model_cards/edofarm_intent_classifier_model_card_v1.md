# EdoFarm AI — Intent Classifier Model Card

**Model version:** 1.0.0  
**Artifact:** `edofarm_intent_classifier_v1.joblib`  
**Status:** Experimental baseline  
**Task:** Agricultural query intent classification  
**Languages evaluated:** English and provisional machine-translated Edo (Bini)  
**Date:** 21 November 2026

---

## 1. Model overview

The EdoFarm AI intent classifier assigns an incoming agricultural question to one of eight intent categories. It is intended to support query routing within the EdoFarm AI crop-production advisory prototype.

The selected model is a bilingual TF-IDF and Logistic Regression pipeline. It combines word-level and character-level TF-IDF features and uses a Logistic Regression classifier with balanced class weights.

The model does **not** generate agricultural answers, diagnose crop diseases, recommend pesticide products or application rates, or establish that an agricultural recommendation is safe or locally correct.

## 2. Intended use

### Intended uses

- Experimental classification of English and Edo (Bini) agricultural questions.
- Routing supported crop-production questions to the relevant advisory workflow.
- Identifying questions provisionally classified as outside the prototype's supported scope.
- Research and evaluation of bilingual intent classification using a small, provisional dataset.

### Out-of-scope uses

- Autonomous agricultural decision-making without appropriate review.
- Definitive crop-disease or pest diagnosis.
- Chemical, pesticide, fertilizer-rate, or treatment prescriptions.
- Treating the `out_of_scope` prediction as a guaranteed safety boundary.
- Claiming reliable understanding of naturally authored Edo/Bini questions based on the current evaluation alone.
- Use as a production-ready model without further data, language, and domain validation.

The prototype's supported subject area is crop-production advisory. Livestock, poultry, veterinary care, aquaculture, and animal husbandry are outside the current project scope.

## 3. Supported labels

The classifier predicts one of the following eight labels:

1. `crop_cultivation`
2. `disease_management`
3. `fertilizer_application`
4. `harvesting`
5. `irrigation`
6. `out_of_scope`
7. `pest_management`
8. `soil_management`

The model always produces a class from this label set. It does not currently provide a separately validated unknown-intent detector or calibrated abstention mechanism.

## 4. Training data

### Sources

The project dataset was assembled from two source datasets:

- FarmerChat Nigeria (`DigiGreen/farmerchat-queries`)
- Agronomy QA (`flamiinngo/agronomy-qa-agriculture`)

Source records were cleaned, filtered, reviewed through the project's dataset-preparation workflow, and assigned provisional intent labels. Source provenance and record identifiers are retained in the project data files.

The Agronomy QA source has regional-context limitations for a Nigeria-focused application. Source answers and intent assignments should not be assumed to be locally validated agricultural guidance.

### Dataset size and split

There are **306 underlying questions** in the accepted classifier dataset. Each question is represented by an English version and a provisional Edo (Bini) machine translation, resulting in 612 language-specific rows.

| Split | Underlying questions | English rows | Edo rows | Total rows |
|---|---:|---:|---:|---:|
| Training | 196 | 196 | 196 | 392 |
| Validation | 49 | 49 | 49 | 98 |
| Test | 61 | 61 | 61 | 122 |
| **Total** | **306** | **306** | **306** | **612** |

Splitting was performed at the underlying record/group level. English and Edo versions of the same question were kept in the same split to prevent language-pair leakage. Duplicate-group checks were also performed.

The 612 rows are **not 612 independent questions**. They represent 306 underlying questions expressed in two languages.

### Training class distribution

The 196 underlying training questions were distributed as follows:

| Intent | Underlying training questions |
|---|---:|
| `crop_cultivation` | 11 |
| `disease_management` | 25 |
| `fertilizer_application` | 26 |
| `harvesting` | 22 |
| `irrigation` | 20 |
| `out_of_scope` | 48 |
| `pest_management` | 16 |
| `soil_management` | 28 |
| **Total** | **196** |

The dataset is small and class-imbalanced. In particular, crop cultivation has only 11 underlying training questions.

## 5. Language and translation considerations

All Edo questions used for this classifier were machine-generated translations of English questions. The classifier was trained and tested on these provisional translations.

As of this model card:

- Human translation review has not been completed.
- Native Edo/Bini speaker validation has not been performed.
- Independently authored natural Edo/Bini agricultural questions have not been included in the evaluation.
- The current results therefore do not establish performance on naturally authored or spoken Edo/Bini queries.

The project intends to involve Edo speakers in future translation review and data collection. An Edo-specific BPE tokenizer and Edo-specific lemmatization are deferred future work, pending sufficient language data and appropriate linguistic review.

## 6. Model architecture and configuration

**Pipeline:**

1. Word-level TF-IDF features (unigrams and bigrams).
2. Character-level TF-IDF features (`char_wb`, n-grams of lengths 2–5).
3. `FeatureUnion` combines both feature sets.
4. Logistic Regression predicts one of the eight intent labels.

**Configuration:**

| Parameter | Value |
|---|---|
| Training strategy | Bilingual (English + provisional Edo) |
| Word analyzer | `word` |
| Word n-gram range | `(1, 2)` |
| Word maximum features | 20,000 |
| Character analyzer | `char_wb` |
| Character n-gram range | `(2, 5)` |
| Character maximum features | 30,000 |
| TF-IDF setting | `sublinear_tf=True` |
| Classifier | `LogisticRegression` |
| Class weights | `balanced` |
| `max_iter` | 3,000 |
| `C` | 1.0 |
| Random state | 42 |

BPE tokenization and lemmatization were not part of this model version.

## 7. Model selection

Six configurations were compared on validation data: three feature representations (word n-grams, character n-grams, and combined word + character features), each trained under English-only and bilingual strategies.

The selected configuration was the bilingual word + character model because it achieved the highest validation macro F1 among the bilingual alternatives on both evaluated languages.

| Training strategy | Feature representation | English validation macro F1 | Edo validation macro F1 |
|---|---|---:|---:|
| English-only | Word n-grams | 0.6949 | 0.4542 |
| English-only | Character n-grams | 0.7979 | 0.5145 |
| English-only | Word + character | 0.8307 | 0.4611 |
| Bilingual | Word n-grams | 0.7331 | 0.7125 |
| Bilingual | Character n-grams | 0.8000 | 0.7224 |
| **Bilingual** | **Word + character** | **0.8187** | **0.7364** |

The final test set was not used for feature selection or hyperparameter tuning.

## 8. Evaluation results

### Final holdout test results

| Metric | English (61 examples) | Edo (61 examples) |
|---|---:|---:|
| Accuracy | 0.7377 | 0.8197 |
| Macro precision | 0.6808 | 0.7488 |
| Macro recall | 0.6956 | 0.7596 |
| **Macro F1** | **0.6687** | **0.7406** |
| Weighted F1 | 0.7121 | 0.8030 |

### Per-intent test F1

| Intent | English F1 | Edo F1 |
|---|---:|---:|
| `crop_cultivation` | 0.0000 | 0.0000 |
| `disease_management` | 0.7500 | 0.8889 |
| `fertilizer_application` | 0.7778 | 0.8000 |
| `harvesting` | 0.9231 | 0.8000 |
| `irrigation` | 0.6000 | 0.7273 |
| `out_of_scope` | 0.6875 | 0.8667 |
| `pest_management` | 0.8333 | 1.0000 |
| `soil_management` | 0.7778 | 0.8421 |

There were only three underlying crop-cultivation test questions per language, and none was correctly classified in either language. The resulting zero F1 is an important failure signal, but the support is too small to treat it as a precise estimate of general performance.

### Validation-to-test comparison

| Metric | English validation | English test | Edo validation | Edo test |
|---|---:|---:|---:|---:|
| Accuracy | 0.8367 | 0.7377 | 0.8163 | 0.8197 |
| Macro F1 | 0.8187 | 0.6687 | 0.7364 | 0.7406 |
| Weighted F1 | 0.8296 | 0.7121 | 0.7896 | 0.8030 |

English test macro F1 is notably lower than English validation macro F1. Edo validation and test macro F1 are similar, but both evaluations use machine-translated questions and do not establish performance on natural Edo/Bini input.

The test set contains 61 underlying questions, not 122 independent questions. The English and Edo rows are paired translations of those questions.

## 9. Error analysis and known limitations

A validation error review identified 10 underlying records with at least one incorrect language-level prediction. The team generated a blind manual-review queue and a separate internal diagnostics file. These records are pending manual review; no automatic relabelling or model retraining was performed from that review queue.

Observed limitations include:

- Poor crop-cultivation performance in the final test set.
- Weaker English test performance than English validation performance.
- Confusion between irrigation, soil management, harvesting, disease management, and out-of-scope questions.
- Small per-class support, making individual class metrics unstable.
- Machine-generated Edo translations may contain unnatural wording, translation errors, or inconsistent terminology.
- No native-speaker assessment or independent natural-Edo test set.
- No agricultural expert validation of the underlying advisory content.
- No probability calibration or validated confidence threshold.
- The classifier's `out_of_scope` label is not a guaranteed safety filter.

## 10. Safety and deployment considerations

This model should be treated as an experimental routing component, not an agricultural authority. Its predicted intent should not be treated as proof that a user question is in scope, safe, or correctly understood.

For deployment in the prototype:

- Keep an uncertainty-aware fallback or clarification path.
- Do not let this classifier independently authorize pesticide, fertilizer, or other high-impact recommendations.
- Route uncertain, ambiguous, or potentially serious crop issues to a safer response or qualified extension support.
- Keep source verification and agricultural safety controls in the response/retrieval workflow.
- Monitor errors by intent and language.
- Do not claim that the model understands natural Edo/Bini until it has been evaluated on independently authored, speaker-reviewed Edo/Bini questions.

## 11. Artifact and reproducibility

The fitted pipeline was exported as a single Joblib artifact containing the fitted TF-IDF feature extractors and Logistic Regression classifier.

**Expected artifact path:**

`models/intent_classifier/v1/edofarm_intent_classifier_v1.joblib`

**Metadata path:**

`models/intent_classifier/v1/model_metadata_v1.json`

The exported artifact was reloaded, and predictions on five sample English test questions matched predictions from the in-memory fitted pipeline.

For reproducible deployment, record and pin the Python, scikit-learn, NumPy, SciPy, and Joblib versions used to train and load the artifact. Joblib files should only be loaded from trusted sources and compatible environments.

## 12. Future work

1. Complete blind review of the ten flagged validation records.
2. Obtain Edo/Bini speaker review of machine translations and agricultural terminology.
3. Collect independently authored Edo/Bini agricultural questions and answers.
4. Increase the number and diversity of underlying questions, especially for crop cultivation and other weak classes.
5. Revisit intent-boundary guidelines where labels overlap.
6. Evaluate on a new natural-Edo/Bini holdout set.
7. Investigate an Edo-specific BPE tokenizer after a sufficiently large, reviewed Edo text corpus is available.
8. Investigate Edo-specific lemmatization only if suitable linguistic resources become available.
9. Calibrate confidence scores and evaluate an abstention/clarification threshold.
10. Reassess model suitability before any production deployment.

---

**Model status statement:** This is an experimental bilingual intent-classification baseline. Its reported test results are valid for the supplied paired English and machine-translated Edo dataset and split. They should not be represented as validated performance on real-world, naturally authored Edo/Bini agricultural queries or as evidence of agricultural-answer accuracy.
