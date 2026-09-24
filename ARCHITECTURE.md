# Architecture: Vetload corpus (C04)

A versioned, reproducible set of awkward files, each paired with the stable truths a truthful inspector must report. It is the engine's regression suite, customers' CI fixture set and a benchmark's ground truth. Data plus generators; nothing runs in production.

Decisions: [ADR-0049 Wave 1 alignment](https://github.com/Vetload/vetload-platform/blob/main/docs/architecture/decisions/0049-wave-1-alignment.md) (program, overrides), [ADR-0001 generator runtime](docs/architecture/decisions/0001-generator-runtime-and-container.md), [ADR-0002 expectation versioning](docs/architecture/decisions/0002-expected-result-versioning.md), [ADR-0003 release format](docs/architecture/decisions/0003-release-format-and-consumption.md). First files: [seed corpus](docs/seed-corpus.md). Schema draft: [`schema/manifest.schema.json`](schema/manifest.schema.json).

## Modules

| Path | Role |
| --- | --- |
| `schema/vendor/` | C01's `vetload-corpus-schema` files and `contracts.lock`, vendored by checksum |
| `specs/<category>.yaml` | Hand-written entries: path, generator, parameters, licence, `expected` |
| `generators/<area>/` | Python functions `(params, rng) -> bytes`, seeded per path |
| `thirdparty/<name>/` | Third-party samples with upstream licence and URL |
| `frozen/` | Bytes that cannot be regenerated deterministically, each with a reason (ADR-0001) |
| `tools/build.py` | Runs generators in the container; writes `dist/` |
| `tools/verify.py` | Schema, hashes against committed `manifest.json`, coverage, licence and bomb guards |
| `tools/release.py` | Builds the reproducible tarball, `SHA256SUMS` and `expectations-diff.json` |
| `tools/fetch.py` | Reference download-and-verify script for consumers |
| `container/` | Dockerfile and apt pins for the generator image |
| `benchmark/` (P1) | Runs open-source detectors; writes `benchmark-X.Y.Z.json` |
| `manifest.json` | Generated and committed. Its hashes are the determinism baseline |

## Data model and access patterns

A header (`manifest_version`, `corpus_version`, `contracts_version`, `contracts_schema`, `generated_by`, `test_limits`) and one entry per file: `path` (stable ID; basename is the claimed name), `sha256`, `bytes`, `claimed`, `origin` (generated, frozen, third party), `license` (SPDX), `categories`, `introduced_in`, `description`, `expected`, `facts`, `revision`. `expected` is C01's expected-result schema, `$ref`ed and never redefined: outcome, reason, kind (`null` when nothing matched), MIME, format, mismatch, dimensions `{stored, display}`, risk flags (ADR-0049 A1 to A4). Consumers gate outcomes by their own capabilities (ADR-0002).

Consumers read the whole manifest and key on `path`, never on a header hash, because truncated and renamed variants share leading bytes.

## Build and verification flow

1. A pull request changes a spec or generator. CI runs `tools/build.py` in the pinned image with `--network none`, then `tools/verify.py`.
2. `verify` fails if any file's SHA-256 differs from the committed `manifest.json`. An intended change updates `manifest.json` in the same pull request, so the diff shows exactly which files changed.
3. A `vX.Y.Z` tag triggers `release.yml`: regenerate, verify, build the archive twice and compare the two, attest, then upload the assets named in ADR-0003.

## Failure modes

| Failure | Effect | Handling |
| --- | --- | --- |
| A tool gives different bytes | CI turns red | Fix flags, or freeze the file with a reason |
| A wrong expectation is released | Consumers fail or wrongly pass | Fix it, bump `revision`, publish in `expectations-diff.json` |
| A live bomb slips in | Hurts consumers | Bomb guard decompresses every stream with a cap |
| Licence missing or incompatible | Legal exposure | Schema requires SPDX; `verify` allowlists licences |
| A release asset is replaced | Tampering | Consumers pin the archive hash; attestations; immutable releases |

## Security

- Public repository, no secrets. Only the workflow `GITHUB_TOKEN`, with write scopes (`contents`, `packages`, `id-token`, `attestations`) only in the release and image workflows.
- No real malware: the EICAR string only, in a separate asset from P1. No real personal data: GPS is 0,0, names are synthetic, and DICOM PHI in P3 is invented. No live bombs: sizes are declared, never expanded, enforced by the bomb guard. Macros are benign with no auto-run; external references use `example.invalid`.
- Generation runs without network; third-party inputs are pinned by upstream SHA-256; `tools/fetch.py` rejects tar-slip entries.
- `gitleaks` runs in CI. Secret scanning, push protection and CodeQL are free here but currently off (checked 2026-09-24); enabling them is a founder action.

## Cost

| Item | Idle | At P1 scale (about 400 files, weekly releases) |
| --- | --- | --- |
| GitHub Actions (public repository) | $0 | $0: standard runners are free for public repositories |
| GitHub Releases storage and downloads | $0 | $0: no bandwidth charge, under 2 GiB per asset |
| GHCR public image | $0 | $0 |
| AWS | none | none |

Consumers pay only their own CI minutes; the platform caches on the pinned hash.

## How it scales

Files grow about tenfold by P3, far below release limits. Generation parallelises by category because seeds are per path. An archive nearing 1 GiB splits by category (ADR-0003).

## Brief conflict

The brief asks for files that cover "every outcome". `processing_timeout` and `processing_crash` are not properties of a file. A correct engine never crashes, and a timeout depends on hardware and deadlines. We propose that C06's fault injection owns those two outcomes, and that the corpus covers the other six.

## Interfaces provided

| Name | Kind | Exact identifier | Consumers | Phase |
| --- | --- | --- | --- | --- |
| Corpus archive | release artefact | `vetload-corpus-X.Y.Z.tar.gz` on `Vetload/vetload-corpus` Releases, tag `vX.Y.Z` | C05, C06, C11 to C15, C18, C24, C25 | Wave 1 (`v0.1.0`) |
| Manifest | file format | `manifest.json`, schema `urn:vetload:corpus:manifest:0.1.0-draft` | same, C10 test mode | Wave 1 |
| Manifest schema | file format | `manifest.schema.json` (JSON Schema 2020-12) | same | Wave 1 |
| Checksums | release artefact | `SHA256SUMS` in GNU coreutils format | same | Wave 1 |
| Expectation diff | release artefact | `expectations-diff.json` | C06, C05 | P0 (from `0.2.0`) |
| Provenance | attestation | GitHub build provenance for each asset | any consumer | Wave 1 |
| Generator image | image | `ghcr.io/vetload/corpus-generator@sha256:…` | reproducers | Wave 1 |
| Fetch script | file format | `tools/fetch.py` (CC0) | consumers | P0 |
| Antivirus fixtures | release artefact | `vetload-corpus-av-X.Y.Z.tar.gz` | C15, C10 | P1 |
| Benchmark results | release artefact | `benchmark-X.Y.Z.json` | C19 | P1 |

## Interfaces consumed

| Name | Kind | Exact identifier | Owner | Phase |
| --- | --- | --- | --- | --- |
| Corpus schema | release artefact, vendored | `vetload-corpus-schema-<v>.tar.gz` from `contracts/v<v>`: `expected-result.schema.json`, registries JSON, MIME list; pinned in `schema/vendor/contracts.lock` | C01 | Wave 1 |
| Capabilities export | file format | C06's format support matrix, used by consumers for gating, not by this repository | C06 | P0 |

The corpus does not use C03's native image, on purpose (ADR-0001).

## Data owned

- GitHub Releases and tags `v*` of `Vetload/vetload-corpus`. Assets are named as in ADR-0003 and never replaced.
- The GHCR package `ghcr.io/vetload/corpus-generator`, referenced by digest; the founder makes it public after its first publish (ADR-0049 F5).
- Repository paths: `specs/`, `generators/`, `thirdparty/`, `frozen/`, `schema/`, `tools/`, `container/`, `benchmark/`, `manifest.json`.
- No AWS resources, tables, buckets or parameters.

## Contract needs

Settled by [ADR-0049](https://github.com/Vetload/vetload-platform/blob/main/docs/architecture/decisions/0049-wave-1-alignment.md) A1 to A4 after Vetload/vetload-platform#39: C01's reason and kind values (`recognised_unsupported`, `empty_file`, `size_limit`, `dimension_limit`, `kind: null`), the flag code `office_external_references`, and one expected-result schema in the `vetload-corpus-schema` artefact. Still open in #39:

1. C01 publishes that artefact at `contracts/v0.1.0`. The corpus states both `stored` and `display` in every `dimensions`.
2. Which limit reason wins when a file exceeds both the pixel and the side limit (the seed avoids such files).
3. C05's `formats.json` supplies `format` IDs; encrypted OOXML uses `application/x-ole-storage` provisionally.

## Assumptions about other Wave 1 components

- **C01** tags `contracts/v0.1.0` with the corpus schema before the corpus tags `v0.1.0` (ADR-0049 E4).
- **C06** pins the corpus by version and archive hash, owns golden documents and known deviations, maps the corpus's fact names to result-document paths, applies capability gating, and reviews the † rows.
- **C05** consumes the same release and compares kind, MIME, format and stored dimensions.
- **C02, C03, C07, C08, C09** have no interface with the corpus in Wave 1.

## Gate G1 deliverable

**Release `v0.1.0` of `Vetload/vetload-corpus`**, tagged after `contracts/v0.1.0`, with 167 files and the assets `vetload-corpus-0.1.0.tar.gz`, `manifest.json`, `manifest.schema.json` and `SHA256SUMS`, all attested.

**Proof it is done:** a clean workflow on a fresh runner downloads the assets, and each of these steps passes:

- `sha256sum -c SHA256SUMS`
- `gh attestation verify` on every asset
- schema validation of the manifest
- `tools/fetch.py`, which checks every file hash and has a tampered-byte control that must fail
- the coverage check: every launch kind, and all six file-determined outcomes, appear

The determinism workflow must be green on the tagged commit, and the archive rebuilt from the tag must be byte-identical. C06 loading the release is a tracker follow-up, not a precondition (ADR-0049 G1).
