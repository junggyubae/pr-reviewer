# pr-reviewer

> Intent-aligned PR reviewer — surfaces only findings grounded in the PR's own stated goals and the repo's architecture.

## What it does

- Reviews PRs through an intent-alignment lens: does the change accomplish what its title/description claim, and does it fit the repo's declared architecture?
- Traces changed behavior through call graphs and tests before emitting findings — eliminates low-confidence noise
- Interprets harness signals (`signals.json`, `test-results.json`) as evidence for concrete findings, not standalone results

## Recommended for users who...

- Run automated PR review in a CI harness (head checked out at `/repo`, harness output in `/repo/.pr-review/`)
- Want precision over recall — zero findings for a clean PR is a valid, useful output
- Need structured JSON review output with per-criterion rubric assessment

## Installation

> Requires the [drwn CLI](https://darwiniantools.com).

Clone the card to your local store:

```sh
drwn card clone github:junggyubae/pr-reviewer#v1.0.0
```

If this is a new project, run `drwn init` first.

Apply the card:

```sh
drwn card apply @junggyubae/pr-reviewer@^1.4.0
```

## What's included

**Mind layers**

| Layer | Entry | Purpose |
|---|---|---|
| L1 persona | `reviewer` | Identity: senior reviewer, intent-alignment lens, advisory-only |
| L2 belief | `calibration` | Selection/reporting principles — precision over recall, this-PR-only |
| L3 belief | `finding-criteria` | What makes a finding real — citation resolves, alignment quotes a rule, survives skeptic |

**Skills**

| Skill | Purpose |
|---|---|
| `pr-reviewer` | Output contract — JSON schema + six-criterion rubric |
| `l2-intent-alignment` | Step 0: extract claim → load declared design → judge diff against both |
| `l2-review-method` | Trace changed behavior and prove each issue before emitting |
| `l2-use-harness-results` | Treat harness output as evidence, not findings |
| `l1-harness-signals` | Read `signals.json` field-by-field |
| `l1-harness-facts` | The full deterministic fact set (`typecheck`, `mergeable`, `sast`, `secret-scan`, `dep-audit`, `arch-context`, …) and the criterion each grounds |

## Versions

| Version | Notes |
|---|---|
| v1.4.0 | Add `l2-intent-alignment` (direct alignment step) + `l1-harness-facts` (full fact set); persona leads with alignment |
| v1.3.0 | Add `l1-harness-signals` |
| v1.2.0 | Decompose into L1 persona + L2/L3 beliefs |
| v1.0.0 | Initial 3-skill bundle |

---

See the [Darwinian Tools documentation](https://docs.darwiniantools.com) for more information on drwn harness cards, installation, version pinning, and project configuration.
