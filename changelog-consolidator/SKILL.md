---
name: changelog-consolidator
description: >-
  Use when the user has changelog documents from multiple frontend modules
  in this VitePress documentation site (each under docs/en/<module>-front/changelog.md,
  git-commit-style logs with hash/author/file/reason/impact) and wants them
  merged into ONE consolidated, high-level changelog grouped by date with a
  breakdown per module. Only consolidates frontend modules — directories
  ending in the "-front" suffix (e.g. fund-front, activity-front) — skipping
  backend, mobile, or other non-front modules even if their changelogs are
  present. Triggers: "gabungkan changelog semua module front", "buat log
  version gabungan module -front", "summarize semua module frontend jadi satu
  changelog per tanggal", or any request to merge multiple per-module
  changelogs into one condensed doc. Key trait: condensation, not
  reformatting — drop file paths, commit hashes, authors, Reason/Impact prose;
  keep only a one-line gist per change, tagged Added/Changed/Fixed/Removed/Other,
  grouped by date then module.
---

# Module Changelog Consolidator

## Purpose

Take changelog documents from every `-front` module in this VitePress
documentation site and merge them into a single Markdown document rendered as a
**three-column table** with one row per date. All modules that shipped changes
on that date appear together inside the `detail` cell, each module getting its
own block of bullets so it stays clear which module each change belongs to:

```
| date       | version | detail |
|------------|---------|--------|
| 2026-06-23 | 1.1     | fund-front<br>Added: short gist of the change<br>Changed: another change<br>activity-front<br>Fixed: short gist of a change in another module |
| 2026-03-13 | 1.2     | fund-front<br>Added: short gist of the change (V1.2) |
```

The `version` column holds a per-date release identifier assigned by the user.
Individual bullets may also carry a module-level version in parentheses at the
end (e.g. `(V1.2)`) when the source module actually has one — most modules log
by git commit and will not include a per-bullet version.

This is a **summarization** task, not a reformatting task. The point is to give
someone a bird's-eye view of "what happened across all modules, on which date,"
without making them read every module's full changelog. Never copy
implementation detail (file names, commit hashes, GitLab/GitHub links, author
names, "Reason:"/"Impact:" paragraphs) into the output — those belong in the
source documents, not in the consolidated view.

## Project Structure

This is a VitePress documentation site with locale-based routing:

```
docs/
├── en/                         ← primary locale (read from here)
│   ├── fund-front/             ← module directory (has -front suffix)
│   │   └── changelog.md        ← the changelog
│   ├── activity-front/
│   │   └── changelog.md
│   ├── credit/                 ← NOT a -front module, skip even if changelog exists
│   │   └── changelog.md
│   └── ...
├── id/                         ← Indonesian locale (skip unless no /en file exists)
│   ├── fund-front/
│   │   └── changelog.md
│   └── ...
└── .vitepress/
```

Only modules whose directory name ends with `-front` are in scope. Currently
the module directories may not yet carry the `-front` suffix; the skill is
designed for when they do. Until then, no modules will match and the
consolidation will produce an empty result — which is expected.

## Workflow

### 1. Identify the source files and their module names

This project hosts changelogs inside locale-namespaced module directories under
`docs/`. **Only ever consolidate modules whose directory name ends with
`-front`** (e.g. `fund-front`, `activity-front`). Any module directory without
the `-front` suffix (e.g. `auth`, `risk`, `lender`) is out of scope and must be
skipped, even if it has a `changelog.md` file.

Use this command to discover candidate modules from the project root:

```bash
find docs/en/ -maxdepth 1 -type d -iname '*-front' -exec test -f '{}/changelog.md' \; -print
```

Only process changelog files from the directories that command returns. The
module name is the directory name as-is (e.g. `fund-front`).

**VitePress i18n locale priority.** Every module has changelogs under both
`docs/en/<module>-front/changelog.md` and `docs/id/<module>-front/changelog.md`
(Indonesian translation). When both exist:

- Read the changelog from `docs/en/<module>-front/changelog.md` only.
- Fall back to `docs/id/<module>-front/changelog.md` for a given module ONLY if
  that module has no `/en` changelog file at all.
- Never read both locales for the same module — `/en` and `/id` are
  translations of each other, so reading both just duplicates every entry.

If a discovered path does not unambiguously belong to a `-front` module, ask
the user — do not guess silently for something that will appear in every
section header.

### 2. Read each file with the right tool

All changelog files in this project are `.md`. Read each one in full. The
structure is consistent across modules:

- `## [<commit-hash>] - YYYY-MM-DD` — dated commit block header
- `### Added` / `### Changed` / `### Fixed` / `### Removed` — category sections
- Each bullet under a category describes a single change with `(Title)`,
  `File:`, `Changes:`, `Reason:`, `Impact:` sub-fields
- A trailing block with `Author:`, `Source:`, `GitLab Commit:` link

### 3. Extract raw entries

For every commit block in a source file, pull out:

- the **date** as written (from the `## [hash] - YYYY-MM-DD` header)
- the **category** (`Added`, `Changed`, `Fixed`, `Removed`)
- the **substance of the change** — the title in parentheses at the start of
  each bullet, plus the first sentence of the `Changes:` field

Ignore everything that is purely bookkeeping: commit hashes, author emails,
GitLab commit links, `File:` paths, `Reason:` paragraphs, `Impact:` paragraphs,
and the metadata footer block.

### 4. Normalize every date to `YYYY-MM-DD`

The project already uses `YYYY-MM-DD` dates in commit headers, so minimal
normalization is needed. Watch for:

- Indonesian month names in free-form text (`Januari`, `Februari`, `Maret`,
  `April`, `Mei`, `Juni`, `Juli`, `Agustus`, `September`, `Oktober`, `November`,
  `Desember`)
