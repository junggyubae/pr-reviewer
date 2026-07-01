---
name: l2-intent-alignment
triggers: [pr-review]
---
Intent-alignment is your central question, so make it an explicit **first step**,
not a background feeling. Before hunting for bugs, establish what this PR is
*supposed* to do and what the repo *requires*, then judge the diff against both.

## Step 0 — run this before any other criterion

1. **Extract the claim.** From the PR title, body, and any linked issue, write a
   one-sentence statement of what this change claims to accomplish. If title and
   body disagree, or the body is empty, note that — an unclear claim is itself an
   alignment risk.
2. **Load the declared design.** Read the repo's own stated rules, in priority
   order:
   - `/repo/.pr-review/arch-context.json` when present (the harness's extracted
     design facts),
   - then `AGENTS.md`, `ARCHITECTURE.md`, `CONTRIBUTING.md`, `docs/`.
   Collect the specific rules that touch the files this PR changes.
3. **Check the diff against both.** Ask, in order:
   - **Does the change do what it claims?** Does the diff actually accomplish the
     stated intent — no more (unscoped extra changes), no less (claim unmet)?
   - **Does it fit the declared architecture?** Does it follow the conventions,
     boundaries, and rules you collected in step 2?
4. **Decide the `aligned` verdict first.** Lead your rubric's `aligned` rationale
   with the claim-vs-diff and diff-vs-architecture judgement. Only then weigh the
   other five criteria — they serve this question.

## What counts as an alignment finding

- **Claim mismatch** — the diff does not accomplish (or contradicts) its stated
  intent, or smuggles in unrelated scope. Ground it in the specific diff hunk.
- **Architecture violation** — the change breaks a rule the repo declares for
  itself. **You must quote the exact rule and its source** (`arch-context.json`
  entry, or `file:line`/section of the repo doc). No documented rule, no
  alignment finding — an unwritten preference is not a violation.
- **Silent divergence** — the change alters intended behavior the description
  never mentions. Surface it even when the code is otherwise correct.

## Discipline

- Alignment is about *fit to stated intent and declared design*, not personal
  taste. If the repo documents no rule and the change meets its claim, `aligned`
  is a pass — say so, and say why it holds.
- Distinguish "diverges from the architecture" (an alignment finding, if a rule
  is quoted) from "I would have designed it differently" (not a finding).
- A passing `aligned` still earns a one-sentence rationale explaining how the
  change matches its intent and the repo's architecture.
