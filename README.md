# seerr + Hide Requested Media

**[seerr](https://github.com/seerr-team/seerr), with the ability to hide already-requested titles from Discover — available today, before it lands upstream.**

```text
ghcr.io/danieldjupvik/seerr:latest
```

Seerr has long had a *Hide Available Media* setting, but nothing to hide titles you've already **requested** — so Discover pages fill up with things you've already asked for. That feature has been requested for years across [Overseerr #3681](https://github.com/sct/overseerr/issues/3681), Jellyseerr, and [seerr #1048](https://github.com/seerr-team/seerr/issues/1048), and an implementation exists in [PR #1855](https://github.com/seerr-team/seerr/pull/1855) — it just hasn't been merged yet.

This repo publishes the **latest official seerr release with that PR applied**, plus two small reviewed fixes to it (documented in [patches/](patches/README.md)). Nothing else is changed. You get a new **Hide Requested Media** checkbox in *Settings → General*, right next to *Hide Available Media*:

- Titles with an **active** request (pending or approved-and-processing) disappear from Discover pages and sliders; declined requests don't hide anything
- They stay visible in **search**, so you can still find them and see their request status
- Global admin setting, off by default — same behavior the upstream PR will ship, with its review findings already fixed

## Quick start

The image is a **drop-in replacement** for `ghcr.io/seerr-team/seerr:latest` — same app, same ports, same config format. Switching in either direction is just changing the image name; your existing config and database are untouched.

### Docker Compose

```yaml
services:
  seerr:
    image: ghcr.io/danieldjupvik/seerr:latest
    container_name: seerr
    environment:
      - TZ=Europe/Oslo
    ports:
      - 5055:5055
    volumes:
      - ./config:/app/config
    restart: unless-stopped
```

### Docker CLI

```bash
docker run -d \
  --name seerr \
  -e TZ=Europe/Oslo \
  -p 5055:5055 \
  -v /path/to/config:/app/config \
  --restart unless-stopped \
  ghcr.io/danieldjupvik/seerr:latest
```

### Unraid

Edit your existing seerr container and change the **Repository** field to `ghcr.io/danieldjupvik/seerr:latest`. Everything else stays the same.

### Enable the feature

After starting: **Settings → General → Hide Requested Media** → check it, save.

## Tags

| Tag | Meaning |
| --- | --- |
| `latest` | Newest upstream release + the patch (mirrors upstream's `latest`) |
| `vX.Y.Z-pr1855` | Pinned build of a specific upstream release, for rollback |

## How it stays up to date

A [scheduled GitHub Actions workflow](.github/workflows/build.yml) runs weekly (and on every merged patch change):

1. Checks that [PR #1855](https://github.com/seerr-team/seerr/pull/1855) is still open upstream.
2. Checks for a new upstream release. If there's nothing new, it exits — no rebuild, no image churn.
3. On a new release: clones upstream at the release tag, applies the **vendored, reviewed patches** (the PR diff pinned to a specific PR commit, plus small reviewed fixes — see [patches/README.md](patches/README.md)), builds with upstream's own unmodified Dockerfile, and pushes. Typically live within a week of an upstream release.

There is no fork being maintained here. Every build starts from pristine upstream source; the only delta is the vendored patches, applied at build time. The patch content is never fetched from the network during a build — it can only change through a reviewed commit in this repo.

## Can I trust this image?

Reasonable question for any third-party image. You don't have to take anyone's word:

- **Everything is built in public.** Every image comes from a GitHub Actions run in this repo — the [full build logs](../../actions) show exactly which upstream tag was cloned and which diff was applied. There are no manual pushes and no secrets involved beyond the repo's own `GITHUB_TOKEN`.
- **The only changes are pinned, vendored diffs** ([patches/](patches/)): the public upstream PR written by [@0xSysR3ll](https://github.com/0xSysR3ll), verbatim, plus a small documented fixup. You can read every line here, and diff the PR copy against the upstream PR yourself. Builds never pull patch content from the network, so an update to the upstream PR branch cannot silently change this image.
- **Don't want to trust it anyway? Run your own.** Fork this repo, enable Actions, and you'll build and publish the identical image into your own GHCR namespace in ~20 minutes.

## Limitations

- **`linux/amd64` only.** No ARM builds (keeps CI fast; open an issue if you'd genuinely use one).
- **Up to a week behind upstream releases** (weekly schedule; run the workflow manually if you want a release sooner).
- **Global setting, not per-user** — same as the upstream PR and the existing Hide Available Media setting.
- **If a new upstream release conflicts with the patch**, no image is published for it: the pipeline fails loudly and the previous version stays available until the vendored diff is updated (reviewed, then committed). You keep a working seerr; you're just temporarily behind.
- **Not affiliated with the seerr team.** If you hit a seerr bug while on this image, please reproduce it on the official image before reporting it upstream — don't send the maintainers chasing ghosts from a patched build.

## What happens when the PR merges upstream?

This repo retires itself. The scheduled workflow detects the merge, opens an issue here with switch-back instructions, and disables itself. At that point: wait for the next official release containing the feature, change your image back to `ghcr.io/seerr-team/seerr:latest`, and keep the checkbox — it'll be the real one. Your config carries over cleanly, including the setting itself.

If you want this to happen sooner, go 👍 [PR #1855](https://github.com/seerr-team/seerr/pull/1855) and [issue #1048](https://github.com/seerr-team/seerr/issues/1048).

## Credits

- The [seerr team](https://github.com/seerr-team) — the actual application (formerly Jellyseerr/Overseerr)
- [@0xSysR3ll](https://github.com/0xSysR3ll) — author of the hide-requested feature in PR #1855

This repo is just plumbing that connects the two a little earlier than the release cycle would.
