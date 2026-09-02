# renovate-test-repo

Minimal sandbox to test the **official Renovate GitHub App** (Mend-hosted),
as a comparison against the self-hosted GitHub Actions setup used in
[flow-ui-react](https://github.com/UNIwise/flow-ui-react/pull/776).

## Why this repo exists

To validate, without needing org-admin/DevOps involvement, how the GitHub
App behaves differently from a self-hosted cron-based workflow — in
particular around detection latency (event-driven vs. scheduled) and PR
behavior (e.g. whether App-created PRs trigger downstream CI automatically,
unlike `GITHUB_TOKEN`-authored ones).

`package.json` intentionally pins `axios@0.21.1`, a version with known
published vulnerabilities (e.g. GHSA-4w2v-q235-vp99, GHSA-cph5-m8f7-6c5x),
so the app has something real to act on as soon as it's installed.

## Setup steps

1. Install the [Renovate GitHub App](https://github.com/apps/renovate) on
   this repository (Settings → Applications → your account → Renovate →
   configure repository access).
2. The app will open an initial "Configure Renovate" onboarding PR — merge
   or close it; `renovate.json` in this repo already provides the
   vulnerability-only configuration, so onboarding is mostly a formality.
3. Within a short time after install, the app should detect the vulnerable
   `axios` version and open a fix PR on its own, without any manual
   workflow dispatch or waiting for a cron.

## `renovate.json`

Same vulnerability-only approach as flow-ui-react: `"enabled": false`
globally (no routine dependency-update noise), with `vulnerabilityAlerts`
and `osvVulnerabilityAlerts` enabled (these bypass the global disable by
design). No automerge — PRs still require manual review.
