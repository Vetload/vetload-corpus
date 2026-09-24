# Implementation plan: Vetload corpus (C04)

Brief: `../vetload-platform/docs/components/C04-corpus.md` (private). Design: [ARCHITECTURE.md](ARCHITECTURE.md). Decisions: [ADR-0001](docs/architecture/decisions/0001-generator-runtime-and-container.md), [ADR-0002](docs/architecture/decisions/0002-expected-result-versioning.md), [ADR-0003](docs/architecture/decisions/0003-release-format-and-consumption.md). First files: [seed corpus](docs/seed-corpus.md).

This plan gives order and scope, not dates.

## Milestones

| Milestone | Phase | Done when |
| --- | --- | --- |
| M0 Planning | Wave 1 | This pull request is merged, the contract-needs issue is answered by C01, and C06 and C05 have reviewed the seed list |
| M1 Pipeline skeleton | Wave 1 | The generator image is published by digest. About 10 files build in CI with a determinism check that has been seen failing |
| M2 Seed release `v0.1.0` | Wave 1, **gate G1** | 166 files (the target was about 150) are released with checksums and attestations, and the G1 proof workflow is green |
| M3 In engine CI | P0 | C06's golden tests run on the pinned release. Corrections ship as `0.2.x`. Legacy Office files are added |
| M4 Public corpus and benchmark | P1 | About 400 files. Section 11 fixtures. EICAR asset. Benchmark JSON for C19. `v1.0.0` |
| M5 Additions and provenance | P2 | C2PA, HDR, VFR variants, animated AVIF, Live Photos, leak-surface fixtures |
| M6 New kinds | P3 | SVG, camera RAW, PSD, DICOM with synthetic PHI, fonts, 3D models, archives with declared-only bombs |

## Tasks

### M1 Pipeline skeleton

1. `pyproject.toml` for Python 3.14 with `uv.lock` (hashes on). Pillow, PyYAML and jsonschema are the only runtime dependencies to start with.
2. `container/Dockerfile`: `debian:trixie-slim` by digest, apt from `snapshot.debian.org` at a fixed timestamp, exact versions in `container/apt-packages.txt`, and `uv` copied by digest. An `image.yml` workflow publishes `ghcr.io/vetload/corpus-generator`.
3. **Determinism spike (labelled spike):** encode one file each with libheif, libavif, x264, x265, libvpx and qpdf AES-256, ten times across two runners. Record which tools reproduce. Anything that does not becomes frozen or gets a different tool. This runs first because it can change the tool list.
4. The spec format (`specs/*.yaml`) and loader. The generator registry with an `rng` seeded from the path.
5. `tools/build.py`, which runs in the container with `--network none`, and `tools/verify.py`: schema, hashes, unique paths, licence allowlist, coverage report.
6. `ci.yml` on pull requests: build image or pull by digest, generate, verify, run gitleaks. Path filters are not needed because the repository is public.
7. The first 10 files: baseline JPEG and PNG, truncations of each, zero bytes, a mismatch, and one of each orientation family.

### M2 Seed release `v0.1.0` (gate G1)

1. Generators for every group in the [seed list](docs/seed-corpus.md): byte-level structure, images, audio and video through ffmpeg, PDF by hand plus qpdf, and OOXML through `zipfile`.
2. A small compound-file (CFB) writer, about 300 lines, written from the published MS-CFB specification. It is used for VBA projects (MS-OVBA compression) and for encrypted OOXML packages (MS-OFFCRYPTO agile encryption with a seeded salt).
3. The bomb guard: decompress every zlib, deflate and ZIP stream in every file up to a 64 MiB cap and fail past it.
4. `tools/release.py`, which builds the reproducible tarball, `SHA256SUMS` and `expectations-diff.json`. `release.yml` runs on `v*` tags and attests the assets.
5. `tools/fetch.py`, the reference consumer script (ADR-0003).
6. Review the † rows with C06, C11, C12 and C13. Record each agreement or change in the spec.
7. Pin the C01 v0 registry values and set `contracts_version`.
8. The G1 proof workflow (`g1-proof.yml`): download the released assets on a fresh runner, check them, and run the tamper control.

### M3 P0

- Support C06 as it wires the pinned release into engine CI. Fix wrong expectations through ADR-0002.
- Add legacy `.doc`, `.xls` and `.ppt` using the CFB writer.
- Add fixtures the kind components ask for (C11 to C13), each with an issue.

### M4 P1

