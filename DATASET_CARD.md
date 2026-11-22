# Dataset Card: EdoFarm AI Bilingual Agricultural Intent Dataset

A provisional bilingual dataset for training and evaluating the **agricultural intent classifier** used in EdoFarm AI. It contains English and machine-translated Edo (Bini) versions of agricultural questions, labelled with the intent the system should recognize. The underlying questions were selected from two existing agricultural Q&A datasets; no new farmer interviews or field data collection were conducted for this version.

This card describes the **classifier dataset**, not a validated agricultural knowledge base. The classifier predicts the type of request. It does not establish that an answer is agronomically correct.

- **Task:** Multilingual agricultural intent classification
- **Languages:** English; Edo (Bini)
- **Underlying records:** 306
- **Language examples:** 612 (306 English + 306 Edo)
- **In-scope Q&A records:** 231
- **Out-of-scope question records:** 75
- **Splits (train / validation / test):** 392 / 98 / 122 language examples
- **Underlying split groups:** 196 / 49 / 61 records
- **Version:** Provisional v1
- **Sources:** [FarmerChat Nigeria](https://huggingface.co/datasets/DigiGreen/farmerchat-queries) and [Agronomy QA Agriculture](https://huggingface.co/datasets/flamiinngo/agronomy-qa-agriculture)
- **Licence:** Mixed upstream licence obligations; no single licence is assigned to the combined derivative dataset yet. See [Licence and attribution](#licence-and-attribution).
- **Translation:** Edo (Bini) text was machine-generated with Gemini and has not yet been reviewed by a native Bini speaker.
- **Agricultural review:** Not expert-verified.

## Load it

The dataset has been exported as local CSV files. It has **not** been published to a Hugging Face dataset repository, so there is no public `load_dataset()` repository/configuration identifier yet.

```python
import pandas as pd

DATA_DIR = "data/processed"

# All 612 bilingual classifier examples
dataset = pd.read_csv(
    f"{DATA_DIR}/classifier_bilingual_dataset_provisional_v1.csv"
)

# Group-aware splits used for model development
train = pd.read_csv(
    f"{DATA_DIR}/classifier_bilingual_train_provisional_v1.csv"
)
validation = pd.read_csv(
    f"{DATA_DIR}/classifier_bilingual_validation_provisional_v1.csv"
)
test = pd.read_csv(
    f"{DATA_DIR}/classifier_bilingual_test_provisional_v1.csv"
)
```

Related files include:

- `data/processed/classifier_bilingual_summary_v1.csv` — split and label summary.
- `data/processed/edofarm_bilingual_dataset_provisional_v1.csv` — provisional bilingual question-and-answer companion data, including translation status.
- `data/processed/edofarm_translation_queue_v2_machine_filled.csv` — machine-filled translation review queue.

The exact CSV exports in the project directory are authoritative if a later notebook revision changes field names or adds metadata columns.

## Intended labels

Each question is assigned one primary intent:

| Label | Meaning |
|---|---|
| `crop_cultivation` | Planting, crop establishment, crop growth and general cultivation practices |
| `pest_management` | Identification or management of crop pests |
| `disease_management` | Questions about crop diseases, symptoms, prevention or management |
| `soil_management` | Soil condition, preparation, conservation and improvement |
| `irrigation` | Water application, irrigation methods and crop water management |
| `fertilizer_application` | Fertilizer selection, use and nutrient application |
| `harvesting` | Crop maturity, harvest timing and harvesting practices |
| `out_of_scope` | Requests outside the crop-production advisory scope, including livestock and veterinary questions |

The `out_of_scope` label is intentional. It helps the prototype identify questions that should not be answered as crop-production advice.

## Label and split distribution

Counts below are the underlying record counts. Each record contributes one English example and one Edo example, and both language versions remain in the same split.

| Intent | Records per language | English + Edo examples |
|---|---:|---:|
| `crop_cultivation` | 17 | 34 |
| `disease_management` | 40 | 80 |
| `fertilizer_application` | 40 | 80 |
| `harvesting` | 33 | 66 |
| `irrigation` | 32 | 64 |
| `out_of_scope` | 75 | 150 |
| `pest_management` | 25 | 50 |
| `soil_management` | 44 | 88 |
| **Total** | **306** | **612** |

| Split | Underlying records | English examples | Edo examples | Total examples |
|---|---:|---:|---:|---:|
| Train | 196 | 196 | 196 | 392 |
| Validation | 49 | 49 | 49 | 98 |
| Test | 61 | 61 | 61 | 122 |
| **Total** | **306** | **306** | **306** | **612** |

## Columns

The classifier exports retain the following core record metadata. The bilingual version also identifies the language of each text example; an underlying record's English and Edo examples share its record and split-group identity.

| Column | Description |
|---|---|
| `record_id` | Identifier for the underlying source/review record. |
| `question` | Classifier input text. It is English or machine-translated Edo (Bini), depending on the language example. |
| `primary_intent` | One of the eight intent labels listed above. |
| `language` | Language of the input example: English or Edo (Bini). |
| `split_group_id` | Group identifier used to keep related examples in one split and prevent record-level leakage. |
| `near_duplicate_group` | Identifier for a detected near-duplicate group, where applicable. |
| `source_dataset` | Upstream dataset from which the underlying record originated. |
| `review_decision` | Provisional screening decision recorded during dataset review. This is not agricultural expert approval. |
| `answer_quality` | Provisional answer-quality metadata inherited from the review workflow; not a factual accuracy guarantee. |

The classifier uses the **question text and intent label**. Answers are not classifier inputs. The companion bilingual Q&A export is a separate artifact and should not be treated as an approved retrieval knowledge base.

## How it was built

1. **Source ingestion.** The project combined a local Nigeria subset of FarmerChat containing 11,541 Q&A rows with 1,914 rows from the Agronomy QA Agriculture dataset, for 13,455 source rows before project filtering and review. Source fields were standardized while preserving source provenance. The source files used for this project are English-language data.
2. **Initial scope screening.** Questions were screened against the project's crop-production scope. Keyword-based matching was used as an initial aid only; keyword matches were not treated as definitive labels or agricultural validation.
3. **Review queue.** A 522-row review queue was created from the two sources for closer screening. The selected records were checked for scope, answer quality and obvious issues. This was a preliminary curation process, not review by a qualified agronomist.
4. **Classifier candidate selection.** 306 records were retained as provisional classifier candidates: 231 agricultural Q&A records and 75 out-of-scope questions. Pending or rejected material was not silently promoted into the classifier dataset.
5. **Label assignment.** Each selected record received one primary intent from the eight-label taxonomy. The labels are intended for request routing, not for asserting the correctness of the source answer.
6. **Duplicate-aware splitting.** Related records were grouped using record/split-group metadata, and train, validation and test splits were created at group level. This keeps the language pair and associated record in one split and reduces the risk of near-duplicate leakage across splits.
7. **Bilingual expansion.** Each of the 306 underlying records was represented once in English and once in Edo (Bini), producing 612 classifier examples. The English and Edo versions share the same intent and split assignment.
8. **Machine translation.** Gemini was used to generate Edo (Bini) question translations. The classifier translation set contains 306 machine-generated question examples. In the companion Q&A data, 230 of 231 agricultural answers received machine translations; one answer was left unresolved rather than forcing an uncertain translation. The 75 out-of-scope records are question-only examples for scope detection.
9. **Validation checks.** The bilingual exports were checked for unique example identifiers, complete English/Edo question pairs, consistent labels and split assignments, non-empty classifier inputs, and no underlying record crossing between splits.

## Evaluation notes

A lightweight TF-IDF word- and character n-gram classifier with Logistic Regression was evaluated on this dataset. On the held-out test set, the provisional results were:

| Language | Accuracy | Macro F1 |
|---|---:|---:|
| English | 0.7377 | 0.6687 |
| Edo (Bini) | 0.8197 | 0.7406 |

These results should be interpreted cautiously. The test set contains only 61 underlying records per language, class support is uneven, and the `crop_cultivation` class had zero recall in both language evaluations (three test examples per language). The metrics are an initial prototype benchmark, not evidence of reliable field performance.

## Intended use

- Training and evaluating an intent classifier for a research prototype that routes crop-production questions.
- Comparing English and provisional Edo (Bini) text classification under the same group-aware data split.
- Studying low-resource bilingual agricultural NLP and identifying data gaps for future collection and review.

## Out-of-scope and unsafe uses

- Do not use this dataset as a standalone source of agronomic facts or as an expert-approved recommendation database.
- Do not use model predictions to diagnose crop disease with certainty or prescribe pesticide, herbicide or fertilizer products, dosages or schedules without locally appropriate, expert-reviewed evidence.
- Do not treat `out_of_scope` as a complete safety system. The prototype requires additional refusal, uncertainty and escalation safeguards.
- Do not claim that the dataset represents all Edo-speaking farmers, all Nigerian farmers or African agriculture generally.
- Do not claim that Edo translations are native-speaker verified or that the source answers were validated by local agricultural experts.

## Limitations

- **No new field collection.** No farmer interviews, native-speaker recordings or locally collected farmer questions were added for this version. The dataset is adapted from existing sources.
- **Regional and source bias.** FarmerChat contributes Nigeria-related questions, while the Agronomy QA source has substantial non-Nigerian context, including Indian and other international examples. Some crops, seasons, varieties, pests and practices may not transfer to Edo State or Nigeria.
- **Source-answer quality.** FarmerChat responses are AI-generated. The Agronomy QA answers are not individually verified by an Edo/Nigerian agronomist for this project. Source inclusion does not imply factual correctness.
- **Machine-generated Edo.** All Edo (Bini) question translations were generated with Gemini. They have not yet undergone native Bini-speaker review, so wording, grammar, dialect and meaning may be inaccurate or unnatural. The translations should be considered provisional.
- **No agricultural expert validation.** The current review process is preliminary screening. No qualified agricultural expert has approved the dataset or its recommendations.
- **Small and imbalanced sample.** There are only 306 underlying records across eight labels. Several classes have limited support, and the out-of-scope class is larger than some agricultural classes.
- **Limited local crop coverage.** Project priorities include cassava, yam, maize, plantain, oil palm, pepper and okra, but the dataset does not provide balanced or comprehensive coverage of these crops.
- **Translation does not add independent evidence.** English and Edo rows are translations of the same underlying records, not independent farmer observations. They must remain grouped during splitting and evaluation.
- **Classifier scope is narrow.** The classifier predicts an intent, not the correct agronomic answer. The current response generator uses Gemini; retrieval from a verified agricultural knowledge base is not active in this prototype.
- **Known class weakness.** The selected model had zero recall for `crop_cultivation` on the small held-out test set. This class needs more varied and carefully reviewed examples.

## Licence and attribution

The combined dataset is derived from sources with different upstream licences:

1. **FarmerChat Nigeria** — Digital Green, [DigiGreen/farmerchat-queries](https://huggingface.co/datasets/DigiGreen/farmerchat-queries), listed as [Creative Commons Attribution 4.0 International (CC BY 4.0)](https://creativecommons.org/licenses/by/4.0/).
2. **Agronomy QA Agriculture** — [flamiinngo/agronomy-qa-agriculture](https://huggingface.co/datasets/flamiinngo/agronomy-qa-agriculture), listed as [Apache License 2.0](https://www.apache.org/licenses/LICENSE-2.0).

Because the project combines material under different upstream licences and adds machine-generated translations and project annotations, **do not label the combined dataset simply CC BY 4.0, Apache-2.0, MIT or another single licence without a separate compatibility and redistribution review**. Preserve source attribution and licence information at record level where possible. Check the current upstream dataset cards and applicable terms again before publishing or redistributing the processed files.

No new farmer data was collected for this release. Gemini was used as a translation tool; the generated translations are not human-verified translations and do not remove the need to comply with the upstream data licences and applicable service terms.

## Future work

- Have Edo (Bini) questions and answers reviewed or corrected by competent native speakers, with dialect and translation status recorded.
- Have crop-specific content and labels reviewed by qualified Nigerian agricultural experts or extension officers.
- Add more locally relevant, consented farmer questions and balance the intent classes, especially `crop_cultivation`.
- Verify source passages before activating retrieval-augmented generation (RAG); raw Q&A answers should not automatically become approved knowledge.
- Expand evaluation with a larger, independently collected test set and human assessment in both languages.
- Consider an Edo-specific subword tokenizer only after collecting a larger, reviewed Edo-language corpus.

## Citation

When using the adapted dataset, cite both upstream sources and identify the EdoFarm AI processing/translation version. Suggested attribution:

- Digital Green. *FarmerChat Agricultural Q&A Dataset*. [Hugging Face dataset](https://huggingface.co/datasets/DigiGreen/farmerchat-queries). CC BY 4.0.
- flamiinngo. *Agronomy QA Agriculture*. [Hugging Face dataset](https://huggingface.co/datasets/flamiinngo/agronomy-qa-agriculture). Apache License 2.0.
- EdoFarm AI project. *EdoFarm AI Bilingual Agricultural Intent Dataset*, provisional v1. Adapted, filtered, labelled and machine-translated for a research prototype; see this dataset card for methodology and limitations.
