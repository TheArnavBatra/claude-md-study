# Dataset card notes (working draft)

Running log of scraping and cleaning decisions, written as we go. Becomes the HuggingFace dataset card. Numbers below are from the 50-repo pilot (`data/raw/pilot_50.parquet`, scraped 2026-11-09), not the final corpus.

## What a row is
One public GitHub repo, with one resolved instruction-file text.

## Scraping decisions
- **Files in scope:** root-level `CLAUDE.md` and `AGENTS.md`. `AGENTS.md` was added with Kaiser's approval (2026-11-04). `.cursorrules`, `.windsurfrules`, `copilot-instructions.md` are out of scope for v1.
- **Resolution rule:** `CLAUDE.md` takes priority. `AGENTS.md` is used only when the repo has no `CLAUDE.md`, or its `CLAUDE.md` is a pointer file. `AGENTS.md` is a covariate in the analysis, not a separate research question (decided at check-in 1). `primary_file` records the repo's group; `resolved_from` records what the final text came from.
- **Redirects:** a `CLAUDE.md` that is a symlink to `AGENTS.md`, or is under 500 characters and mentions `AGENTS.md`, is a pure pointer and resolves to the `AGENTS.md` text. A `CLAUDE.md` with its own content that imports `@AGENTS.md` is joined with it (`CLAUDE.md+AGENTS.md`). One hop only; other imports are not followed.
- **Search:** GitHub code search, `filename:<name> path:/ size:<bucket>`, one query per file type and size bucket. Both searches feed one candidate pool; a repo's group comes from its actual root listing, not from which search found it.
- **Sampling:** stratified by group, then by stars (bins 0-10, 10-100, 100-1000, 1000+). The pilot used a 25% `AGENTS.md`-only share as a design parameter; for the full run (decided at check-in 1) there is no fixed quota and `AGENTS.md`-only repos appear in their natural proportion.
- **Exclusions:** forks, archived repos, files over 1 MB, repos where neither file exists at the root.
- **Repo signals:** `has_tests` = a test-like directory at the root; `has_ci` = a CI config at the root.

## Cleaning decisions
- **Secrets:** regex redaction of known key formats (GitHub, AWS, Anthropic, OpenAI, Slack, Google, Stripe, JWT, private keys) and `key = "value"` assignments, replaced with `[REDACTED:<type>]`. Placeholders such as `your-api-key-here` are kept. Best effort, not a guarantee. Pilot result: 0 secrets found in 50 files; the detector was checked on synthetic keys. The released data contains only the redacted text, never the raw text.
- **Tiny files:** under 100 characters after stripping whitespace are dropped and logged (pilot: 1 of 50).
- **Non-English:** files with over 20% non-ASCII letters (`non_english_likely`) are excluded and logged (decided at check-in 1; pilot: 1 of 49).
- **Dedup:** exact (normalized hash) and near (word 5-gram Jaccard of at least 0.85); the most-starred repo is kept. Pilot result: 0 duplicates.

## Pilot numbers (48 repos after cleaning)
- Groups: 36 `CLAUDE.md`, 12 `AGENTS.md`-only. Resolved from: 31 `CLAUDE.md`, 16 `AGENTS.md`, 1 `CLAUDE.md+AGENTS.md`.
- Redirect share 10%; both files present in 23% of repos.
- Length: median 3,831 characters (516 words), mean 12,007, 90th percentile 26,338, max 165,009. `CLAUDE.md`-group median is 5,461 vs 2,796 for `AGENTS.md`-only. The distribution is heavily right-skewed, so report medians.
- Top languages: TypeScript 9, Python 7, Rust 5, Go 5, C# 4.
- `has_tests` 56%, `has_ci` 83%.
- Stars: median 65, max 473. Bins: 6 in 0-10, 25 in 10-100, 17 in 100-1000, none above 1000.

## Known limitations and biases
- English-dominant; public repos only; a snapshot in time (scraped 2026-11-09).
- Size-bucketed search over-samples unusual file lengths, so length statistics from this pilot are not representative. The full run needs a different sampling scheme.
- Code search returns at most about 1,000 results per query, so the candidate pool is a convenience sample, not a random sample of GitHub.
- No symlinks appeared in the pilot, so the symlink path is untested on real data.
- File contents keep their original repo licenses; dataset metadata is CC-BY.
