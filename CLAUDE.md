# CLAUDE.md Corpus Study

## Project
Empirical study of `CLAUDE.md` files scraped from public GitHub repos. Independent research project mentored by Prof. Kaiser Pister (UW–Madison) for an LOR. Aim: build the first public dataset and taxonomy of these files, and look at how patterns in them relate to repo characteristics.

## Research questions
1. **Descriptive** — what do developers actually put in `CLAUDE.md` files? Length, structure, sections, tools mentioned, code vs. prose.
2. **Structural** — do files cluster into recognizable styles (build cheat sheet, architecture overview, style enforcer, persona setter, guardrails, etc.)? Expect ~10–20 clusters.
3. **Correlational** — do specific patterns line up with repo characteristics (language, activity, test presence)? Framed as associations, not causes.

## Method
- Scrape ~1,000 `CLAUDE.md` files from public GitHub via the code-search API, paginated past the 1k-result cap by star/date buckets
- Save file content + repo metadata (stars, forks, language, created_at, last_push_at, size, has_ci, has_tests)
- Dedupe, clean, basic descriptive stats
- Embed each file (start with sentence-transformers `all-MiniLM-L6-v2`; fall back to OpenAI `text-embedding-3-small` if clusters are weak)
- Cluster with HDBSCAN (primary) and k-means (robustness check); hand-label clusters
- Extract structured features per file: `has_persona`, `has_donts`, `has_commands`, `has_file_paths`, `mentioned_tools`, section counts, code-block counts
- Join file features with repo metadata; cross-tabulate; run regressions (OLS for continuous outcomes, logistic for binary) with controls for language, repo age, size

## Deliverables
- Public dataset on HuggingFace (`claude-md-corpus`) with a proper dataset card
- Six analysis notebooks in a public GitHub repo, one per phase:
  - `01_scrape.ipynb`
  - `02_clean.ipynb`
  - `03_descriptive.ipynb`
  - `04_clusters.ipynb`
  - `05_features.ipynb`
  - `06_correlations.ipynb`
- Writeup on personal site (arnavbatra.com) summarizing key findings

## Tech stack
- Python 3.11+
- Data: `pandas`, Parquet, SQLite
- Scraping: `requests` + GitHub REST and code-search APIs; auth via personal access token
- NLP: `sentence-transformers`, optionally `openai` embeddings
- Clustering: `hdbscan`, `scikit-learn` (k-means), `umap-learn` for 2D viz
- Stats: `statsmodels` for regressions, `scipy.stats` for tests
- Viz: `matplotlib`, `seaborn`, `plotly`
- Release: `datasets` library for HuggingFace upload
- Notebooks: Jupyter

## Scope guardrails
- **In scope v1**: only `CLAUDE.md` files at repo root; ~1k unique files after dedup; descriptive + correlational analysis
- **Out of scope v1**: `.cursorrules`, `AGENTS.md`, `.windsurfrules`, `copilot-instructions.md`; web dashboard; LLM-based feature extraction (regex and rules first); causal claims; fine-tuning or training a model
- **Hard no**: scope creep. If a feature isn't in this doc, it's a v2 candidate, not a v1 addition.

## Timeline
~7 hrs/week over 10 weeks (~70 hrs total). Phases:
1. Weeks 1–2: scraper + cleaned data
2. Week 3: first check-in with Kaiser
3. Weeks 4–5: embedding, clustering, feature extraction
4. Week 6: repo metadata joined
5. Week 7: second check-in with Kaiser
6. Week 8: correlational analysis
7. Week 9: writeup + HuggingFace release
8. Week 10: buffer + third check-in

## Working style
- Notebooks first; refactor to scripts only when a step is stable
- Checkpoint data to Parquet between every stage so no step has to re-run upstream work
- Commit after every working step; use branches for experiments
- Prefer boring, well-tested libraries over clever ones
- Document every scraping decision in the dataset card as you go, not at the end
- No premature optimization — if the full pipeline runs in under an hour on 1k files, that's fine

## Ethical and legal
- Public repos only; respect GitHub rate limits
- Strip any accidentally-committed secrets (API keys, tokens) during cleaning
- Dataset card documents limitations and biases (English-dominant, public-repo-only, snapshot in time)
- File content retains original repo licenses; dataset metadata released CC-BY

## Owner
Arnav Batra — UW–Madison, triple major CS/DS/Econ, class of 2027. GitHub: TheArnavBatra. Site: arnavbatra.com.
