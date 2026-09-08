# action-review — the org AI PR-review gate (charly config)

This repo declares the validator that replaced `opencharly/pi-review-action`: a
GitHub Actions workflow (standard runner by default, self-hosted opt-in) that runs
the `plugin-review` charly plugin. Everything except secrets is configured here
in `charly.yml`; secrets arrive as GitHub secrets → job env vars.

## Repos

- `opencharly/plugin-review` — the charly plugin (verb:pr + command:review + --plan executor).
- `opencharly/action-review` (this repo) — the workflow + config.

## Environment (`ai-review`)

Vars: AI_REVIEW_PROVIDER, AI_REVIEW_MODEL, AI_REVIEW_BASE_URL, AI_REVIEW_MAX_TURNS,
CHARLY_VERSION (optional pin), REVIEW_RUNNER_LABEL (self-hosted opt-in).
Secrets: AI_REVIEW_API_KEY. Automatic: GITHUB_TOKEN, PR_NUMBER.

## Self-hosted variant (optional)

```bash
TOKEN=$(gh api -X POST /orgs/opencharly/actions/runners/registration-token --jq .token)
charly config review-runner -e RUNNER_ORG=opencharly -e RUNNER_TOKEN="$TOKEN"
charly start review-runner
```
then set `REVIEW_RUNNER_LABEL=review-runner` in the repo/org environment. The
default path (ubuntu-latest) needs none of this — the workflow downloads the charly
release binary + welded-plugins tarball and runs the same steps.
