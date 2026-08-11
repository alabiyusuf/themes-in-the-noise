# Themes in the Noise

Topic modelling applied to UK housing crisis research.

831 peer-reviewed abstracts from Scopus, 2015 to 2026, across 350 journals.
Five model configurations across three approaches. All of them found the same seven topics.

---

## What this is

The UK housing crisis has generated a large and fragmented body of academic literature.
Urban planners, economists, public health researchers and housing policy analysts are all
studying the same problem using entirely different vocabularies. A keyword search won't
connect them. Reading across 350 journals manually isn't feasible.

This notebook applies topic modelling to map the dominant research themes, compare how
different algorithms recover them, and surface structure that isn't visible from reading
papers one at a time.

The finding that prompted the write-up: energy retrofit emerged as a fully independent
research cluster in every model, unprompted. Topic T4. Separate from homelessness.
Separate from health and housing conditions. Three algorithms drew the same boundary
without any coordination between them.

Companion article: [link]

---

## Dataset

831 abstracts collected manually from Scopus via the University of Salford institutional
subscription. Six targeted TITLE-ABS-KEY queries covered different sub-dimensions of the
crisis rather than one broad search term:

```
TITLE-ABS-KEY("housing crisis" AND "United Kingdom")
TITLE-ABS-KEY("affordable housing" AND "England")
TITLE-ABS-KEY("social housing" AND "policy" AND "UK")
TITLE-ABS-KEY("homelessness" AND "urban" AND "Britain")
TITLE-ABS-KEY("housing affordability" AND "Manchester")
TITLE-ABS-KEY("tenant" AND "rental market" AND "England")
```

Each query was exported as a separate CSV then merged with deduplication on DOI.

The raw data is not in this repo. Redistributing Scopus exports falls outside the
institutional licence terms. The queries above and the merging logic in the notebook
are enough for anyone with Scopus access to rebuild the corpus. Place the merged file
at `uk_housing_crisis_merged.csv` in the repo root before running locally.

| Attribute | Value |
|---|---|
| Total abstracts | 831 |
| Year range | 2015 to 2026 |
| Unique journals | 350 |
| Mean abstract length | 230 words |
| Missing abstracts after deduplication | 0 |

---

## Models

The topic count k=7 was chosen through independent coherence searches for LDA and NMF
across k values of 5, 7, 9, 10, 12 and 15. Both peaked at 7. All five model runs use
the same k so results are directly comparable.

| Model | Coherence (c_v) | Training time |
|---|---|---|
| NMF Frobenius | 0.5056 | 0.32s |
| NMF KL-divergence | 0.5187 | 5.04s |
| LDA | 0.3952 | 11.84s |
| BERTopic v1 | see note | 9.56s + 159s embed |
| BERTopic v2 refined | see note | 10.56s (embed reused) |

BERTopic's c_v score is not directly comparable to NMF and LDA. The c_v metric measures
word co-occurrence; BERTopic works from semantic embeddings, so co-occurrence coherence
undersells what it actually recovers.

---

## Results

Seven topics, consistent across all approaches:

| Topic | Label |
|---|---|
| T1 | Health, Wellbeing and Housing Conditions |
| T2 | Housing Finance, Mortgages and Market Risk |
| T3 | Land Use, Planning and Green Belt Policy |
| T4 | Energy Retrofit and Low-Carbon Housing |
| T5 | Private Rented Sector and Tenancy Rights |
| T6 | Homelessness and Urban Housing Need |
| T7 | Social Housing Management and Allocation |

---

## Repo structure

```
themes-in-the-noise/
├── uk-housing-topic-modelling.ipynb
├── requirements.txt
├── .gitignore
└── README.md
```

The notebook is self-contained. Run it top to bottom and it produces all EDA charts,
coherence curves, model outputs and the comparative evaluation. BERTopic interactive
visualisations save as HTML files locally during the run. They are not committed to the
repo because of size but generate automatically on each run.

---

## Setup

Python 3.11 recommended. Works on Google Colab and local environments. The data loader
detects which and adjusts the file path automatically.

```bash
pip install -r requirements.txt
```

NLTK downloads `stopwords`, `wordnet`, `omw-1.4` and `punkt` on first run, silently.

BERTopic encodes 831 abstracts using `all-MiniLM-L6-v2`. On CPU that takes roughly
2 to 3 minutes the first time. Embeddings are cached after that so subsequent
configurations reuse them without re-encoding.

---

## Methodology notes

**Stop-word construction.** 286 terms across three tiers. The NLTK baseline covers
function words. A domain tier from EDA frequency analysis removes terms like "study,"
"result" and "find" that appear in virtually every abstract regardless of sub-topic.
A third tier was added after inspecting first-pass model outputs and finding that
"policy," "government" and "market" were flooding the top ten of multiple topics
simultaneously, making them indistinguishable.

**Vectorisation.** CountVectorizer for LDA because its variational inference assumes
integer counts. TF-IDF for NMF. Using TF-IDF before LDA is a known methodological
error and was avoided here. BERTopic bypasses vectorisation entirely and works from
sentence embeddings.

**N-gram detection.** Gensim's Phrases model detects statistically significant bigrams
and trigrams before vectorisation, producing compound tokens like `housing_affordability`
and `social_housing`. This is preferable to passing `ngram_range=(1,2)` directly to
CountVectorizer, which generates every possible adjacent pair regardless of whether
they co-occur meaningfully in the corpus.

**BERTopic text input.** BERTopic receives lightly cleaned natural language text, not
the lemmatised version used by NMF and LDA. Sentence transformers are trained on
natural sentences; feeding them heavily preprocessed tokens degrades embedding quality.

---

## Article

Non-technical write-up on Substack: https://yusufalabi.substack.com/p/i-analysed-831-papers-on-the-uk-housing?r=5uos48&utm_campaign=post&utm_medium=web&showWelcomeOnShare=true


Also on Medium: https://medium.com/@alabiyusuf793/i-analysed-831-papers-on-the-uk-housing-crisis-energy-retrofit-wasnt-what-i-expected-to-find-7651d3efeb3e

---

## Licence

MIT. Use and adapt freely. The Scopus data is not included and remains subject to its
original licence terms.
