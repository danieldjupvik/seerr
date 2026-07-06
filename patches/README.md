# Vendored patches

Applied by the build workflow on top of the upstream release source, in lexicographic filename order.

| File | Source | Pinned at |
| --- | --- | --- |
| `pr1855.diff` | [seerr-team/seerr#1855](https://github.com/seerr-team/seerr/pull/1855), verbatim | head `cfeeb8ce8416f9775e503cfa8a03845d8f9f56f4` (2026-06-18) |
| `pr1855.fixups.diff` | This repo (review findings on the PR code) | — |

## What the fixups change

Two issues found in review of the PR code, fixed server-side in `Media.getRelatedMedia`:

1. **Only active requests hide a title.** The PR hides any title with request rows, but declined request rows persist forever — a declined title would never reappear in Discover. The join now excludes `DECLINED` and `COMPLETED` requests, matching how the rest of seerr defines active requests.
2. **Minimal payload instead of full request rows.** The PR's `leftJoinAndSelect` serialized entire `MediaRequest` rows into every discover/search/related-media response. The join now selects only `id` and `status` — all the frontend needs for its length check.

Both fixes live in one server-side join, so the PR's frontend code works unmodified.

## Updating a patch

To pick up a rebased/updated upstream PR: re-download the diff, **review it**, replace `pr1855.diff`, re-verify the fixups still apply on top, and commit — the content shipped into the image only ever changes through a reviewed commit in this repo.

## Whitespace

Vendored PR diffs are kept byte-identical to their upstream source so they can be audited against the PR, including any whitespace quirks in the code they add. `.gitattributes` exempts `patches/*.diff` from git whitespace checks instead of altering the files.
