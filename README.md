# smvp-runner

**Public** GitHub Actions runner for the **private** [`e7mac/social-media-video-pipeline`](https://github.com/e7mac/social-media-video-pipeline) repo.

## Why this exists

GitHub bills Actions minutes to the repo that *owns the workflow run*, not the
repo whose code runs. **Public repos get unlimited free Actions minutes.** The
daily content pipeline is compute-heavy (video rendering ≈ 1,200 min/month), so
running it here instead of in the private repo takes that line off the bill while
keeping the source code private.

Each workflow checks out the private repo at runtime with a read-only PAT
(`SMVP_REPO_PAT`) and runs its scripts exactly as the private repo's own
workflows did. Only the **cron schedules** were moved here; the private repo
keeps `workflow_dispatch` so the drafts.e7mac.com buttons still work.

## Schedules (all UTC)

| Workflow | Cron | Does |
|---|---|---|
| `daily-pipeline.yml` | `0 13 * * *` | test → render whozart + stave batches + RET recipe → email digest |
| `publish.yml` | `0 14 * * *` | schedule approved items into Postiz (gated by the Settings toggle) |
| `analytics.yml` | `30 14 * * *` | pull channel analytics from Postiz |
| `reconcile.yml` | `0 22 * * *` | flip failed Postiz deliveries to `publish_failed` |

## Security model

This repo is public, so anyone can open a PR. The setup is safe because:

- **No `pull_request` / `pull_request_target` triggers.** Workflows run only on
  `schedule` and `workflow_dispatch`, so a fork PR cannot trigger them and never
  receives secrets. Do **not** add a `pull_request_target` workflow here.
- **Read-only, single-repo PAT.** `SMVP_REPO_PAT` is a fine-grained token scoped
  to *only* the social-media-video-pipeline repo with **Contents: Read-only** —
  enough to check out, nothing more. If it ever leaks, it grants read access to
  one repo and nothing else. Rotate it from the token's GitHub settings page.
- **Read-only `GITHUB_TOKEN`.** Every workflow sets `permissions: contents: read`.
- **Fork-PR workflow approval required for all outside contributors** (repo
  Settings → Actions → "Require approval for all external contributors").

## Secrets to set on THIS repo

Mirror of the private repo's secrets, plus the PAT:

`SMVP_REPO_PAT`, `DATABASE_URL`, `CORPUS_DATABASE_URL`, `MUSIC_AI_API_KEY`,
`POSTIZ_API_KEY`, `AWS_ACCESS_KEY_ID`, `AWS_SECRET_ACCESS_KEY`,
`AWS_DEFAULT_REGION`, `SMTP_SERVER_ADDRESS`, `SMTP_SERVER_PORT`, `SMTP_USERNAME`,
`SMTP_PASSWORD`, `SMTP_FROM_ADDRESS`, `YTDLP_COOKIES_B64`, `YTDLP_USER_AGENT`.

When pipeline secrets rotate, update them in **both** repos (or retire the
private repo's cron secrets if you fully cut over).
