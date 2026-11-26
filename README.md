# CLAUDE.md Corpus Study

An empirical study of `CLAUDE.md` and `AGENTS.md` files in public GitHub repositories: what developers write in the instruction files they give to AI coding agents.

Independent research project by Arnav Batra (UW–Madison), mentored by Prof. Kaiser Pister.

## Research question
What do developers put in `CLAUDE.md` files? Length, structure, sections, tools mentioned, code vs. prose, and how the files talk to the agent. The study is descriptive: it reports what is in the files, not whether any of it works, and makes no causal claims.

## Method (summary)
1. **Sampling.** Public repos found through GitHub code search, split into 23 file-size bins so the search can get past its 1,000-result cap, and sampled from each bin in proportion to its share of the searchable population.
2. **Resolution.** One text per repo. `CLAUDE.md` takes priority, and `AGENTS.md` is used when a repo has no `CLAUDE.md` or its `CLAUDE.md` is a pointer to `AGENTS.md`.
3. **Cleaning.** Accidentally committed secrets are stripped, tiny and non-English files are filtered out, and exact and near duplicates are removed.
4. **Descriptive analysis.** Length, structure, section themes, tools, tone, instructions and template-generated text. Shares carry 95% confidence intervals.

## Notebooks
| Notebook | What it does |
|---|---|
| `01_scrape.ipynb` | Sample repos via code search; fetch files and repo metadata |
| `02_clean.ipynb` | Resolve `CLAUDE.md` → `AGENTS.md` pointers, strip secrets, filter, dedup |
| `03_descriptive.ipynb` | Length, structure, section themes, tools, tone, instructions, generated boilerplate |

The headline numbers are in [`results/key_findings.csv`](results/key_findings.csv) and the figures in [`figures/`](figures/).

## Status
The study is complete on a snapshot of 1,126 repos scraped 2026-11-10. See [CLAUDE.md](CLAUDE.md) for the project brief and scope, and [DATA_NOTES.md](DATA_NOTES.md) for every scraping and cleaning decision.

## Data
Raw and intermediate data (`data/`) are not committed, because the file text stays under the original repos' licenses.

## Ethics and licensing
- Public repositories only; GitHub rate limits respected.
- Accidentally committed secrets are stripped during cleaning.
- File contents retain their original repositories' licenses; the dataset itself is not redistributed.

## Author
Arnav Batra · [arnavbatra.com](https://arnavbatra.com) · [@TheArnavBatra](https://github.com/TheArnavBatra)
