# Vendored patches

Applied by the build workflow on top of the upstream release source, in lexicographic filename order.

| File | Source | Pinned at |
| --- | --- | --- |
| `pr1855.diff` | [seerr-team/seerr#1855](https://github.com/seerr-team/seerr/pull/1855), verbatim | head `cfeeb8ce8416f9775e503cfa8a03845d8f9f56f4` (2026-06-18) |
| `pr1855.fixups.diff` | This repo (review findings on the PR code) | — |

## What the fixups change

Three issues found reviewing and running the PR code:

1. **Hide by media status, not request rows.** The PR hides only titles with seerr request rows — but media synced from Radarr/Sonarr (added there directly or picked up by the scheduled scan) shows the same "Requested"/"Processing" badge while having no request rows, so it stayed visible. The discover filters now hide on media status `PENDING`/`PROCESSING`, exactly matching the badge, and covering both seerr requests and arr-synced items. A declined request resets media status, so declined titles correctly reappear.
2. **Minimal payload instead of full request rows.** The PR's `leftJoinAndSelect` serialized entire `MediaRequest` rows into every discover/search/related-media response. The join now selects only `id` and `status`.
3. **Only active requests in the payload.** The join excludes `DECLINED` and `COMPLETED` request rows, matching how the rest of seerr defines active requests.

## Updating a patch

To pick up a rebased/updated upstream PR: re-download the diff, **review it**, replace `pr1855.diff`, re-verify the fixups still apply on top, and commit — the content shipped into the image only ever changes through a reviewed commit in this repo.

## Whitespace

Vendored PR diffs are kept byte-identical to their upstream source so they can be audited against the PR, including any whitespace quirks in the code they add. `.gitattributes` exempts `patches/*.diff` from git whitespace checks instead of altering the files.

## License

The diffs in this directory are portions of [seerr](https://github.com/seerr-team/seerr), licensed under the [MIT License](https://github.com/seerr-team/seerr/blob/develop/LICENSE), Copyright (c) 2020 sct and seerr contributors. This repo's own [LICENSE](../LICENSE) covers only the build pipeline, not the vendored patch content.
