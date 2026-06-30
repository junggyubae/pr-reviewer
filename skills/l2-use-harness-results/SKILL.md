---
name: l2-use-harness-results
triggers: [pr-review]
---
Use harness output as evidence, not as findings by itself.

- `signals.json` highlights where to look harder.
- `test-results.json` says whether tests ran and whether they passed.
- Use failing tests as evidence for concrete findings when the failure connects
  to the PR change.
- Do not mention LCOV, coverage percentages, or changed-line coverage.
- A missing or passing test result is not a finding by itself; it can support a
  test-gap finding only when the changed behavior clearly needs coverage.
