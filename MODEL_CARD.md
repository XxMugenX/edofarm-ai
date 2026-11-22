# Model Card: EdoFarm AI Bilingual Agricultural Intent Classifier

## Model summary

EdoFarm AI is a research prototype for routing crop-production questions in **English and Edo (Bini)**. Its trained machine-learning component is a lightweight text classifier that predicts the intent of a user's question. The classifier is not a conversational language model and does not generate agricultural advice.

The current prototype uses a separate **Gemini Flash** API component to generate responses after intent classification. Gemini is accessed as a hosted model; it was not fine-tuned by this project. A verified agricultural retrieval-augmented generation (RAG) knowledge base is not active in this version.

- **Model name:** EdoFarm AI Bilingual Agricultural Intent Classifier
- **Version:** Provisional v1
- **Task:** Multilingual text classification / agricultural intent routing
- **Languages:** English; Edo (Bini)
- **Architecture:** TF-IDF word n-gram and character n-gram features with Logistic Regression
- **Selected feature configuration:** Combined word- and character-level TF-IDF features (`word_character_combined`)
- **Training examples:** 392
- **Validation examples:** 98
- **Test examples:** 122
- **Underlying records:** 306, represented once in each language
- **Labels:** 8 agricultural-intent and scope labels
- **Training approach:** Supervised learning; no foundation-model fine-tuning
- **Response generation:** Separate Gemini Flash API call
- **Knowledge retrieval:** Not active; zero approved knowledge passages at the time of this version
- **Status:** Research prototype; not approved for operational agricultural advice

## Model components

EdoFarm AI currently consists of two distinct model components:

| Component | Role | Training / deployment status |
|---|---|---|
| Intent classifier | Predicts the likely intent of an incoming question and helps route it | Trained locally using TF-IDF word and character features with Logistic Regression |
| Gemini Flash | Generates a natural-language response after the question is processed | Accessed through an API; not fine-tuned as part of this project |

The classifier's prediction is not proof that the user's question has been understood correctly. The response generator can also produce incorrect or unsupported information. The two components must therefore be evaluated separately as well as together.

## Intended use

The classifier is intended for:

- Research and demonstration of bilingual agricultural-intent classification.
- Routing English and provisional Edo (Bini) crop-production questions to one of the supported intent categories.
- Identifying questions that appear to fall outside the prototype's crop-production scope.
- Comparing a lightweight word/character n-gram baseline across English and a low-resource Nigerian language.
- Supporting further research into locally relevant agricultural conversational systems.

The complete prototype may be used in supervised demonstrations where users are told that responses are AI-generated and have not been validated as agricultural recommendations.

## Out-of-scope use

Do not use this model as:

- An autonomous agronomist, crop doctor or agricultural extension officer.
- A definitive crop-disease, pest or nutrient-deficiency diagnostic system.
- A source of verified fertilizer, pesticide or herbicide recommendations, rates, dosages, mixtures or application schedules.
- A substitute for local agronomic guidance, field inspection, laboratory testing or advice from a qualified agricultural extension officer.
- A production service for unsupervised farmer decision-making without further data collection, expert review, safety testing and field evaluation.
- Evidence that the system understands all Edo (Bini) dialects or the full range of English and Edo-speaking farmers' language.

The `out_of_scope` prediction must not be treated as a complete safety or refusal mechanism. Classification errors can cause an in-scope question to be rejected or an out-of-scope question to be routed as agricultural.

## Supported intent labels

The classifier predicts one primary label from the following set:

| Label | Description |
|---|---|
| `crop_cultivation` | Planting, crop establishment, crop growth and general cultivation practices |
| `pest_management` | Questions about crop pests and their prevention or management |
| `disease_management` | Questions about crop diseases, symptoms, prevention or management |
| `soil_management` | Soil condition, preparation, conservation and improvement |
| `irrigation` | Water application, irrigation methods and crop water management |
| `fertilizer_application` | Fertilizer selection, use and nutrient application |
| `harvesting` | Crop maturity, harvest timing and harvesting practices |
| `out_of_scope` | Questions outside the defined crop-production advisory scope, including livestock and veterinary requests |

The label set is an operational taxonomy created for this prototype. It is not a complete taxonomy of agricultural knowledge, and some real questions may reasonably belong to more than one category.

## Training data

The classifier was trained using the provisional **EdoFarm AI Bilingual Agricultural Intent Dataset v1**. See [`DATASET_CARD.md`](DATASET_CARD.md) for the full dataset description, source attribution, preparation history and licensing notes.

- **Underlying records:** 306
- **Agricultural Q&A records:** 231
- **Out-of-scope question records:** 75
- **English examples:** 306
- **Edo (Bini) examples:** 306
- **Total language examples:** 612
- **Train / validation / test:** 392 / 98 / 122 language examples
- **Underlying records per split:** 196 / 49 / 61

Each underlying record has an English and an Edo (Bini) question example. The paired examples share the same intent and split assignment. The split was performed by record/group rather than randomly splitting individual language rows, so an English question and its Edo translation do not appear in different splits.

