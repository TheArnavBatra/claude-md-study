# CLAUDE.md Corpus Study

An empirical study of `CLAUDE.md` and `AGENTS.md` files in public GitHub repositories. The goal is the first public dataset and taxonomy of these files, plus an analysis of how their contents relate to repo characteristics.

Independent research project by Arnav Batra (UW–Madison), mentored by Prof. Kaiser Pister.

## Research questions
1. **Descriptive:** What do developers put in `CLAUDE.md` files (length, structure, sections, tools, code vs. prose)?
2. **Structural:** Do files cluster into recognizable styles?
3. **Correlational:** Do content patterns line up with repo characteristics (language, activity, test presence)? These are associations, not causal claims.

## Method (summary)
Scrape ~1,000 repos via the GitHub API, using the root `CLAUDE.md` (or the `AGENTS.md` when there is no `CLAUDE.md`, following `CLAUDE.md` → `AGENTS.md` redirects), collect repo metadata, dedupe and clean, embed and cluster the files, extract rule-based features, then join with repo metadata and run regressions.

## Phases and notebooks
The project ships in two phases, each publishable on its own.

| Phase | Notebook | What it does | Status |
|---|---|---|---|
| 1 | `01_scrape.ipynb` | Sample repos via code search; fetch files and repo metadata | Done |
| 1 | `02_clean.ipynb` | Resolve `CLAUDE.md` → `AGENTS.md` pointers, strip secrets, filter, dedup | Done |
| 1 | `03_descriptive.ipynb` | Length, structure, section themes, tools, tone, instructions, generated boilerplate | Done |
| 2 | `04_clusters.ipynb` | Embeddings and clustering into file styles | Not started |
| 2 | `05_features.ipynb` | Structured feature extraction | Not started |
| 2 | `06_correlations.ipynb` | Joins, cross-tabs, regressions | Not started |

**Phase 1** (research question 1) builds the corpus and describes it. Its writeup is a blog post that uses only `03_descriptive`: the headline numbers are in [`results/phase1_key_findings.csv`](results/phase1_key_findings.csv) and the figures in [`figures/`](figures/). **Phase 2** (research questions 2 and 3) adds clusters, features and correlations, plus the HuggingFace dataset release.

## Status
Phase 1 notebooks are done on a snapshot of 1,126 repos (scraped 2026-11-10). The Phase 1 blog post is next. See [CLAUDE.md](CLAUDE.md) for the full project brief, scope, and timeline, and [DATASET_CARD_NOTES.md](DATASET_CARD_NOTES.md) for every scraping and cleaning decision.

## Data
Raw and intermediate data (`data/`) are not committed: they contain file text under the original repos' licenses. The dataset will be released on HuggingFace in Phase 2 as `claude-md-corpus` with a dataset card documenting scraping decisions, limitations, and biases (English-dominant, public repos only, a snapshot in time).

## Ethics and licensing
- Public repositories only; GitHub rate limits respected.
- Accidentally committed secrets are stripped during cleaning.
- File contents retain their original repository licenses; dataset metadata is released under CC-BY.

## Author
Arnav Batra · [arnavbatra.com](https://arnavbatra.com) · [@TheArnavBatra](https://github.com/TheArnavBatra)
