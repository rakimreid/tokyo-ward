# Tokyo Ward Recommender: review analytics for "Tokyo Just for You"

[![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/rakimreid/tokyo-ward/blob/main/Tokyo_Ward_Recommender_Capstone.ipynb)

MSBA capstone (Quantic School of Business and Technology), Rakim, November 2026.

This repository turns Inside Airbnb data for 11 Tokyo wards into one evidence-based profile per ward, for a future app that recommends a base ward to inbound tourists. The pipeline cleans 738,305 guest reviews and host overviews, compares three sentiment models, finds topics with BERTopic, scores ten traveler aspects (food, nightlife, shopping, culture, family needs, nature, transit, safety, value, crowd level), validates the results against human-audited labels, and writes one JSON profile per ward.

**To see the results without running anything, open [`Tokyo_Ward_Recommender_Capstone.ipynb`](Tokyo_Ward_Recommender_Capstone.ipynb)**. It is saved with the outputs of a full run. The ward profiles are in [`data/ward_profiles/`](data/ward_profiles/).

## Key results

| | |
|---|---|
| Data | 20,370 listings and 738,305 reviews in 11 wards (Inside Airbnb, snapshot 29 September 2025) |
| Sentiment model | nlptown (multilingual BERT) and cardiffnlp tie on English (3-class weighted F1 0.854 vs 0.852); both beat VADER (0.724) on the label set. Weighted to the real, mostly positive review mix, only cardiffnlp's edge over VADER stays significant. nlptown was chosen for its six-language coverage. |
| Labels | LLM-assisted, human-audited: quadratic-weighted kappa 0.979 (reviews, n = 38) and 0.816 (sentences, n = 30) |
| Topics | 41 topics on 42,147 English sentences; largest topic 6.3%; c_v 0.634, c_npmi 0.121 |
| Aspects | 360,044 aspect mentions; tag precision about 0.92 (weighted, optimistic) |
| Ward vs Tokyo | Of 110 ward × aspect cells, 24 are above and 27 below the Tokyo mean (95% cluster-bootstrap CIs) |
| Validity | All 8 face-validity checks pass; ward location ratings correlate with walkability sentiment (ρ = 0.87, n = 11) |

Main limitations: about 34% of reviews (Japanese, Chinese, Korean) are not scored; sentence-level sentiment is weaker than review-level, so negative quotes were checked one by one (36 of 120 kept); and the 110 ward-vs-Tokyo flags are not adjusted for multiple comparisons. The full list is at the end of the notebook.

## Run it

**Google Colab (no installation).** Click the badge above, then choose *Runtime → Run all*. The setup cell clones this repository (code, data and precomputed GPU outputs) into the Colab session. A CPU runtime is enough, and a full run takes about 30 minutes.

**Locally.**
```bash
git clone https://github.com/rakimreid/tokyo-ward.git
cd tokyo-ward
python -m venv .venv && source .venv/bin/activate      # Windows: .venv\Scripts\activate
pip install -r requirements.txt
jupyter lab Tokyo_Ward_Recommender_Capstone.ipynb      # then Run All
```
Use Python 3.12 or 3.13. `requirements.txt` pins the versions the notebook was tested with. Installing BERTopic also installs PyTorch, which is a large download.

**GPU steps.** Three steps use transformer models: sentiment scoring (Stage 2b), sentence embeddings (3b) and aspect scoring (4b). Their outputs are included in `data/colab_out/`, so the notebook skips these steps and reproduces the reported numbers. To recompute them, delete those files and run on a Colab GPU (T4).

**Tested.** The notebook was run end to end without errors in two environments:
- Colab's package versions (Python 3.13, pandas 2.2.3, numpy 2.4.6, scikit-learn 1.6.1, umap-learn 0.5.12). This run produced the saved outputs.
- A newer stack (pandas 3.0, numpy 2.5, scikit-learn 1.9, umap-learn 0.6).

All downstream outputs are identical in the two environments. Only the tuning-grid rows for `n_neighbors` 10 and 30 differ slightly, because UMAP's results depend on package versions. The GPU code paths were tested on CPU with small stand-in models; the real models were run on a Colab T4.

## Repository layout

```
Tokyo_Ward_Recommender_Capstone.ipynb   the whole pipeline, with saved outputs
requirements.txt                        tested package versions
raw/                                    Inside Airbnb input files (see raw/README.md)
data/colab_out/                         precomputed GPU outputs (embeddings split into 3 parts)
data/ward_profiles/                     results: one JSON per ward, a combined file, sanity_check.csv
data/interim/                           stage summaries and topic tables (larger intermediates are regenerated)
```
The notebook writes its source modules to `src/` and its embedded labels to `data/labels/` when it runs.

## Data and licences

- **Input data:** Inside Airbnb, Tokyo, snapshot of 29 September 2025 (https://insideairbnb.com/get-the-data/), used under [CC BY 4.0](https://creativecommons.org/licenses/by/4.0/). The files here keep only the columns the pipeline reads, with reviewer and host names removed. They are compressed and split so that each file is under 25 MB. See [`raw/README.md`](raw/README.md).
- **Ward profiles:** these contain short quotes from guest reviews (from the same CC BY 4.0 data), shown without reviewer names.
- **Models:** `nlptown/bert-base-multilingual-uncased-sentiment`, `cardiffnlp/twitter-roberta-base-sentiment-latest` and `sentence-transformers/paraphrase-multilingual-mpnet-base-v2` from Hugging Face, each under its own licence.

## AI assistance

Claude (Anthropic) helped write the code and produced the LLM-assisted labels; Rakim audited samples of these labels blind. Claude also read every negative quote before publication. Every label source is identified in the notebook, and the capstone report has a full disclosure.
