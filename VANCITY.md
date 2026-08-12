# vancity-in/odysseus

A build fork of [odysseus-dev/odysseus](https://github.com/odysseus-dev/odysseus).
We publish an image; we do not patch the application.

## Why this fork exists

Upstream ships `.github/workflows/docker-publish.yml`, which pushes to
`ghcr.io/<owner>/<repo>` — but **their package is private**. Anonymous pulls get
`401` from the GHCR token endpoint and `403` on the manifest, so Railway (or
anything else) cannot pull it. There is no official public Odysseus image.

The alternative on the Railway template marketplace is worse. The `odysseus-1`
template is unverified, has single-digit deploys, and runs
`xiaosong233/odysseus-railway:latest` — an unknown personal Docker Hub account
behind a mutable tag, for a service that holds email, calendar, and documents.

So we rebuild upstream's own `Dockerfile`, unmodified, under our org.

## Branch contract

| Branch    | Contents | Rule |
| --------- | -------- | ---- |
| `main`    | A pure mirror of upstream `main`. | Fast-forwarded by CI. **Never hand-edit.** |
| `vancity` | Only this file and `.github/workflows/vancity-build.yml`. Default branch. | Where our changes go. |

`vancity` is an orphan branch on purpose. Keeping our workflow off `main` means
the mirror never diverges, so the sync stays a fast-forward forever and the
published image always corresponds to an exact upstream commit. Scheduled
workflows only run from the default branch, which is why `vancity` is default
rather than `main`.

We track upstream `main`, not `dev`. Upstream fast-forwards `main` at each
curated release; `dev` is their firehose.

## Tags

| Tag | Meaning |
| --- | ------- |
| `ghcr.io/vancity-in/odysseus:main-<sha7>` | An exact upstream commit. **Pin Railway to this.** |
| `ghcr.io/vancity-in/odysseus:stable` | Moving pointer at the newest successful build. Convenience only. |

`linux/amd64` only — Railway runs amd64, and upstream's build compiles
realesrgan wheels from source, so an arm64 leg under QEMU roughly triples
wall-clock for a platform nothing consumes.

## Licensing

Odysseus is AGPL-3.0. This fork is public, and the image we publish is built
from the commit this repo mirrors — which is what satisfies the corresponding-
source obligation for a network service. Keep this repo public.

## Bumping

CI syncs daily at 04:20 UTC and builds only when upstream `main` has moved.
To force one: **Actions → Sync upstream stable and publish to GHCR → Run
workflow** with `force` checked. Then repoint the Railway service at the new
`main-<sha7>` tag.
