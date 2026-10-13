# AGENTS.md — action-review

The GitHub Actions config for OpenCharly's AI PR-review gate — the replacement
for the retired `opencharly/pi-review-action`, built on charly + the
[plugin-review](https://github.com/opencharly/plugin-review) plugin only. The
gate runs the welded `charly review` word on a standard GitHub runner by default
(self-hosted opt-in via `vars.REVIEW_RUNNER_LABEL`), configured entirely from
`charly.yml` — the rulebook in force being the org variable `AI_REVIEW_PROMPT`.

Canonical files:

- `charly.yml` — the review **contract**: the `review-contract:` candy (the
  `AI_REVIEW_*` `var:` surface + the committed `AI_REVIEW_PROMPT` default, composing
  `plugin-review`) and the `review-runner:` candy (the `enabled: false`
  self-hosted image definition).
- `.github/workflows/ai-review.yml` — the gate itself; `.github/workflows/ci.yml`
  — `charly box validate` + the retired-mechanism absence gates + actionlint;
  `.github/workflows/tag-on-merge.yml` — CalVer tag + `CHANGELOG/` on merge.
- `docs/action-review.md` — the env/secret table, deploy, and cutover notes.
- `README.md` — user overview only; never agent guidance.

The repo carries no `skill:` entity, so no owning `/charly-<family>:<name>` skill
is projected into the marketplace corpus.

## Load these skills first (R0)

- `/charly-internals:repo-setup` — the org ruleset, the org-required
  `charly/pr-validator` workflow, native auto-merge, and tag-on-merge CalVer; this
  repo IS the org gate config.
- `/charly-internals:git-workflow` — before any git/PR action.
- `/charly-review:review` — the PR-review engine the gate drives.

There is no dedicated `/charly-*:action-review` owning skill — the gap (no owning
skill projected) is recorded against `opencharly/opencharly#291`.

## Build / validate / test

- `.github/workflows/ci.yml` is the config gate, and it runs **ONE engine — the org pin**:
  it installs the release `vars.CHARLY_VERSION` names (the same release the review gate
  runs), asserts that release's binary reports the version the pin names, and then runs
  `charly box validate` with **that** binary. It also asserts the retired `--plan`
  mechanism is ABSENT (`review-plan.yml`, `prompt/validator.md`, and any
  `REVIEW_PLAN_PATH`/`REVIEW_PROMPT_PATH` in `charly.yml`), asserts the contract still
  declares `AI_REVIEW_PROMPT`, and lints the workflows with `actionlint`. So the pin the
  review gate enforces is verified **and** is what validates the contract — a config gate
  green for an engine no gate runs is the `opencharly/.github#126` defect.
  It used to install the LATEST release for `box validate` while only verifying the pin,
  because the pin of that day (`v2026.271.0950`) predated the config-stamp retirement
  (`charly#754` / umbrella `#340`) and rejected the stamp-less contract. The pin has since
  advanced past that retirement — measured 2026-10-13, `v2026.278.0912` reports
  `charly box validate: OK — checked 5 candies, 7 distros, 7 builders; 0 warnings, 0 errors`
  on this tree — so the workaround is gone and the divergence is closed
  (`opencharly/.github#162`).
- The gate itself is `.github/workflows/ai-review.yml`. It is configured entirely by
  the `AI_REVIEW_*` vars: it runs the org-pinned engine and forwards the live org
  variable `AI_REVIEW_PROMPT`, so this check and the org-wide required gate review
  with ONE rulebook (`opencharly/.github#126`).
- The merge gate is the **org-wide** `charly/pr-validator` (required check
  `validate / validate`, defined in `opencharly/.github`).

## Modify this repo

- Every `AI_REVIEW_*` value is declared in `charly.yml` at its currently-used
  production default (API keys are secrets, never committed); the opencharly org
  GitHub settings can override every value, since each name is an org VARIABLE
  (`AI_REVIEW_API_KEY` is a SECRET) forwarded to the runner as env.
- There is no step list and no plan file: what a review runs is configured by the
  env alone, so retuning a review is a `charly.yml` change or an org-variable change —
  never a workflow edit.
- The rulebook actually in force is the org variable `AI_REVIEW_PROMPT`; the
  committed `charly.yml` default MUST be a byte-for-byte copy of it (both gates
  forward the variable, so the committed copy is the fallback and the in-tree
  reading). Re-sync that copy whenever the variable changes — a drifted copy is the
  two-rulebooks defect of `opencharly/.github#126`.

## Landing

Every change lands through a pull request gated by the org-required
`charly/pr-validator`. The landing mechanics — the `feat/` branch, the PR-only
rule, `CHANGELOG/` history, and the tag-on-merge CalVer — are owned by
`/charly-internals:git-workflow` and the umbrella `AGENTS.md` /
`charly/AGENTS.md`; this signpost points at them and does not restate them.
