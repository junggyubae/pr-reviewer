# Installing and running the pr-reviewer card

This card turns an agent session into an intent-aligned PR reviewer. It ships an
L1 persona, L2/L3 beliefs, and a six-skill ordered pipeline (facts →
intent-alignment → trace/prove → verify → output). Below is how to install it,
activate it in a project, and run a review.

> Requires the [`drwn` CLI](https://darwiniantools.com) (`npm i -g darwinian-minds`).

---

## 1. Install the card into your local store

Pick either source.

**From GitHub (pinned to a tag):**

```sh
drwn card clone git+https://github.com/junggyubae/pr-reviewer.git#v1.7.0
```

**From the community catalog (discoverable by name):**

```sh
drwn library catalog refresh @community
drwn search card pr-reviewer --scope @community
drwn card clone git+https://github.com/junggyubae/pr-reviewer.git#v1.7.0
```

Verify it landed:

```sh
drwn card validate @junggyubae/pr-reviewer@1.7.0
```

---

## 2. Activate it in a project

Activation is **per project** — do this inside the repo whose PRs you want to
review.

```sh
cd /path/to/your/project
drwn init --non-interactive          # only if the project has no .agents/drwn yet
drwn apply @junggyubae/pr-reviewer@^1.7.0
drwn write
```

`drwn write` materializes the six skills into `.claude/skills/` (and `.codex/`,
`.cursor/`). To let a chat session actually *behave* as the reviewer, add a
`CLAUDE.md` bridge at the project root that adopts the persona/beliefs and
defines the trigger — see [the bridge template](#claudemd-bridge-template) below.

---

## 3. Run a review

### Interactive mode (in a Claude / Codex / Cursor session)

Open an agent session rooted in the activated project and paste a PR URL:

```
Review this PR: https://github.com/OWNER/REPO/pull/123
```

The session runs the ordered pipeline and replies **chat-style**: a verdict
line, an intent-alignment-led summary, the six-criterion rubric, findings in
plain language, and the test status — with the raw JSON available on request.

### Harness mode (CI / programmatic)

When the PR head is checked out at `/repo` and the deterministic fact files are
written to `/repo/.pr-review/`, the reviewer emits **exactly one JSON object**
matching the schema in the `pr-reviewer` skill — no prose — for a bot to post as
inline comments and a check run.

---

## What you get

Six criteria, every review, led by **intent-alignment**:

| Criterion | What it checks |
|---|---|
| aligned | Does the change do what it claims, and fit the repo's declared design? |
| correct | Logic, edge cases, regressions |
| tested | Meaningful coverage of the changed behavior |
| readable | Naming, structure, no dead code |
| safe | Auth, secrets, data integrity |
| mergeable | Conflicts, CI, breaking contracts |

The reviewer is **advisory** — it emits findings only, never approves or gates —
and it is calibrated for precision over recall: zero findings on a clean PR is a
valid result.

---

## CLAUDE.md bridge template

Drop this at the project root so an interactive session activates the reviewer
on a pasted PR link:

```markdown
# PR reviewer (active mind: @junggyubae/pr-reviewer)

When the user pastes a GitHub pull request URL, adopt the reviewer and produce
one review. Otherwise behave normally.

On a pasted PR link:
1. Fetch it: `gh pr view OWNER/REPO#N --json title,body,...` and `gh pr diff OWNER/REPO#N`.
2. Run the ordered pipeline in the `pr-reviewer` skill: gather facts (including
   the declared-design corpus via `gh api .../git/trees/HEAD?recursive=1`
   filtered to `*.md`) → intent-alignment first → trace & prove → verify → output.
3. Present chat-style (verdict, summary, rubric, findings, tests); offer the raw JSON.
```

---

See the [README](./README.md) for the full layer/skill listing and the
[Darwinian Minds docs](https://docs.darwiniantools.com) for card mechanics.
