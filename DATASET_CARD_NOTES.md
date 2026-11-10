# Dataset card notes (working draft)

Running log of scraping and cleaning decisions, written as we go. Becomes the HuggingFace dataset card. Numbers below are from the 50-repo pilot (`data/raw/pilot_50.parquet`, scraped 2026-11-09), not the final corpus.

## What a row is
One public GitHub repo, with one resolved instruction-file text.

## Scraping decisions
- **Files in scope:** root-level `CLAUDE.md` and `AGENTS.md`. `AGENTS.md` was added with Kaiser's approval (2026-11-04). `.cursorrules`, `.windsurfrules`, `copilot-instructions.md` are out of scope for v1.
- **Resolution rule:** `CLAUDE.md` takes priority. If the repo has no `CLAUDE.md`, its `AGENTS.md` is used. `primary_file` records the repo's group; `resolved_from` records what the final text came from.
- **Redirects:** a `CLAUDE.md` that is a symlink to `AGENTS.md`, or is under 500 characters and mentions `AGENTS.md`, is a pure pointer and resolves to the `AGENTS.md` text. A `CLAUDE.md` with its own content that imports `@AGENTS.md` is joined with it (`CLAUDE.md+AGENTS.md`). One hop only; other imports are not followed.
- **Search:** GitHub code search, `filename:<name> path:/ size:<bucket>`, one query per file type and size bucket. Both searches feed one candidate pool; a repo's group comes from its actual root listing, not from which search found it.
- **Sampling:** stratified by group, then by stars (bins 0-10, 10-100, 100-1000, 1000+). The `AGENTS.md`-only share is a design parameter (25% in the pilot), not a measured share of GitHub. To confirm with Kaiser.
- **Exclusions:** forks, archived repos, files over 1 MB, repos where neither file exists at the root.
- **Repo signals:** `has_tests` = a test-like directory at the root; `has_ci` = a CI config at the root.

## Cleaning decisions
- **Secrets:** regex redaction of known key formats (GitHub, AWS, Anthropic, OpenAI, Slack, Google, Stripe, JWT, private keys) and `key = "value"` assignments, replaced with `[REDACTED:<type>]`. Placeholders such as `your-api-key-here` are kept. Best effort, not a guarantee. Pilot result: 0 secrets found in 50 files; the detector was checked on synthetic keys. The released data contains only the redacted text, never the raw text.
- **Tiny files:** under 100 characters after stripping whitespace are dropped and logged (pilot: 1 of 50).
- **Non-English:** flagged (`non_english_likely`, over 20% of letters non-ASCII), not dropped (pilot: 1 of 49). Whether to exclude them is open.
- **Dedup:** exact (normalized hash) and near (word 5-gram Jaccard of at least 0.85); the most-starred repo is kept. Pilot result: 0 duplicates.

## Pilot numbers (49 repos after cleaning)
- Groups: 37 `CLAUDE.md`, 12 `AGENTS.md`-only. Resolved from: 31 `CLAUDE.md`, 16 `AGENTS.md`, 2 `CLAUDE.md+AGENTS.md`.
- Redirect share 12%; both files present in 24% of repos.
- Length: median 3,854 characters (523 words), mean 11,918, 90th percentile 26,218, max 165,009. `CLAUDE.md`-group median is 5,619 vs 2,796 for `AGENTS.md`-only. The distribution is heavily right-skewed, so report medians.
- Top languages: TypeScript 9, Python 7, Rust 6, Go 5, C# 4.
- `has_tests` 57%, `has_ci` 84%.
- Stars: median 65, max 473. Bins: 6 in 0-10, 25 in 10-100, 18 in 100-1000, none above 1000.

## Known limitations and biases
- English-dominant; public repos only; a snapshot in time (scraped 2026-11-09).
- Size-bucketed search over-samples unusual file lengths, so length statistics from this pilot are not representative. The full run needs a different sampling scheme.
- Code search returns at most about 1,000 results per query, so the candidate pool is a convenience sample, not a random sample of GitHub.
- No symlinks appeared in the pilot, so the symlink path is untested on real data.
- File contents keep their original repo licenses; dataset metadata is CC-BY.