The source data came from FarmerChat Nigeria and Agronomy QA Agriculture. The project did not conduct new farmer interviews or collect new farmer speech for this version. Edo (Bini) question translations were generated with Gemini and have not yet been reviewed by a native Bini speaker. The source answers and labels have not been approved by a qualified agricultural expert for this project.

## Training procedure

The selected baseline represents question text using two complementary feature types:

1. **Word-level TF-IDF n-grams**, which capture individual terms and short sequences of words.
2. **Character-level TF-IDF n-grams**, which can capture partial word patterns and may be more tolerant of spelling variation, inflection and spacing differences.

The two feature representations are combined and passed to a Logistic Regression classifier. The combined word/character configuration was selected during validation experiments and then evaluated on the held-out test split.

This is a conventional supervised text-classification pipeline. It is not a transformer, does not contain a trained generative language model, and does not use a custom Edo subword/BPE tokenizer. Lemmatization was not used in the selected baseline.

The project records the following methodological choices, but the exact fitted hyperparameters and software environment should be taken from the final training notebook and exported model metadata before a reproducible public release. In particular, this card does not assert unrecorded values for the TF-IDF n-gram ranges, vocabulary limits, Logistic Regression regularization, solver, random seed or library versions.

## Evaluation

The selected bilingual word/character model was evaluated on the held-out test split. Each language has 61 examples, corresponding to the same 61 underlying test records.

| Language | Accuracy | Macro precision | Macro recall | Macro F1 | Weighted F1 |
|---|---:|---:|---:|---:|---:|
| English | 0.7377 | 0.6808 | 0.6956 | 0.6687 | 0.7121 |
| Edo (Bini) | 0.8197 | 0.7488 | 0.7596 | 0.7406 | 0.8030 |

These are preliminary results from a small, internally constructed test set. They should not be interpreted as expected accuracy on real farmer queries or as evidence of reliable deployment performance.

### Important evaluation finding

The `crop_cultivation` class had **zero recall in both language test evaluations**, with only three test examples for that class in each language. This means the evaluated model failed to correctly identify any of those crop-cultivation test examples. The class needs more varied examples, improved labelling and further evaluation before the classifier can be relied upon for that category.

The test set is also small and class support is uneven. The language examples are translations of the same underlying records, not independent samples from separate English- and Edo-speaking populations. Results across languages are therefore useful for an initial controlled comparison, but they do not establish general language performance.

## Known limitations and risks

### Data limitations

- **Small dataset:** 306 underlying records are used across eight labels. This is limited coverage for a broad agricultural domain.
- **Source variation:** The combined sources have different geographic and agronomic contexts. Some Agronomy QA material is associated with non-Nigerian agricultural conditions and may not transfer directly to Edo State.
- **No direct local data collection:** No new interviews, farmer-submitted questions or audio recordings were collected for this version.
- **Machine-translated Edo:** Edo (Bini) classifier inputs were generated by Gemini. No native-speaker validation has yet been completed. Translation errors, unnatural phrasing, dialect differences or semantic drift may affect model behaviour.
- **No expert sign-off:** Neither the source answers nor the resulting system have been validated by a qualified Nigerian agronomist or agricultural extension team.
- **Imbalanced intent coverage:** Some labels have relatively few examples. The crop-cultivation test failure demonstrates the impact of limited class coverage.

### Model limitations

- **Intent only:** The classifier predicts a category; it does not verify facts, diagnose crop problems or generate recommendations.
- **Vocabulary sensitivity:** TF-IDF relies on text patterns observed during training. Unseen crops, local names, spelling variations, code-switching and uncommon phrasing may reduce performance.
- **Low-resource language constraints:** The Edo examples are machine-translated and do not adequately represent natural variation in spoken or written Bini.
- **Potential overconfidence:** A classifier may return a label even when the query is ambiguous, incomplete or unlike the training data. A prediction should not automatically trigger a high-impact recommendation.
- **Observed routing error:** During an integration scenario, a cassava yellowing question was predicted as `out_of_scope`, although crop symptoms are within the intended scope. This illustrates that routing errors remain possible.
- **No verified retrieval layer:** RAG is not active. The response generator does not currently ground its answers in a project-approved, source-verified set of agricultural passages.

### Response-generation limitations

Gemini Flash is a separate hosted service and its responses are not covered by the classifier's test metrics. Generated responses can be incomplete, inaccurate, overgeneralized or inconsistent with local conditions. Model behaviour may also change when the provider updates the hosted model. Record the exact Gemini model identifier, API version and generation settings used for each formal evaluation or release.

## Safety considerations and recommended safeguards

Before any farmer-facing deployment, the project should:

