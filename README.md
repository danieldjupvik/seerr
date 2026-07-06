# seerr (patched)

Automated build of [seerr](https://github.com/seerr-team/seerr) with [PR #1855](https://github.com/seerr-team/seerr/pull/1855) ("hide already requested media") applied on top of the latest official release. Adds a **Hide Requested Media** checkbox to Settings → General, next to the existing Hide Available Media setting.

This is a temporary bridge until PR #1855 merges upstream — see [issue #1048](https://github.com/seerr-team/seerr/issues/1048). The same pipeline can carry additional patches in the future: each one is just another diff applied in [build.yml](.github/workflows/build.yml) before the image is built.

## Image

```
ghcr.io/danieldjupvik/seerr:latest
```

Drop-in replacement for `ghcr.io/seerr-team/seerr:latest` — same app, same config format, same volumes and ports (`linux/amd64` only). In the Unraid container settings, change the **Repository** field to the image above; leave everything else untouched. Switching back later is the same edit in reverse.

Each build is also tagged `<upstream version>-pr1855` (e.g. `v3.3.0-pr1855`) if you ever need to pin or roll back.

## How it works

A daily scheduled workflow ([build.yml](.github/workflows/build.yml)):

1. Checks whether PR #1855 is still open upstream. If it was **merged or closed**, the workflow opens an issue in this repo telling you to switch back to the official image, disables itself, and fails — that issue is your signal to retire the patch.
2. Fetches the latest upstream release tag and compares it to [.last-built](.last-built). If already built, it exits in seconds (with an occasional empty keepalive commit so GitHub doesn't auto-disable the schedule after 60 days of inactivity).
3. On a new release: clones upstream at that tag, applies the PR diff fresh from the GitHub API, builds with upstream's own Dockerfile, and pushes to GHCR. If the diff no longer applies, it opens an issue and fails instead of shipping a broken image.

There is no fork to keep in sync — every build starts from pristine upstream source plus the PR diff.

## One-time setup after the first successful build

The first push creates the GHCR package as **private**. Make it public so Unraid can pull without credentials and it doesn't count against private package storage:

GitHub profile → Packages → `seerr` → Package settings → Change visibility → Public.

## Manual build

Actions → build → Run workflow (tick **force** to rebuild the current release, e.g. to pick up a rebased version of the PR diff).

## Failure modes

| Event | What happens |
| --- | --- |
| PR merged upstream | Issue opened with switch-back instructions, workflow disables itself. Your running image keeps working. |
| PR closed without merge | Issue opened, workflow disables itself. |
| PR diff conflicts with a new release | Issue opened, build fails, previous image stays available. You're behind upstream until the PR is rebased or you retire the patch. |
| No new release | ~5-second no-op run. |
