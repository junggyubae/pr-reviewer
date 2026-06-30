---
name: l1-harness-signals
triggers: [pr-review]
---
Before you read the diff, the harness has already computed deterministic signals
about this PR and written them to `/repo/.pr-review/signals.json`. They are facts,
not findings — they tell you **where to look harder**. Read them first and let
them direct your attention.

## signals.json schema

```json
{
  "filesChanged": 12,
  "linesAdded": 340,
  "linesRemoved": 28,
  "testToCodeRatio": 0.4,
  "newDependencies": true,
  "sensitivePaths": ["migrations/004_add_index.sql", ".github/workflows/deploy.yml"],
  "testsOrAssertionsDeleted": false,
  "hotspots": [{ "file": "src/auth/session.ts", "churn": 180 }]
}
```

## How to read each signal

- **filesChanged / linesAdded / linesRemoved** — size and shape. A large
  `linesRemoved` with small `linesAdded` is a deletion/refactor; confirm nothing
  load-bearing was dropped. Big adds in few files concentrate risk.
- **testToCodeRatio** — added test lines ÷ added non-test lines (`null` when no
  code was added). A low or `null` ratio on a behavior change is a cue to check
  for a meaningful test gap — but only report one when the changed behavior
  clearly needs coverage (see `l2-use-harness-results`).
- **newDependencies** — `true` when a manifest/lockfile gained entries. Look at
  what was pulled in: supply-chain surface, license, whether it's actually used.
- **sensitivePaths** — files matching migrations, `.github/workflows/`,
  `Dockerfile`, `.env*`, or `wrangler.*`. These carry outsized blast radius
  (data integrity, CI/CD, secrets, deploy config). Give every entry extra
  scrutiny even if the change looks small.
- **testsOrAssertionsDeleted** — `true` when test files lost lines or a test file
  was deleted. Treat as a strong cue: was coverage silently weakened to make a
  change pass? Trace which behavior is now unguarded.
- **hotspots** — the highest-churn files (add + delete), most-changed first.
  These are where bugs concentrate; start your trace here.

## Discipline

- Signals are **evidence, not findings.** `sensitivePaths` containing a migration
  is not itself a finding; a migration that drops a column without a backfill is.
- A signal never gets cited in your output. It directs where you look; the
  finding must still stand on a concrete code path (see `l2-review-method`).
- Absence of a signal is not a guarantee. A clean `signals.json` still requires
  reading the diff.
