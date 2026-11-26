# CLAUDE.md Corpus Study

## Project
Empirical study of `CLAUDE.md` (and `AGENTS.md`) files scraped from public GitHub repos. Independent research project mentored by Prof. Kaiser Pister (UW–Madison) for an LOR. Aim: describe what developers put in these files, from a sample of about 1,000 public repos.

## Research question
What do developers actually put in `CLAUDE.md` files? Length, structure, sections, tools mentioned, code vs. prose, and how the files talk to the agent. Descriptive only: no causal claims.

## Method
- Scrape ~1,000 repos from public GitHub via the code-search API, getting past the 1k-result cap with file-size buckets sampled in proportion to the population (code search has no star or date qualifier, so stars and dates are covariates). The sample is mostly repos with a `CLAUDE.md`, plus `AGENTS.md`-only repos as the fallback group (`primary_file` per repo; no fixed quota, they appear in their natural proportion); fetch both files when a repo has both
- Save file content + repo metadata (stars, forks, language, created_at, last_push_at, size, has_ci, has_tests)
- Dedupe, clean, descriptive stats (length, structure, section themes, tools, tone, instructions, generated boilerplate; shares with 95% CIs)

## Deliverables
- Notebooks: `01_scrape.ipynb`, `02_clean.ipynb`, `03_descriptive.ipynb`, in the public GitHub repo
- A blog post written from `03_descriptive` (headline table in `results/key_findings.csv`, figures in `figures/`)

## Tech stack
- Python 3.11+
- Data: `pandas`, Parquet, SQLite
- Scraping: `requests` + GitHub REST and code-search APIs; auth via personal access token
- Viz: `matplotlib`, `seaborn`
- Notebooks: Jupyter

## Scope guardrails
- **In scope**: only `CLAUDE.md` and `AGENTS.md` files at repo root (`AGENTS.md` added with Kaiser's approval, 2026-11-04); ~1k unique repos after dedup (confirmed with Kaiser); non-English files excluded; `AGENTS.md` is a covariate (`primary_file` / `resolved_from`), not a separate research question; descriptive analysis
- **Resolution rule (one instruction file per repo)**: `CLAUDE.md` takes priority. Go to `AGENTS.md` only when the repo has no `CLAUDE.md`, or its `CLAUDE.md` is a pointer file. Record which one was used in `resolved_from`.
- **Redirects**: if a `CLAUDE.md` points to `AGENTS.md` (symlink, `@AGENTS.md` import, or a near-empty "see AGENTS.md" pointer), follow that one hop. Store `content_raw`, `redirects_to`, and `content_resolved`; analyze on resolved content and count each resolved text once. Do not chase other imports.
- **Out of scope**: `.cursorrules`, `.windsurfrules`, `copilot-instructions.md`; web dashboard; LLM-based feature extraction (the study uses regex and rules); causal claims; fine-tuning or training a model
- **Fixed scope**: anything not listed in this doc is outside the study.

## Timeline
~7 hrs/week. Scraper and cleaned data in weeks 1–2; descriptive analysis and the first check-in with Kaiser in week 3; then the blog post.

## Working style
- Notebooks first; refactor to scripts only when a step is stable
- Checkpoint data to Parquet between every stage so no step has to re-run upstream work
- Commit after every working step; use branches for experiments
- Prefer boring, well-tested libraries over clever ones
- Document every scraping decision in `DATA_NOTES.md` as you go, not at the end
- No premature optimization — if the full pipeline runs in under an hour on 1k files, that's fine

## Ethical and legal
- Public repos only; respect GitHub rate limits
- Strip any accidentally-committed secrets (API keys, tokens) during cleaning
- `DATA_NOTES.md` documents limitations and biases (English-dominant, public-repo-only, snapshot in time)
- File contents retain their original repositories' licenses; the dataset itself is not redistributed.

## Owner
Arnav Batra — UW–Madison, triple major CS/DS/Econ, class of 2027. GitHub: TheArnavBatra. Site: arnavbatra.com.
