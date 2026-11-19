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
- Redirects: 10.3% of repos have a `CLAUDE.md` that redirects to `AGENTS.md`. That is 93 pure pointers (8.3%, text resolved to `AGENTS.md`) plus 22 that import `@AGENTS.md` and add their own text (joined), plus 1 pointer whose target is missing.
- Both files are present in 28.5% of repos. In 206 of them the `CLAUDE.md` has its own content, so by the resolution rule their `AGENTS.md` is not analysed. This is a known limitation, not an error.
- **`primary_file` is not a clean file-type comparison.** The `CLAUDE.md` group contains the 93 pointer repos whose text is `AGENTS.md`. Comparisons of file contents use `resolved_from` (text source) instead, leaving out the 22 joined and 1 unresolved files.
- Length: median 4,483 characters (614 words), mean 8,363, 90th percentile 18,017, max 228,370. By text source, median 647 words for `CLAUDE.md` text vs 525 for `AGENTS.md` text (594 for `CLAUDE.md` text without the `/init` default). Right-skewed, so report medians.
- Top languages: TypeScript 276, Python 253, JavaScript 99, Rust 65, Go 51.
- `has_tests` 38%, `has_ci` 52%.
- Stars: median 0, mean 20, max 5,330. Bins: 1,010 in 0-10, 91 in 10-100, 18 in 100-1000, 7 above 1000.
- Size-bin check: the sample's share per size bin matches the population share to within about 0.1 percentage points.

## Structure and tools (Phase 1, from `03_descriptive`)
All headline numbers, with 95% Wilson intervals for shares, are in `results/phase1_key_findings.csv`.
- Structure: median 8 headings and 6 level-2 sections per file; about 40% of prose lines are bullets; 1.5% of files have no headings; 26.6% contain a markdown table.
- Code: median 1 fenced code block, and 41.7% of files have none. Almost all of those still use inline code, so only 3.5% contain no code of any kind. Median code share is 6% of non-blank lines.
- Section themes (keyword rules on h1–h3 heading titles; a file can hit several): architecture and code layout 59%, commands (build/run/set up) 58%, code style and conventions 48%, rules (dos and don'ts) 44%, workflow (git/PRs/deploys) 36%, project overview 35%, testing 32%, tech stack 22%, environment and configuration 20%. 11.3% of files match none.
- Theme rules were checked against what they match and tightened twice. "Project overview" is its own theme. Bare "development" no longer counts as commands, and "gotchas"/"pitfalls" count as rules. False matches are now excluded: "context menu" (overview), commit/git conventions (style; they are workflow), bare "important" or "critical" (rules), "design system" (architecture), "dependency injection" (tech stack), "CSS variables" (environment). The notebook prints the top titles each rule matches. The rules match about half of all heading occurrences; the rest are project-specific headings.
- Among files without the `/init` default (904 files), the top four themes are close to tied: architecture 50%, commands 49%, rules 47%, style 45%.
- Anatomy (843 files with 4+ level-2/3 headings; joined texts excluded): files open with an overview (38% start with one), then commands, architecture and tech stack in the first quarter, rules near the middle, and testing, style and workflow in the second half. The order holds without `/init` files; `/init` moves commands earlier and rules later.
- Tone (prose only, the two known templates stripped): 74% of files contain a prohibition (never / don't / must not / should not / avoid) and 61% a requirement (always / must / make sure). There are 2.3 prohibition lines per requirement line, and 58% of files have more prohibitions than requirements (15% the reverse). 28% bold a directive, 15% use ALL-CAPS words like IMPORTANT, 5% use a warning emoji. 39% address the reader as "you", 27% name Claude, 58% use generic agent words (agent, assistant, AI, LLM; "AI" also catches products with AI features), 10% name two or more agent products.
- Instructions (the verb after never/don't vs always/must, counted in files): don't add 16%, use 14%, commit 12% (secrets and `.env` above all), edit 9% (generated files), create 7%, invent 5%, assume 4%. Always use 12%, run 6%, stay 6%, pass 4%, follow 4%. Shared framework templates (for example Laravel's agent guidelines) repeat lines across repos, so exact phrases are not ranked.
- Tools (keyword lexicon, matched anywhere in the file): 87% name at least one tool, median 2 per file. Most common: git 37%, npm 27%, docker 14%, MCP 14%, pytest 12%. By category: package/build 60%, version control 39%, lint/format 24%, testing 24%, infra/CI 20%. A mention is not use: "don't use npm" counts as naming npm.
- Generated text: 19.7% of files (34.5% of `CLAUDE.md` text, 1 `AGENTS.md` text) contain the default `/init` sentence "provides guidance to Claude Code". 2.4% contain the create-next-app `AGENTS.md` block ("This is NOT the Next.js you know"). "Claude Code" appears in 35% of files, but in only 19.5% of files without the `/init` default.
- **The boilerplate numbers are floors.** Only these exact phrases are detected, so other generators, older `/init` wordings, or files whose author deleted the opening sentence are missed. A flagged file can still be mostly hand-written below the default opening.
- `/init`-started files are longer (median 711 vs 594 words), have more headings (12 vs 8) and more code (19% vs 4% of lines), and nearly all have commands (92%) and architecture (95%) sections.

## Known limitations and biases
- English-dominant; public repos only; a snapshot in time (scraped 2026-11-10).
- The sample reflects the searchable population: about 90% of repos have under 10 stars and the median repo has 0. Findings describe typical repos, not popular ones. A star-balanced top-up is possible but not done.
- Code search shows at most 1,000 results per query bin, and `total_count` is approximate, so the sample is representative of the visible part of the population, not a strict random sample of GitHub.
- A repo with both files is slightly more likely to be sampled (it can be found through either search).
- One text per repo: where both files have their own content, only `CLAUDE.md` is analysed (206 repos). In 33 of these the `CLAUDE.md` tells the agent in prose to read `AGENTS.md` (a soft reference, not an import), and that `AGENTS.md` is still not followed. 17 of the 93 pointer `CLAUDE.md` files carry a few lines of their own on top of the pointer; those lines are not analysed.
- Joined texts are the `CLAUDE.md` text followed by the `AGENTS.md` text. Claude Code expands an `@AGENTS.md` import in place, so content is right but order is approximate; joined texts are left out of the section-order analysis. All 22 imports were checked to be in prose, not inside code blocks.
- Section themes, tool mentions, tone and instructions are keyword rules, not reading. A rules section titled "Notes" is missed, naming a tool is not using it, and *never* inside a quotation counts as a prohibition.
- No real symlink objects were observed (the API returned them as text), so symlink handling was only exercised through the text-pointer path.
- File contents keep their original repo licenses; dataset metadata is CC-BY.
