# 0001. Generator runtime and container base image

- **Status:** proposed
- **Date:** 2026-09-24
- **Component:** C04

## Context

The brief fixes Python generators managed with `uv`, with pinned tool versions in a container. Its quality bar says CI regenerates the corpus in a clean container and fails if any SHA-256 changes, so every tool that writes bytes must give the same output on every run.

What forces the choice:

- **Encoders are needed.** Fixtures need JPEG, PNG, WebP, HEIC, AVIF, TIFF, H.264, HEVC, VP9, AAC, Opus, Vorbis, FLAC and MP3 encoders, plus qpdf for PDF encryption. The native stack (C03) builds decoders only, with a minimal LGPL ffmpeg, so its base image cannot produce most fixtures.
- **Independence from the engine.** If the corpus were generated with the same decoder builds the engine tests with, a bug shared by both could never show up. Generating with a separate toolchain keeps the check able to fail.
- **Determinism depends on the platform.** SIMD paths in encoders can give different bytes on different CPU architectures. GitHub-hosted `linux/amd64` runners are free and unlimited for public repositories.
- **Pinning must survive time.** Mirrors of most distributions delete old package versions. Debian keeps every published package at `snapshot.debian.org`. We checked that it serves trixie today: the snapshot at 2026-09-01 returns the trixie `Release` file, version 13.6.
- Versions checked on 2026-09-24: Python 3.14.7 is the current 3.14 release (endoflife.date). `uv` 0.12.18 is the latest release on GitHub. `debian:trixie-slim` resolves to `sha256:a99cfc517144bc59b1978475ec53b46ecabec7e43635402ee5b77cc54cd1b20a` (Docker Hub, updated 2026-09-19).

## Decision

1. **Runtime:** Python 3.14, `requires-python = "==3.14.*"`, dependencies locked with hashes in a committed `uv.lock`. The container copies the `uv` binary from `ghcr.io/astral-sh/uv`, pinned by digest.
2. **Base image:** `debian:trixie-slim`, pinned by digest. Apt reads only from `snapshot.debian.org` at a fixed timestamp, and every package is installed at an exact version recorded in `container/apt-packages.txt`. Tools come from Debian packages: ffmpeg, libvips, libheif with its encoder plugins, libavif, qpdf, and Pillow built against them.
3. **Published generator image:** CI in this repository builds the image and publishes it to `ghcr.io/vetload/corpus-generator`. Each corpus version records the image digest in `manifest.json`. The image does not have to rebuild byte for byte. The corpus files do.
4. **Authoritative platform:** `linux/amd64` on GitHub-hosted Ubuntu runners. Developers on arm64 run the same image under emulation. Output from any other platform is never committed.
5. **Deterministic execution:** generation runs with `--network none`, `SOURCE_DATE_EPOCH` fixed, `TZ=UTC`, `LC_ALL=C.UTF-8` and `PYTHONHASHSEED=0`. Encoders run single-threaded. All randomness comes from a PRNG seeded from the file's path. ZIP entries get fixed timestamps and a fixed order. ffmpeg runs with `-fflags +bitexact`, `-flags:v +bitexact`, `-flags:a +bitexact`, `-map_metadata -1` and `-threads 1`. qpdf runs with `--deterministic-id` for plain files and `--static-id --static-aes-iv` for encrypted ones. The qpdf manual describes those two options as unsafe for real use and meant only for reproducible test files, which is exactly our purpose.
6. **Frozen escape hatch:** when a tool cannot be made deterministic, the file is produced once by its recorded generator and committed as bytes, with `origin.type: frozen` and a written reason. CI still checks its hash. The aim is under 5 percent of files, and every frozen file is listed in the release notes.

## Options considered

| Option | For | Against |
| --- | --- | --- |
| Debian slim by digest, snapshot apt pins, `uv.lock` (chosen) | Every version stays fetchable. Familiar to any agent. Encoders come ready-built | `snapshot.debian.org` can be slow or down, which would block image rebuilds |
| Nix flake | Strongest reproducibility, including the image itself | Steep learning curve for every future agent. A large store. Does not by itself fix encoder threading or timestamps |
| Alpine by digest | Small | Old package versions disappear from mirrors, so pins break; musl differences |
| `uv` on the host, no container | Simplest | Native libraries differ between machines, so hashes drift |
| Reuse the C03 native base image | One toolchain | Decoders only, no encoders. Circular: the corpus would test the engine against itself. Couples a public repository to the platform's private lock |

## Consequences

- Easier: anyone can rebuild the corpus with one `docker run`, and a hash change points to one generator.
- Harder: upgrading any tool changes bytes, so tool upgrades become deliberate corpus releases (see [ADR-0002](0002-expected-result-versioning.md)).
- The image contains GPL encoders such as x264 and x265. That is fine for a build tool. The fixtures it makes are not derived from the encoder's code. Publishing the image publicly means offering the matching source, which the pinned Debian snapshot provides. The README says so.
- **Cost:** $0 at idle and at scale. Actions on public repositories and public GHCR images are free. A full regeneration is estimated at under 15 minutes, which is not measured yet.

## Revisit when

- `snapshot.debian.org` fails a rebuild twice in a quarter. Mirror the needed `.deb` files as a release asset instead.
- A GitHub runner image update changes output from an unchanged container. That would mean a kernel or CPU dependency, so pin the CPU features or move the affected files to frozen.
- A consumer needs arm64-authoritative files.