- Two-digit years (`-23` → `2023`, `-24` → `2024`)
- Entries with no real date — use your best judgment to place them near
  neighboring dated entries; never invent a fake precise date.

### 5. Condense each entry into a category + a one-line description

Be aggressive about cutting detail, but do not lose the "what changed" meaning.
Pick a category based on the gist of the original wording:

- **Added** — new feature, new field, new integration, new page/component
- **Changed** — modified behavior, updated wording/config/UI, migration, refactor
- **Fixed** — bug fix, correction of broken behavior, typo fix
- **Removed** — deleted/deprecated something
- **Other** — anything that does not cleanly fit

**Example — this project's changelog format → condensed output:**

> Input (from `docs/en/fund-front/changelog.md`):
>
> ```
> ### Changed
> - (Capital Flow Config) Added payout channel selection column and refactored CSS syntax
>   - File: `src/views/capital/flow/config.vue`
>   - Changes: Added `payoutChannelId` column with select dropdown. Added `getPayoutChannelList()` method to load channel data. Changed CSS syntax from `/deep/` to `::v-deep` for Vue 3 compatibility.
>   - Reason: Flow configuration needs to display and select payout channels. `/deep/` syntax is deprecated in Vue 3.
>   - Impact: Users can select payout channels when editing flow configuration. CSS is compatible with Vue 3.
> ```

> Condensed: category `Changed`, description `Capital flow config: added payout channel selection column, migrated /deep/ to ::v-deep for Vue 3 compatibility`

> Input:
>
> ```
> ### Fixed
> - (Translation) Fixed typo in English translation for loan failure status
>   - File: `src/i18n/en.ts`
>   - Change: Changed value of key `channel.loanFailure` from "Loss Failed" to "Loan Failed"
>   - Reason: A typo caused the word "Loss" to be used instead of "Loan"
> ```

> Condensed: category `Fixed`, description `English translation: corrected "Loss Failed" → "Loan Failed" typo for loan failure key`

Notice that the file name, reason paragraph, author, hash, and link are all
dropped entirely. Only the title in parentheses and the core change description
survive.

### 6. Build a structured entries list

Collect everything into a simple list, one item per condensed change. Group
entries by date since each date row in the output table gets a single version:

```json
[
  {
    "date": "2026-06-23",
    "version": "1.1",
    "module": "fund-front",
    "category": "Changed",
    "description": "Capital flow config: added payout channel selection column, migrated /deep/ to ::v-deep for Vue 3 compatibility"
  },
  {
    "date": "2026-06-23",
    "version": "1.1",
    "module": "activity-front",
    "category": "Fixed",
    "description": "SSO proxy helper: removed hardcoded zh-CN locale from error log timestamps"
  },
  {
    "date": "2026-03-13",
    "version": "1.2",
    "module": "fund-front",
    "category": "Added",
    "description": "Pinned Node.js version via .nvmrc (v14.21.3) for consistent dev/CI builds"
  }
]
```

Save this as `entries.json` in the working directory for reference and reuse.

### 7. Render the final document

Group entries by date (newest first). Each date becomes one table row with
three columns: `date`, `version`, and `detail`.

Inside the `detail` cell, group entries by module. Each module gets its name
on its own line, followed by its bullets on subsequent lines. Use `<br>` tags
for line breaks inside the table cell so the Markdown renders correctly:

```
| date       | version | detail |
|------------|---------|--------|
```

Write the output to `consolidated-changelog.md` at the project root:

```markdown
# Consolidated Changelog

| date       | version | detail                                                                                                                                                                                                                                                                                                                                          |
| ---------- | ------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 2026-06-23 | 1.1     | fund-front<br>Changed: Capital flow config: added payout channel selection column, migrated /deep/ to ::v-deep for Vue 3 compatibility<br>Removed: CI/CD pipeline: dropped all Jenkins configuration files after migration to new system<br>activity-front<br>Fixed: SSO proxy helper: removed hardcoded zh-CN locale from error log timestamps |
| 2026-03-13 | 1.2     | fund-front<br>Added: Pinned Node.js version via .nvmrc (v14.21.3) for consistent dev/CI builds (V1.2)<br>Changed: Header language UX: added direct click trigger on language label                                                                                                                                                              |
```

Rules for the `detail` cell:

- Each module block starts with the module name on its own line (no prefix, no
  bold).
- The module name is followed by its bullets, one per line, each starting with
  `- ` removed and using the `Category: description` pattern.
- Different modules inside the same `detail` cell are separated by a line break
  (the next module name appears directly after the previous module's last
  bullet — no blank line between modules inside the table cell).
- Use `<br>` for every line break within the cell so the raw Markdown table
  stays parseable.

If the user wants oldest-first ordering, reverse the row order so the earliest
date appears first.

### 8. Deliver the result

Present the consolidated output to the user and briefly mention how many source
files and modules were merged and the date range covered. If a module's
changelog used a format the workflow above did not anticipate, say so
explicitly rather than silently dropping entries. If the user asks for `.docx`
or `.pdf`, convert the Markdown output using the **docx** or **pdf** skill
rather than hand-rolling a new format.

## Notes

- If the user adds new modules later, re-run from step 1 with just the new
  files plus the previous `entries.json` (append, do not redo everything).
- If two or more modules changed on the exact same date, each appears as its
  own name-line + bullet block inside the same `detail` cell — never merge
  different modules' bullets into one undifferentiated list.
- Keep each bullet to roughly one sentence. If a module had many small related
  commits on one date, it is fine to merge them into one bullet rather than
  listing ten near-duplicate lines.
- When the module directories do not yet have the `-front` suffix, the
  consolidation will return zero modules. That is correct and expected behavior
  — the skill is designed to activate once the suffix convention is applied.
