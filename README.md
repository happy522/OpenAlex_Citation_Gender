# OpenAlex Citation Gender Analysis

A Jupyter notebook analyzing inferred author gender, authorship composition, and citation impact across three academic subfields, using data from [OpenAlex](https://openalex.org).

## Corpora

The analysis covers three OpenAlex subfields:

1. **Library and Information Sciences** (subfield `3309`)
2. **Computer Graphics and Computer-Aided Design** (subfield `1704`)
3. **Theoretical Computer Science** (subfield `2614`)

## What it does

For each corpus, the notebook:

- Fetches all works (papers) and their authorships, either from the OpenAlex API (cursor pagination, no 10k-row cap) or from a local CSV dump.
- Infers each author's gender from their given name, using **WGND 2.0** (`global_gender_predictor`, ~26M name–country records) as the primary signal, with `gender_guesser` as a fallback when WGND has no confident match. Low-confidence/ambiguous names are labeled `unknown` rather than dropped.
- Builds three analysis-ready tables — per-authorship, per-paper, and per-author (aggregated across a researcher's papers within a corpus) — and saves them as Parquet (CSV fallback if no Parquet engine is available).
- Answers a series of research questions with tables, statistical tests, and plots (see below).

## Research questions covered

| RQ | Question | Method |
|----|----------|--------|
| RQ1 | Author gender distribution per corpus | Counts/shares of inferred author gender, overall and by first author |
| — | Only-female / only-male / mixed-gender authorship | Per-paper classification and stacked bar charts |
| RQ2 | Gender composition vs. citation counts | Binned means with 95% CI; fractional citation-share attribution; share of citations to female authors over time |
| RQ3 | Citations by first-author gender | Descriptive stats, Mann–Whitney U test, repeated with FWCI (field- and year-normalized impact), percentile bar charts, survival-curve comparisons |
| — | Representation & inequality | Representation ratio (top-x% by citations) and Gini coefficient, following Jaramillo, Macedo, Oliveira, Karimi & Menezes (2025) |
| RQ4 | Author gender over time | Normalized yearly share of female/male first-authored papers, with rolling averages, per corpus and combined |

Data-quality diagnostics (missing fields, gender-source breakdown, capped-authorship works, corpus overlap/Jaccard similarity) are included before the RQs.

## Setup

```bash
python3 -m venv venv && source venv/bin/activate   # venv\Scripts\activate on Windows
pip install requests pandas numpy matplotlib scipy pyarrow global_gender_predictor gender-guesser jupyter
```

Then launch the notebook:

```bash
jupyter notebook OpenAlex_Citation_Gender.ipynb
```

### Configuration (top of notebook)

- `USE_LOCAL_CSV` — if `True`, reads `LOCAL_CSV_PATH` (default `openalex_full_works.csv`) instead of calling the API. Set to `False` to fetch fresh data from OpenAlex.
- `MAX_WORKS_PER_CORPUS` — set an integer (e.g. `2000`) to cap each corpus for a fast test run, or `None` for a full run.
- `OPENALEX_API_KEY` — read from the `OPENALEX_API_KEY` environment variable, or entered interactively at runtime; optional but recommended if OpenAlex rate-limits anonymous requests.

## Data layout

```
openalex_data/
├── raw/     # per-corpus .jsonl.gz archives (one line per work)
└── output/  # final Parquet (or CSV) tables:
    ├── openalex_per_corpus_papers.parquet
    ├── openalex_per_corpus_authors.parquet
    └── openalex_per_corpus_authors_aggregated.parquet
```

If re-running a fetch, delete the relevant `.jsonl.gz` file first — otherwise works will be appended and duplicated.

## Notes & limitations

- Gender is **inferred from given names**, not self-reported; it is a probabilistic approximation and is retained as `unknown` when confidence is low, rather than guessed.
- Works with 100+ authors may be affected by OpenAlex's authorship list truncation.
- Publication years before `MIN_PLAUSIBLE_YEAR` (default 1950) are excluded from time-trend plots only, as likely metadata errors.
- Citation counts are raw (`cited_by_count`) in most analyses; FWCI is used separately where field/year normalization matters.

## Dependencies

`requests`, `pandas`, `numpy`, `matplotlib`, `scipy`, `pyarrow`, `global_gender_predictor`, `gender-guesser`, `jupyter`