# CLAUDE.md Corpus Study

An empirical study of `CLAUDE.md` files in public GitHub repositories. The goal is the first public dataset and taxonomy of these files, plus an analysis of how their contents relate to repo characteristics.

Independent research project by Arnav Batra (UW–Madison), mentored by Prof. Kaiser Pister.

## Research questions
1. **Descriptive:** What do developers put in `CLAUDE.md` files (length, structure, sections, tools, code vs. prose)?
2. **Structural:** Do files cluster into recognizable styles?
3. **Correlational:** Do content patterns line up with repo characteristics (language, activity, test presence)? These are associations, not causal claims.

## Method (summary)
Scrape ~1,000 root-level `CLAUDE.md` files via the GitHub API, collect repo metadata, dedupe and clean, embed and cluster the files, extract rule-based features, then join with repo metadata and run regressions.

## Notebooks
| Notebook | Phase |
|---|---|
| `01_scrape.ipynb` | Scraping files and repo metadata |
| `02_clean.ipynb` | Dedup, cleaning, secret stripping |
| `03_descriptive.ipynb` | Descriptive statistics |
| `04_clusters.ipynb` | Embeddings and clustering |
| `05_features.ipynb` | Structured feature extraction |
| `06_correlations.ipynb` | Joins, cross-tabs, regressions |

## Status
Planning. See [CLAUDE.md](CLAUDE.md) for the full project brief, scope, and timeline.

## Data
The dataset will be released on HuggingFace as `claude-md-corpus` with a dataset card documenting scraping decisions, limitations, and biases (English-dominant, public repos only, a snapshot in time).

## Ethics and licensing
- Public repositories only; GitHub rate limits respected.
- Accidentally committed secrets are stripped during cleaning.
- File contents retain their original repository licenses; dataset metadata is released under CC-BY.

## Author
Arnav Batra · [arnavbatra.com](https://arnavbatra.com) · [@TheArnavBatra](https://github.com/TheArnavBatra)
