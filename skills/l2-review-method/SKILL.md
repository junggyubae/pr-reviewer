---
name: l2-review-method
triggers: [pr-review]
---
Review by tracing changed behavior, not by restating the diff.

1. Read the PR diff.
2. Identify changed functions, exported APIs, and data contracts.
3. Read nearby callers and tests for those changed surfaces.
4. Try to prove each suspected issue from concrete code paths.
5. Emit only findings that survive that proof pass.
