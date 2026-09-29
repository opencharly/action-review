# AGENTS.md — action-review

The GitHub Actions config for OpenCharly's AI PR-review gate — the replacement
for the retired `opencharly/pi-review-action`, built on charly + the
[plugin-review](https://github.com/opencharly/plugin-review) plugin only. The
gate runs the welded `charly review` word on a standard GitHub runner by default
(self-hosted opt-in via `vars.REVIEW_RUNNER_LABEL`), configured entirely from
`charly.yml` + `review-plan.yml`.

Canonical files:

- `charly.yml` — the review **contract**: the `review-contract:` candy (the
  `AI_REVIEW_*` `var:` surface + the full `AI_REVIEW_PROMPT` rulebook, composing
  `plugin-review`) and the `review-runner:` candy (the `enabled: false`
  self-hosted image definition).
- `review-plan.yml` — the step list the `charly review --plan` executor walks;
  runtime plugins join the workflow purely through config.
- `prompt/validator.md` — the PR-validator prompt (its PASS output template
  carries a line `Verdict: PASS`).
- `.github/workflows/ai-review.yml` — the gate itself; `.github/workflows/ci.yml`
  — `charly box validate` + the prompt/plan presence gates + actionlint;
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

- `.github/workflows/ci.yml` is the config gate: it installs charly from the
  pinned release assets, runs `charly box validate`, asserts `review-plan.yml` +
  `prompt/validator.md` exist and that the prompt contains a line `Verdict: PASS`
  (`grep -q '^Verdict: PASS' prompt/validator.md`), and lints the workflows with
  `actionlint`.
- The gate itself is `.github/workflows/ai-review.yml`; `review-plan.yml` decides
  which plugins/steps run.
- The merge gate is the **org-wide** `charly/pr-validator` (required check
  `validate / validate`, defined in `opencharly/.github`).

## Modify this repo

- Every `AI_REVIEW_*` value is declared in `charly.yml` at its currently-used
  production default (API keys are secrets, never committed); the opencharly org
  GitHub settings can override every value, since each name is an org VARIABLE
  (`AI_REVIEW_API_KEY` is a SECRET) forwarded to the runner as env.
- A step-list change belongs in `review-plan.yml`; the `ai-review.yml` workflow
  never changes for that.
- The full review rulebook is `AI_REVIEW_PROMPT` in `charly.yml` — keep the
  prompt actually in force OBVIOUS, not hidden in org settings.

## Landing

Every change lands through a pull request gated by the org-required
`charly/pr-validator`. The landing mechanics — the `feat/` branch, the PR-only
rule, `CHANGELOG/` history, and the tag-on-merge CalVer — are owned by
`/charly-internals:git-workflow` and the umbrella `AGENTS.md` /
`charly/AGENTS.md`; this signpost points at them and does not restate them.
