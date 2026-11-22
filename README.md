# EdoFarm AI --- Bilingual Agricultural Advisory Assistant

[![Project](https://img.shields.io/badge/Project-EdoFarm%20AI-2ea44f)](#)
[![Task](https://img.shields.io/badge/Task-Intent%20Classification%20%2B%20Conversational%20AI-blue)](#)
[![Languages](https://img.shields.io/badge/Languages-English%20%7C%20Edo%20(Bini)-yellow)](#)
[![Status](https://img.shields.io/badge/Status-Research%20Prototype-orange)](#)

**Project Category:** Conversational Assistant\
**Domain:** Crop-production advisory\
**Supported languages:** English and Edo (Bini)\
**Intent classifier:** TF-IDF word and character n-grams + Logistic
Regression\
**Response-generation model:** Gemini Flash API\
**Retrieval-augmented generation (RAG):** Planned; not yet enabled with
approved knowledge passages\
**Project status:** Working research prototype

------------------------------------------------------------------------

## 🎯 The Problem

Smallholder farmers need accessible, understandable information about
crop production, soil management, pests, diseases, irrigation,
fertilizer application and harvesting. Access to agricultural extension
support may be limited, and language can make it harder for farmers to
obtain or use relevant information.

EdoFarm AI explores a bilingual, conversational interface that can
classify a farmer's question and generate a helpful response in English
or Edo (Bini).

## 💡 The Solution

EdoFarm AI combines two components:

1.  **An intent-classification model** that predicts the category of a
    farmer's question.
2.  **Gemini Flash** that generates a conversational response using the
    question, predicted intent and the system's crop-advisory
    instructions.

The classifier acts as an initial routing and scope-control component.
Gemini generates the response; the classifier itself does not contain
the agricultural answers.

The current prototype supports crop-production questions and identifies
out-of-scope requests. It is not a substitute for qualified agricultural
extension advice, laboratory diagnosis or professional veterinary
services.

## 👥 Who is this for?

-   Smallholder farmers and prospective users of digital agricultural
    advisory services
-   Agricultural extension services and community-based advisory
    programmes
-   Agricultural researchers and universities
-   Students exploring machine learning, bilingual NLP and
    conversational AI for African languages

## 🔬 Research Focus

The project investigates how a lightweight intent classifier, trained
with English and machine-translated Edo (Bini) question examples, can
support a bilingual agricultural conversational assistant.

Key questions include:

-   How well can the classifier identify supported agricultural intents
    in English and Edo (Bini)?
-   Does including Edo (Bini) examples improve classification for
    Edo-language inputs?
-   What are the limitations of machine-generated translations when
    building NLP resources for a lower-resource language?
-   How can a general-purpose language model be constrained to provide
    safer, more relevant crop-production responses?

The current results are preliminary. The Edo (Bini) translations have
not yet undergone human review by fluent speakers, and the agricultural
answers have not been validated by an agricultural expert.

## 🌱 Supported Scope

The current classifier uses eight labels:

  -----------------------------------------------------------------------
  Intent                              Description
  ----------------------------------- -----------------------------------
  `crop_cultivation`                  Crop establishment and general
                                      cultivation practices

  `pest_management`                   Management of pests affecting crops

  `disease_management`                Crop disease symptoms and
                                      management

  `soil_management`                   Soil preparation, soil health and
                                      soil management

  `irrigation`                        Water supply, drainage and crop
                                      water management

  `fertilizer_application`            Fertilizer and nutrient application

  `harvesting`                        Harvest timing and
                                      post-harvest-related crop questions

  `out_of_scope`                      Questions outside the supported
                                      crop-production scope
  -----------------------------------------------------------------------

Livestock, poultry, veterinary medicine and aquaculture are outside the
current advisory scope. The prototype should direct users with
animal-health concerns to an appropriate veterinary professional.

## 📊 Dataset Card

### Source datasets

The initial dataset was assembled from two existing agricultural
question-and-answer resources:

1.  **FarmerChat Nigeria** --- `DigiGreen/farmerchat-queries`
2.  **Agronomy QA Pairs** --- `flamiinngo/agronomy-qa-agriculture`

The original combined source collection contained **13,455 records**
before filtering and review. The sources have different origins and
limitations. In particular, the Agronomy QA collection may contain
regional assumptions that do not directly transfer to Edo State or
Nigeria.

### Curated classifier dataset

After filtering, preliminary AI-assisted review and selection, the
classifier dataset contains:

  Item                                       Count
  ---------------------------------------- -------
  Underlying question records                  306
  English examples                             306
  Edo (Bini) machine-translated examples       306
  Total language examples                      612
  Training examples                            392
  Validation examples                           98
  Test examples                                122

Each underlying record contributes an English and an Edo (Bini)
question. The paired examples retain the same group assignment so that
translations of one question do not appear across different data splits.

The 306 underlying records comprise 231 agricultural Q&A records and 75
out-of-scope questions. The classifier uses the question text and intent
label; it does not train on the answer text.

### Languages and translation

-   English: source-language questions.
-   Edo (Bini): machine-generated translations produced with Gemini.
-   Human translation/review: pending.

The Edo (Bini) examples are experimental machine translations, not a
verified community language dataset. Some translations may contain
unnatural wording, inconsistent terminology or meaning shifts. They
should be reviewed by fluent Edo (Bini) speakers before being treated as
reliable language resources or used in a public deployment.

### Data quality and provenance

-   Source records retain identifiers and source metadata for
    traceability.
-   English/Edo pairs share a common underlying record and split group.
-   Agricultural accuracy is marked `not_expert_verified`.
-   The AI-assisted review process is preliminary and does not
    constitute agronomic expert validation.
-   No claim is made that the dataset was collected directly from
    farmers in Edo State.

## 🧠 Model and Implementation

### 1. Intent classifier

The classifier uses a scikit-learn pipeline built around:

-   TF-IDF feature extraction
-   Word n-grams
-   Character n-grams
-   Combined word-and-character features
-   Logistic Regression classification

The selected candidate uses combined word and character features. Word
n-grams capture useful word-level patterns, while character n-grams can
help represent spelling variations and morphology without requiring a
separately trained subword tokenizer.

The classifier returns a predicted intent and class probabilities.
Probabilities should be treated as model confidence estimates, not
guarantees that a prediction is correct.

### 2. Conversational response generation

Gemini Flash is called after intent classification. The application
supplies the farmer's question, the predicted intent and system
instructions that define the assistant's role and scope.

The classifier and Gemini have different responsibilities:

  -----------------------------------------------------------------------
  Component                           Responsibility
  ----------------------------------- -----------------------------------
  Intent classifier                   Classifies the question and helps
                                      identify out-of-scope requests

  Gemini Flash                        Generates the natural-language
                                      response

  Application instructions            Define supported topics, response
                                      style and safety boundaries
  -----------------------------------------------------------------------

Gemini is a general-purpose model. Its responses are not automatically
verified agricultural recommendations. The system must communicate
uncertainty and avoid unsupported precision, especially for chemical
use, pesticide rates, fertilizer dosage and diagnosis.

### 3. Retrieval-augmented generation (planned)

A source-grounded retrieval layer is a future improvement. The project
has started identifying agricultural reference documents, but the
current knowledge review has **zero approved knowledge passages**. Some
downloaded PDF guides are image-based and require OCR or manual
inspection before their content can be reliably extracted.

RAG should only be enabled after passages have been checked for source
identity, relevance and agricultural reliability. Raw dataset answers
and unverified AI-generated answers must not automatically be treated as
trusted knowledge.

### 4. Future language technology

A dedicated Edo (Bini) subword tokenizer, such as a BPE tokenizer, is a
possible future direction. It is not part of the current implementation.
Building one should involve a larger, carefully curated Edo-language
corpus and collaboration with fluent speakers.

## 📏 Evaluation

The classifier was evaluated separately on English and Edo (Bini)
examples using a held-out test split of 61 examples per language.

### Final test results

  -----------------------------------------------------------------------------
  Evaluation       Accuracy        Macro Macro recall     Macro F1  Weighted F1
  language                     precision                           
  ------------ ------------ ------------ ------------ ------------ ------------
  English            0.7377       0.6808       0.6956       0.6687       0.7121

  Edo (Bini)         0.8197       0.7488       0.7596       0.7406       0.8030
  -----------------------------------------------------------------------------

These results are an initial baseline, not evidence that the model is
production-ready or that the Edo (Bini) translations are linguistically
accurate. The test set is small, and the Edo-language examples are
machine-translated. The `crop_cultivation` class was not correctly
identified in either final test evaluation, which indicates a clear area
for improvement.

The validation and test results should be interpreted separately. The
final test set was held out from model selection and is the more
appropriate estimate of performance on this dataset, but further
evaluation on naturally written Edo (Bini) questions is needed.

## 🛡️ Safety and Limitations

EdoFarm AI is a research prototype. Important limitations include:

-   The dataset is relatively small and may not represent the full range
    of farmer questions.
-   Edo (Bini) translations are machine-generated and have not yet been
    reviewed by fluent speakers.
-   Agricultural answers and source records have not received
    comprehensive expert validation.
-   Gemini may produce plausible-sounding but inaccurate or overly
    general advice.
-   The RAG knowledge base is not yet active because no passages have
    been approved.
-   The system should not invent pesticide products, chemical
    application rates or fertilizer dosages.
-   The system should not claim a definitive diagnosis from a short
    description or image.
-   Serious crop problems should be referred to a qualified agricultural
    extension professional.
-   Veterinary and livestock-health requests are outside the system's
    scope.

A production deployment should include human language review,
agricultural expert review, additional field evaluation and ongoing
monitoring.

## 🚀 Getting Started

The project is developed locally using Python notebooks. The
conversational prototype also uses the Gemini API.

1.  Clone or download the project repository.
2.  Create and activate a Python virtual environment.
3.  Install the dependencies listed in `requirements.txt` (if present).
4.  Configure the Gemini API key as an environment variable. Do not
    commit API keys to Git.
5.  Open the notebooks in the `notebooks/` directory.
6.  Run the notebooks in their documented order, using the generated
    files in `data/processed/` and `data/metadata/` where required.
7.  Follow the integration instructions for the classifier and Gemini
    API.

Example environment variable:

``` bash
GEMINI_API_KEY=your_api_key_here
```

Do not put a real API key in a notebook committed to a public
repository. Use a local `.env` file excluded by `.gitignore`, or
configure a secret in the deployment environment.

### Main project directories

``` text
edofarm-ai/
├── data/
│   ├── raw/          # Original source datasets
│   ├── interim/      # Intermediate extraction and processing outputs
│   ├── processed/    # Curated datasets, splits and model artifacts
│   └── metadata/     # Source register, review metadata and reports
├── notebooks/        # Data preparation, translation and modelling notebooks
├── models/           # Exported model artifacts, if stored here
├── src/              # Application and model source code, as developed
├── requirements.txt
└── README.md
```

This is a guide to the intended project organization; use the actual
files and folders present in the repository.

## 🔭 Future Work

-   Review and correct machine-generated Edo (Bini) translations with
    fluent speakers.
-   Expand the dataset with naturally written or spoken Edo (Bini)
    farmer questions, with appropriate consent and documentation.
-   Obtain agricultural expert review of question-answer content and
    reference passages.
-   Improve weak intent classes, especially `crop_cultivation`.
-   Evaluate on independently collected, naturally written farmer
    questions.
-   Build and evaluate an Edo-specific BPE tokenizer when enough
    suitable language data and speaker support are available.
-   Complete OCR and source verification for agricultural reference
    documents.
-   Add RAG using only approved, traceable agricultural passages.
-   Add human review and escalation workflows for uncertain or high-risk
    responses.
-   Deploy the classifier and Gemini integration behind a secure backend
    API.
-   Monitor model confidence, failure cases, language quality and
    response safety after deployment.

## ❓ Frequently Asked Questions

<details>
<summary>Is EdoFarm AI a fine-tuned small language model?</summary>

Not at the current stage. The implemented system uses a lightweight intent classifier together with Gemini Flash for response generation. Fine-tuning a domain-specific language model may be explored later, but it should not be described as completed work.

</details>

<details>
<summary>Does the intent classifier generate answers?</summary>

No. The classifier predicts the category of a question, such as `soil_management` or `disease_management`. Gemini Flash generates the conversational answer. The classifier is used for routing and scope handling, not as the answer-generation model.

</details>

<details>
<summary>Why use word and character n-grams instead of BPE?</summary>

Word and character TF-IDF n-grams provide a lightweight baseline that can be trained with the available dataset. A dedicated BPE tokenizer could be useful in the future, but training a meaningful Edo-specific tokenizer requires a larger and better-curated Edo-language corpus and input from fluent speakers.

</details>

<details>
<summary>Are the Edo (Bini) translations human-verified?</summary>

No. The current translations were generated with Gemini and remain provisional. Human review by fluent Edo (Bini) speakers is planned. Until then, the translations should not be presented as verified or fully representative of natural Edo speech.

</details>

<details>
<summary>Does EdoFarm AI use RAG?</summary>

Not yet. Agricultural documents have been identified and some have been downloaded, but document extraction and verification are incomplete. The approved knowledge-passage count is currently zero. Retrieval should only be activated after the passages have been checked and approved.

</details>

<details>
<summary>Can farmers rely on the answers as professional agricultural advice?</summary>

No. This is a research prototype. Gemini can make mistakes, and the current dataset has not been comprehensively validated by agricultural experts. The assistant should provide general information, communicate uncertainty and refer high-risk or uncertain cases to qualified agricultural extension professionals.

</details>

<details>
<summary>How can the application be deployed?</summary>

The intended approach is to serve the classifier and Gemini integration through a secure backend API. The API key must remain server-side. The exact deployment configuration will depend on the final backend and hosting environment.

</details>

## 📚 References and Resources

-   [FarmerChat Nigeria dataset --- Hugging
    Face](https://huggingface.co/datasets/DigiGreen/farmerchat-queries)
-   [Agronomy QA Agriculture dataset --- Hugging
    Face](https://huggingface.co/datasets/flamiinngo/agronomy-qa-agriculture)
-   [Google Gemini API
    documentation](https://ai.google.dev/gemini-api/docs)
-   [scikit-learn documentation](https://scikit-learn.org/stable/)
-   [TF-IDF vectorizer
    documentation](https://scikit-learn.org/stable/modules/generated/sklearn.feature_extraction.text.TfidfVectorizer.html)
-   [Logistic Regression
    documentation](https://scikit-learn.org/stable/modules/generated/sklearn.linear_model.LogisticRegression.html)

------------------------------------------------------------------------

**EdoFarm AI --- Research prototype for bilingual agricultural question
understanding and response generation.**
