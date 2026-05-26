# Reproducer for dependabot-core issue #15112

## Problem

When `.npmrc` sets `min-release-age`, npm refuses to install package versions
released more recently than the configured age. Dependabot ignores its own
`cooldown` setting for security updates, but `min-release-age` is enforced by
npm itself at runtime, so security update PRs fail with an `ETARGET` error.

This repository uses `min-release-age=36500` (100 years) so the failure is
always reproducible regardless of when you run it.

`lodash@4.17.15` has a known prototype-pollution vulnerability
([CVE-2020-8203](https://github.com/advisories/GHSA-p6mc-m468-83gw)).
The fix is `lodash@4.17.21`, but with `min-release-age` set npm will reject
that version with:

```
npm error code ETARGET
npm error notarget No matching version found for lodash@4.17.21 with a date before ...
```

## Reproducing locally

```sh
# Fails - mimics what Dependabot runs for the version check
npm install lodash@4.17.21 --package-lock-only --dry-run=true --ignore-scripts

# Succeeds - the fix applied in dependabot-core PR for issue #15112
npm install lodash@4.17.21 --package-lock-only --dry-run=true --ignore-scripts --min-release-age=0
```

## Fix

Pass `--min-release-age=0` to npm when the update is a security update, so the
`.npmrc` setting is overridden only for that invocation.

See: https://github.com/dependabot/dependabot-core/issues/15112
