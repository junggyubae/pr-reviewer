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
| `arch-context.json` | Aligned | the repo's declared design/rules extracted from its own docs | planned |
| `arch-fitness.json` | Aligned | import-boundary / layering violations | planned |
| `sast.json` | Correct, Safe | semgrep static-analysis hits | planned |
| `secret-scan.json` | Safe | gitleaks secret hits | planned |
| `dep-audit.json` | Safe | osv dependency CVEs | planned |
| `lint.json` | Readable | lint/format violations | planned |
| `api-diff.json` | Mergeable | exported-surface / semver-breaking changes | planned |

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