- Grow to about 400 files: section 11.1, 11.4 and 11.8 fixtures, more polyglots, more encodings.
- Ship EICAR files in `vetload-corpus-av-X.Y.Z.tar.gz` only.
- Build the benchmark harness in the generator image or a sibling image. It runs `file`/libmagic, python-magic, `file-type` (Node 24), Apache Tika, ffprobe and ImageMagick against a release, and writes per-detector accuracy on MIME and kind to `benchmark-X.Y.Z.json`. The output schema is agreed with C19.
- Write a public README with usage for CI and a licence table. Release `v1.0.0`.

### M5 P2 and M6 P3

- M5: C2PA manifests signed with a self-generated test certificate (the private key is generated at build time and never committed), gain-map HDR, HDR10 and HLG video, VFR variants, Live Photo pairs, unapplied redactions, hidden layers.
- M6: new-kind fixtures. Third-party files come only from allowlisted, per-file-cleared licences, such as CC0 RAW and DNG conformance files.

## Test plan

Every check is shown failing once, on purpose, in the pull request that adds it. The pull request records the output.

| Check | How it is shown failing | Positive control for "nothing found" |
| --- | --- | --- |
| Determinism: regenerated hashes equal `manifest.json` | Change one generator parameter; CI goes red on exactly that path | The verifier reports the count of files compared, which must equal the entry count |
| Schema validation | A spec with a reason on `success`, an unknown flag, or a missing licence fails (already demonstrated on the draft schema in this pull request) | Validation reports how many entries it checked |
| Unique paths and claimed-name consistency | Duplicate a path | none needed |
| Licence allowlist | Add an entry with `CC-BY-NC-4.0` | The number of licences seen is printed |
| Bomb guard | A test file that inflates to 65 MiB must fail | Reports the largest expansion found, which must be above 0 |
| No network during generation | A test generator that opens a socket must fail | none needed |
| Coverage: every launch kind and six outcomes present | Remove all `encrypted` entries from a copy of the specs | none needed |
| Reproducible tarball | Build twice with a different `SOURCE_DATE_EPOCH`; the hashes must differ, then match with the same value | none needed |
| `tools/fetch.py` | Flip one byte in an extracted file; verification fails. Add an unlisted file; it fails | Reports the files verified |
| Tar-slip rejection | An archive with a `../x` entry is rejected | none needed |
| gitleaks | Run it on a temporary directory outside git that holds a fake AWS-shaped key; it must be flagged | none needed |

## Risks

| Risk | Likelihood | Mitigation |
| --- | --- | --- |
| HEIC, AVIF or HEVC encoders are not deterministic even single-threaded | Medium | M1 spike first; frozen files as a fallback; kvazaar or other encoders |
| qpdf AES-256 salts are random even with `--static-id` | Medium (not verified) | Our own seeded writer for the PDF standard security handler (AES-256, revision 6), or a frozen file |
| Disagreement over † expectations delays G1 | Medium | Release `v0.1.0` with agreed rows only if needed. Disputed rows follow in `0.2.0`, listed in the release notes |
| C01 renames reasons or flags after `v0.1.0` | Medium | A minor release, and `expectations-diff.json` shows every rename |
| Consumers' security tools quarantine polyglots or macro files | Low to medium | Documented. EICAR kept apart. Macros benign with no auto-run |
| `snapshot.debian.org` throttling | Low | Reuse the published image. Mirror the `.deb` files as a release asset if needed |
| The public repository reveals plans | Low | Only kind and phase labels, no dates, pricing or strategy. Reviewed in every pull request |

## Open questions

1. Is `empty` a reason under `unsupported_type`, or does the zero-byte file get its own outcome? (C01)
2. Should the corpus record C05's range-read budget per file (`reads_used`), or does C05 keep that on its side? (C05)
3. Does a file exactly at `max_image_pixels` pass or fail? Row 76 of the seed list assumes equal to the limit passes. (C01, C06)
4. Which exact `facts` names does C06 want, and how do they map to result-document paths? (C06)
5. Does C10's test mode (P1) identify corpus files by `sha256` from this manifest? If so, the manifest is its lookup table, and paths stay immutable. (C10, wave 2)

## Founder actions

| Action | Blocks |
| --- | --- |
| Allow `Vetload/vetload-corpus` to publish public packages to GHCR under the `vetload` organisation | M1 image publishing |
| Turn on immutable releases for this repository (the API shows `enabled: false`) | M2 hardening, not G1 itself |
| Turn on secret scanning, push protection and CodeQL default setup (all currently off, and free for public repositories) | Security baseline |
| Merge planning, then tag `v0.1.0` on `main` or approve the agent to push the tag | G1 |
