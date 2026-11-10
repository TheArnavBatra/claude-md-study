# Dataset card notes (working draft)

Running log of scraping and cleaning decisions, written as we go. Becomes the HuggingFace dataset card. Numbers below are from the full run (`data/raw/full_1250.parquet`, scraped 2026-11-10) after `02_clean`. The earlier 50-repo pilot used a different sampling design (star-stratified, 25% `AGENTS.md`-only quota) and is not part of the corpus.

## What a row is
One public GitHub repo, with one resolved instruction-file text.

## Scraping decisions
- **Files in scope:** root-level `CLAUDE.md` and `AGENTS.md`. `AGENTS.md` was added with Kaiser's approval (2026-11-04). `.cursorrules`, `.windsurfrules`, `copilot-instructions.md` are out of scope for v1.
- **Resolution rule:** `CLAUDE.md` takes priority. `AGENTS.md` is used only when the repo has no `CLAUDE.md`, or its `CLAUDE.md` is a pointer file. `AGENTS.md` is a covariate in the analysis, not a separate research question (decided at check-in 1). `primary_file` records the repo's group; `resolved_from` records what the final text came from.
- **Redirects:** a `CLAUDE.md` that is a symlink to `AGENTS.md`, or is under 500 characters and mentions `AGENTS.md` (this includes the `@AGENTS.md` import), is a pure pointer and resolves to the `AGENTS.md` text. A `CLAUDE.md` with its own content that imports `@AGENTS.md` is joined with it (`CLAUDE.md+AGENTS.md`). One hop only; other imports are not followed. Symlinks come back from the contents API as tiny text files, so they are caught by the same rule.
- **Population and sampling:** GitHub code search, `filename:<file> path:/ size:<lo>..<hi>`, over 23 log-spaced file-size bins (0 B to 1 MB). Each bin's `total_count` gives the size distribution of the searchable population; candidates are allocated to bins in proportion to it, so file-length statistics are not distorted by the bucketing used to get past the 1,000-result cap. The two file types are allocated in proportion to their total counts (about 643k each, approximate), so the `AGENTS.md`-only share is natural, not a quota. Within a bin, random pages of the (at most 1,000) visible results are sampled. 1,250 candidates were drawn; 1,246 unique repos resulted.
- **No star/date stratification:** code search has no star or date qualifier, so stars and dates are covariates, not sampling strata. Consequence: the sample mirrors the real population, which is overwhelmingly low-star (see below).
- **Exclusions:** forks, archived repos, files over 1 MB, repos with neither file at the root, repos that error. In this run none of the 1,246 candidates hit any of these (code search appears to return only non-fork, non-archived repos).
- **Repo signals:** `has_tests` = a test-like directory at the root; `has_ci` = a CI config at the root.

## Cleaning decisions
- **Secrets:** regex redaction of known key formats (GitHub, AWS, Anthropic, OpenAI, Slack, Google, Stripe, JWT, private keys) and `key = "value"` assignments, replaced with `[REDACTED:<type>]`. Placeholders such as `your-api-key-here` are kept. Best effort, not a guarantee. The detector was checked on synthetic keys. Result: 2 redactions (1 OpenAI-format key, 1 assigned secret). The released data contains only the redacted text (`content_clean`), never the raw text.
- **Tiny files:** under 100 characters after stripping whitespace are dropped and logged (5).
- **Non-English:** files with over 20% non-ASCII letters are excluded and logged (103). Decided at check-in 1. This is a heuristic, not language ID.
- **Dedup:** exact (normalized hash) and near (word 5-gram Jaccard of at least 0.85); the most-starred repo is kept (12 exact duplicates, 0 near).
- **Funnel:** 1,250 candidates -> 1,246 unique repos -> 1,126 after cleaning.

## Full-run numbers (1,126 repos)
- Groups: 757 `CLAUDE.md` (67%), 369 `AGENTS.md`-only (33%). Resolved from: 641 `CLAUDE.md`, 462 `AGENTS.md`, 22 `CLAUDE.md+AGENTS.md`, 1 unresolved (a pointer whose target file is missing).
- Redirect share 10%; both files present in 29% of repos.
- Length: median 4,483 characters (614 words), mean 8,363, 90th percentile 18,017, max 228,370. `CLAUDE.md`-group median 5,005 vs 3,632 for `AGENTS.md`-only. Right-skewed, so report medians.
- Top languages: TypeScript 276, Python 253, JavaScript 99, Rust 65, Go 51.
- `has_tests` 38%, `has_ci` 52%.
- Stars: median 0, mean 20, max 5,330. Bins: 1,010 in 0-10, 91 in 10-100, 18 in 100-1000, 7 above 1000.
- Size-bin check: the sample's share per size bin matches the population share to within about 0.1 percentage points.

## Known limitations and biases
- English-dominant; public repos only; a snapshot in time (scraped 2026-11-10).
- The sample reflects the searchable population: about 90% of repos have under 10 stars and the median repo has 0. Findings describe typical repos, not popular ones. A star-balanced top-up is possible but not done.
- Code search shows at most 1,000 results per query bin, and `total_count` is approximate, so the sample is representative of the visible part of the population, not a strict random sample of GitHub.
- A repo with both files is slightly more likely to be sampled (it can be found through either search).
- No real symlink objects were observed (the API returned them as text), so symlink handling was only exercised through the text-pointer path.
- File contents keep their original repo licenses; dataset metadata is CC-BY.
