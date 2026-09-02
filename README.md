# renovate-test-repo

Minimal sandbox to test the **official Renovate GitHub App** (Mend-hosted),
as a comparison against the self-hosted GitHub Actions setup.

## Why this repo exists

To validate, without needing org-admin/DevOps involvement, how the GitHub
App behaves differently from a self-hosted cron-based workflow — in
particular around detection latency (event-driven vs. scheduled) and PR
behavior (e.g. whether App-created PRs trigger downstream CI automatically,
unlike `GITHUB_TOKEN`-authored ones).

`package.json` intentionally pins two known-vulnerable states so there's
something real to act on as soon as an automation tool is installed:

- **`axios@0.21.1`** — a *direct* dependency with known published
  vulnerabilities (e.g. GHSA-4w2v-q235-vp99, GHSA-cph5-m8f7-6c5x).
- **`execa@5.1.1`** → transitively depends on **`cross-spawn@7.0.3`** — a
  *nested/internal sub-dependency* vulnerability
  ([GHSA-3xgq-45jj-v275](https://github.com/advisories/GHSA-3xgq-45jj-v275),
  ReDoS, fixed in `7.0.5`). `cross-spawn` is **not** listed in `package.json`
  at all — it only appears in `pnpm-lock.yaml`, nested under `execa`. This
  mirrors the common real-world case where `pnpm update` alone (no manifest
  edit, lockfile-only change) is enough to fix the vulnerability, since
  `execa`'s own manifest already allows `cross-spawn: ^7.0.3`, a range wide
  enough to include the patched `7.0.6`.

Verify locally with `pnpm audit` (both vulnerabilities show up) and
`pnpm update cross-spawn` (fixes the second one with **zero** `package.json`
changes — confirmed by diffing `package.json` before/after).

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