- Add explicit uncertainty handling for low-confidence, ambiguous or out-of-distribution questions. Do not rely on the predicted class alone as the safety check.
- Require the response generator to avoid definitive diagnosis and unsupported chemical or fertilizer instructions.
- Require clarification or referral to a qualified extension officer when a recommendation depends on location, crop variety, growth stage, severity, weather, soil test results or other missing context.
- Use only locally appropriate, source-verified and expert-reviewed material for factual retrieval. Do not automatically promote raw source Q&A answers into the knowledge base.
- Have Edo (Bini) content reviewed by competent native speakers and agricultural content reviewed by qualified local experts.
- Test the system with real-world questions from intended users, with informed consent and appropriate oversight, before operational use.
- Monitor misclassification, unsafe response generation, translation issues and user feedback; maintain a process for correcting or withdrawing problematic content.

For the current research prototype, responses should be labelled as AI-generated and preliminary. Users should be directed to local agricultural extension services for consequential crop-management decisions.

## Ethical and social considerations

The project aims to explore access to crop-production information in English and Edo (Bini). However, a bilingual interface does not by itself guarantee language inclusion or equitable performance. Machine translation may fail to reflect natural Bini usage, and the source data may not represent the priorities or farming practices of local communities. The system should not be presented as community-validated until local speakers and agricultural practitioners have participated in review and evaluation.

No claim is made that the data represents all Edo-speaking farmers, all farmers in Nigeria or African smallholder agriculture generally. Future collection of local data should use informed consent, clear data-use terms and appropriate protection for personal or farm-specific information.

## Deployment and reproducibility

The classifier was developed in a Python/scikit-learn notebook workflow. The response-generation component calls Gemini Flash separately. The proposed application architecture may expose classifier inference through a Python service and keep application/business logic in a JavaScript backend, but that full production service architecture should not be considered implemented merely because notebook-level integration tests succeeded.

A reproducible release should publish or record:

- The exact serialized classifier artifact and its SHA-256 checksum.
- The matching vectorizer/preprocessing pipeline, including whether it is bundled with the classifier.
- Training notebook revision, dataset version and split files.
- Python, scikit-learn, NumPy, SciPy and joblib versions.
- Exact TF-IDF and Logistic Regression hyperparameters.
- A fixed random seed where applicable.
- Evaluation scripts, per-class precision/recall/F1, confusion matrices and prediction examples.
- The Gemini model identifier, API version and generation settings used for any end-to-end evaluation (without publishing API keys or secrets).

Example inference pattern (replace the path with the actual exported artifact path in the project):

```python
import joblib

MODEL_PATH = "models/intent_classifier.joblib"  # Update to the actual exported path.
classifier = joblib.load(MODEL_PATH)

question = "What should I consider before planting maize?"
predicted_intent = classifier.predict([question])[0]

print(predicted_intent)
```

This example assumes the serialized artifact contains the fitted text preprocessing/vectorization and classifier pipeline. If the project exported the vectorizer and classifier separately, load and apply both in the exact order used during training. Do not use a guessed artifact path in a deployed application.

## Future work

- Collect a larger, consented set of locally relevant English and Edo (Bini) farmer questions.
- Have Bini speakers review and correct translations, including spelling, dialect and naturalness.
- Have agricultural experts review labels and source content, especially disease, pest and chemical-management examples.
- Add diverse examples for `crop_cultivation` and other weak or underrepresented intents.
- Expand the held-out evaluation set and add a separately collected, locally representative test set.
- Evaluate performance by crop, intent, language form, spelling variation and code-switching; publish per-class metrics and confusion matrices.
- Add calibrated confidence thresholds, abstention and safe routing for uncertain predictions.
- Evaluate the Gemini response generator independently for factuality, local appropriateness, completeness and safety.
- Build a source-verified, expert-reviewed RAG collection before relying on retrieved agricultural recommendations.
- Consider an Edo-specific subword tokenizer only after a sufficiently large and reviewed Bini-language corpus is available.

## Citation and attribution

When using or discussing this model, cite the EdoFarm AI project and its associated dataset card. The classifier was trained on the provisional EdoFarm AI Bilingual Agricultural Intent Dataset, which is adapted from the following upstream sources:

- Digital Green. *FarmerChat Agricultural Q&A Dataset*. [Hugging Face dataset](https://huggingface.co/datasets/DigiGreen/farmerchat-queries). See the upstream dataset card for its licence and attribution requirements.
- flamiinngo. *Agronomy QA Agriculture*. [Hugging Face dataset](https://huggingface.co/datasets/flamiinngo/agronomy-qa-agriculture). See the upstream dataset card for its licence and attribution requirements.
- EdoFarm AI project. *EdoFarm AI Bilingual Agricultural Intent Dataset*, provisional v1. See [`DATASET_CARD.md`](DATASET_CARD.md) for adaptation details and licence notes.

The combined training data has mixed upstream licence obligations. Refer to the dataset card and the current upstream terms before redistributing the dataset or model artifact. This model card does not grant a separate licence to the upstream data or its derivatives.

## Model card status

**Provisional research documentation — 21 November 2026.** Update this card whenever the training data, model artifact, label taxonomy, evaluation results, translation status, Gemini model identifier or deployment architecture changes.
