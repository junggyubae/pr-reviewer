---
name: pr-reviewer
triggers: [pr-review]
---
The procedural contract for producing one PR review. Your identity and stance
come from the `reviewer` persona; your selection and reporting rules come from
the `calibration` and `finding-criteria` beliefs. This skill covers only *how
to gather evidence and shape the output*.

## Environment
- The PR head is checked out at `/repo`. The change is the diff `base..head`.
- Run `git -C /repo diff <base> <head>` to see the change; read any file in `/repo`.
- For intent-alignment, read the repo's own declared design: `AGENTS.md`,
  `ARCHITECTURE.md`, `CONTRIBUTING.md`, `docs/`, and (when present)
  `/repo/.pr-review/arch-context.json`.
- Harness evidence: `/repo/.pr-review/signals.json`,
  `/repo/.pr-review/test-results.json`.

## Output rules
- Tag each finding with its rubric `criterion`.
- In `rubric`, assess all six criteria (correct, tested, aligned, readable, safe,
  mergeable). For each give a `status` (pass | concern | not_evaluated) and a
  one-sentence `rationale`. **Even when a criterion passes, briefly explain why it
  holds and why it is strong — not just that it passed.** Lead the `aligned`
  rationale with how the change matches (or diverges from) its stated intent and
  the repo's architecture.
- Output exactly one JSON object matching the schema. No prose outside JSON.

## Output Schema

Return:

```json
{
  "summary": "1-3 sentence PR-level summary, leading with intent-alignment",
  "test_summary": "tests pass|tests fail|tests not_run|tests unknown plus one short reason",
  "findings": [
    {
      "file": "path/from/repo/root.ts",
      "line": 123,
      "severity": "high|medium|low",
      "confidence": "high|medium|low",
      "category": "bug|edge_case|security|test_gap|maintainability|style_rule|performance|docs_mismatch",
      "criterion": "correct|tested|aligned|readable|safe|mergeable",
      "title": "short finding title",
      "detail": "what is wrong, grounded in the diff",
      "impact": "why it matters to users/runtime/security",
      "suggested_direction": "specific fix direction",
      "evidence": "compact evidence, or an empty string"
    }
  ],
  "suggested_followups": ["optional concise follow-up"],
  "rubric": [
    {
      "criterion": "correct|tested|aligned|readable|safe|mergeable",
      "status": "pass|concern|not_evaluated",
      "rationale": "why it holds and why it is strong (or the concern)"
    }
  ]
}
```
