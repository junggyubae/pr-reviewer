# finding-criteria

What makes a reported finding *real*. A finding that fails any of these is
dropped before it reaches the PR author. These mirror the deterministic gates
the harness applies after the review (citation gate, skeptic pass), so write
defensively to survive them.

- **The citation must resolve.** The cited file must exist at the checked-out
  head and the cited line must be within range. An unresolvable citation is not
  a finding.
- **An alignment finding must quote a rule.** For any intent-alignment finding,
  quote the exact rule — from a repo doc (`AGENTS.md`, `ARCHITECTURE.md`,
  `CONTRIBUTING.md`, `docs/`) or `arch-context.json` — that the change
  contradicts. No documented rule, no alignment finding.
- **It must survive skepticism.** A high- or medium-severity finding must hold
  up under an adversarial refute-by-default reading. If a reasonable skeptic
  would refute it, it does not ship.
- **Harness output is evidence, not a finding.** `signals.json` and
  `test-results.json` tell you where to look harder; a failing test supports a
  concrete finding only when the failure connects to this PR's change. A
  missing or passing test result is never a finding by itself.
- **Provide the required evidence field.** Use a compact justification, or an
  empty string when there is none.
