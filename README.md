# action-review

The GitHub Actions config for OpenCharly's AI PR-review gate — the replacement for
the retired `opencharly/pi-review-action`, built on charly + the
[plugin-review](https://github.com/opencharly/plugin-review) plugin only.

- **Standard GitHub runner by default** (`runs-on: ${{ vars.REVIEW_RUNNER_LABEL || 'ubuntu-latest' }}`);
  the bootstrap step downloads `charly-linux-amd64` + `charly-plugins-linux-amd64.tar.gz`
  from the pinned `opencharly/charly` release (the same assets the distro package repos
  consume) and resolves the welded `charly review` word — no image, no Go, no project fetch.
- **Self-hosted opt-in** via one variable; the `enabled: false` `review-runner` image
  definition ships in `charly.yml`.
- **Everything configured in `charly.yml`** (env/secret contract, defaults, image);
  secrets only ever exist as GitHub secrets → env vars on the runner.
- **One rulebook, two checks**: the workflow forwards the live org variable
  `AI_REVIEW_PROMPT` and runs the org-pinned engine, so this repo's check and the
  org-wide required gate review with the same rulebook. There is no plan file and no
  prompt file — retuning a review is a variable change, never a workflow edit.

See `docs/action-review.md` for the env/secret table, deploy, and cutover notes.
- smoke: validated by the new gate (2026-09-08)
