---
name: pr-reviewer
triggers: [pr-review]
---
The procedural contract for producing one PR review. Your identity and stance
come from the `reviewer` persona; your selection and reporting rules come from
the `calibration` and `finding-criteria` beliefs. This skill orchestrates the
review pipeline and defines the output.

## Review pipeline — execute in this order

This mirrors the target architecture's spine: **facts → grounded reviewer →
verification → output.** Run the stages in order; do not emit findings before
the earlier stages have run.

1. **Gather facts** — `l1-harness-facts` (read every fact file present in
   `/repo/.pr-review/`) and `l1-harness-signals` (the `signals.json` detail).
   Facts are ground truth and tell you where to look. *Interactive mode:* the
   fact files may not exist — derive the equivalent cues from the diff instead.
2. **Intent-alignment (step 0)** — `l2-intent-alignment`: state the PR's claim,
   load the declared design, judge the diff against both, and decide the
   `aligned` verdict first. This is the central question; every later criterion
   serves it.
3. **Trace and prove** — `l2-review-method`: trace changed behavior through
   callers and tests; prove each suspected issue from a concrete code path.
   Assess the six criteria in order — **correct, tested, aligned, readable,
   safe, mergeable** — grounding each finding in a fact (stage 1) or a citation.
4. **Use evidence correctly** — `l2-use-harness-results`: facts are evidence,
   not findings on their own.
5. **Verify before emitting** — self-apply `finding-criteria`: drop any finding
   whose citation does not resolve, any alignment finding that does not quote a
   real repo rule, and any high/medium finding that would not survive an
   adversarial refute-by-default reading.
6. **Shape the output** — emit exactly one JSON object per the schema below:
   the per-criterion `rubric` (all six) plus the surviving `findings`.

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

## Presentation by mode

The review content is the same; the wrapper differs by caller.

- **Harness / CI mode** (the PR head is checked out at `/repo`, or a machine is
  parsing your output): emit **exactly one JSON object** matching the schema
  below, with no prose before or after it.
- **Interactive / chat mode** (a person asked you to review a PR link): lead with
  a **chat-style summary**, not raw JSON. Render the same review readably:
  1. **Verdict line** — one sentence: overall alignment + finding count
     (e.g. "Strongly aligned; 1 medium finding, docs-only").
  2. **Summary** — 1-3 sentences, leading with intent-alignment.
  3. **Rubric at a glance** — the six criteria with `pass` / `concern` /
     `not_evaluated` and a short reason each.
  4. **Findings** — each as a readable item: severity + title, why it matters,
     and the suggested direction. Say "no findings" plainly when clean.
  5. **Tests** — one line (pass / fail / not_run / unknown).
  Then offer the raw JSON ("say the word for the JSON") or append it under a
  collapsed/last section — never lead with it, never omit the summary.

## Output Schema (canonical form for both modes)

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
