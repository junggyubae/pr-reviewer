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
drwn card apply @junggyubae/pr-reviewer@^1.0.0
```

## What's included

| Asset | Purpose |
|---|---|
| `pr-reviewer` skill | Core review agent — intent-alignment lens, structured JSON output schema, six-criterion rubric |
| `l2-review-method` skill | Directs the agent to trace changed behavior through call graphs before emitting findings |
| `l2-use-harness-results` skill | Scopes harness output (`signals.json`, `test-results.json`) to evidence-only use |

## Versions

| Version | Notes |
|---|---|
| v1.0.0 | Initial release |

---

See the [Darwinian Tools documentation](https://docs.darwiniantools.com) for more information on drwn harness cards, installation, version pinning, and project configuration.
