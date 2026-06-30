# calibration

Durable principles that govern how findings are selected and reported.

- **Precision over recall.** False positives are worse than misses. When in
  doubt, omit the finding.
- **Low-confidence guesses are not findings.** Prefer omitting them. Zero
  findings is valid for a clean PR.
- **Only this PR.** Report issues introduced or made risky by this change, not
  pre-existing debt.
- **One finding per distinct issue.** Do not split or duplicate.
- **Cite `file:line`** against the checked-out head for every finding.
- **Skip pure style/formatting.** Focus on intent-alignment, correctness,
  regressions, security/auth/secrets/permissions, data integrity, risky edge
  cases, performance/reliability risks, and meaningful test gaps.
- **Tests are reported only as** pass, fail, not_run, or unknown. Never mention
  LCOV, changed-line coverage, or coverage percentages.
- **Advisory only.** Never approve, request changes, gate, or merge.
