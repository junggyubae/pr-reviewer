---
name: l1-harness-facts
triggers: [pr-review]
---
The harness runs a set of deterministic **fact-producers** before you review,
each writing JSON to `/repo/.pr-review/`. These are ground truth — read every
file that exists, do not re-derive them, and never cite one as a finding on its
own (a fact tells you *where* to look; the finding must stand on a code path).

**Read whichever files are present.** The producer set is per-repo configurable
(`.curation/pr-reviewer.json`), and some producers are still being built, so a
given run may emit a subset. Absence of a file is not evidence of safety.

## Fact files and the criterion each grounds

| File in `/repo/.pr-review/` | Grounds | What it carries | Status |
|---|---|---|---|
| `signals.json` | cross-cutting | churn, sensitive paths, test-to-code ratio, deleted tests, hotspots — see `l1-harness-signals` | live |
| `test-results.json` | Tested, Correct | ran? passed? command, exit code — see `l2-use-harness-results` | live |
| `typecheck.json` | Correct | `{ ran, passed, errors[] }` — type/build errors introduced by the PR | live |
| `mergeable.json` | Mergeable | `{ mergeable, mergeable_state, conflicts, failing_checks[] }` — merge state + red CI checks | live |
| `arch-context.json` | Aligned | **every `.md` document in the repo**, collected into one declared-design corpus — see below | live (agent-gathered) |
| `arch-fitness.json` | Aligned | import-boundary / layering violations | planned |
| `sast.json` | Correct, Safe | semgrep static-analysis hits | planned |
| `secret-scan.json` | Safe | gitleaks secret hits | planned |
| `dep-audit.json` | Safe | osv dependency CVEs | planned |
| `lint.json` | Readable | lint/format violations | planned |
| `api-diff.json` | Mergeable | exported-surface / semver-breaking changes | planned |

## Declared-design corpus (`arch-context`)

Do not assume a repo keeps its design in `AGENTS.md` / `ARCHITECTURE.md`. Many
repos put the load-bearing rules elsewhere (`.ai/analyses/`, `.ai/tasks/`,
`docs/`, `adr/`, package-level `README`s). So the alignment fact is gathered
**deterministically from every markdown document in the repo**, not a fixed
file list.

Gather it before reviewing:

1. **Find every `.md` / `.mdx` document** tracked in the repo. Use the repo's
   own file list so vendored/generated docs are excluded:
   ```bash
   git -C /repo ls-files '*.md' '*.mdx'
   ```
   This already omits `node_modules`, `dist`, and other git-ignored trees.
2. **Save the corpus** to `/repo/.pr-review/arch-context.json` as an array of
   `{ "path": "<repo-relative>", "content": "<full text>" }`. This is the
   deterministic declared-design signal the `aligned` criterion cites against.
3. **Prioritize when the corpus is large.** Root `*.md`, `docs/`, `.ai/`,
   `adr/`, and any file whose changed paths this PR touches come first; deep or
   peripheral docs come last. Never silently drop docs — if you cap for size,
   record which paths were not fully read.

*Interactive mode:* no harness writes the file, so gather the corpus yourself
with the `git ls-files` command above and hold it in context; the reviewer is
responsible for reading the docs, not just the fixed AGENTS/ARCHITECTURE set.

An `aligned` finding must quote a rule from this corpus by `path` + line/section
(see `finding-criteria`). A rule that is not in the corpus is not a violation.

## How facts set severity

- **Deterministic = high, no LLM judgement needed.** A `secret-scan` hit, a
  `dep-audit` CVE, a `typecheck` error, or a `sast` high — these are already
  proven. Surface them as high; do not soften or re-litigate them.
- **`mergeable.json` is advisory.** Report conflicts and `failing_checks` in the
  summary; never treat them as a merge gate — you are advisory.
- **When a deterministic linter covers a class** (e.g. `lint.json` present),
  emit no residual style findings of that class — the tool owns it.
- **Facts you have no producer for still need judgement.** If `sast.json` is
  absent, you still reason about correctness from the diff; you just can't lean
  on a proven fact.

## Discipline

- A fact is evidence, never a standalone finding. `sensitivePaths` naming a
  migration is not a finding; a migration that drops a column with no backfill is.
- Prefer the fact over your own re-derivation. If `typecheck.json` says it
  passed, do not invent type errors; if it failed, cite its errors.
