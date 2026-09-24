# 0003. Release format, checksums and how consumers use a release

- **Status:** proposed
- **Date:** 2026-09-24
- **Component:** C04

## Context

The playbook says consumers depend across repositories only on published artefacts, never on a git reference. The engine (C06) and identification (C05) run the corpus in the platform's private CI. The SDKs (C18), the CLI (C24), integrations (C25) and customers run it in their own CI. The brief puts releases on GitHub Releases. We checked GitHub's documentation on 2026-09-24: each asset must be under 2 GiB, a release can hold up to 1,000 assets, and there is no limit on total size or bandwidth. The repository's immutable-releases setting is currently off (`GET /repos/Vetload/vetload-corpus/immutable-releases` returns `enabled: false`).

## Decision

**Versions and tags.** Semver, tagged `vX.Y.Z` on a commit on `main`, with meanings as in [ADR-0002](0002-expected-result-versioning.md). The seed corpus for gate G1 is `v0.1.0`. `v1.0.0` is the public launch release in P1. Pushing a tag starts the release workflow. That workflow regenerates everything in the pinned generator image, checks every hash against the committed `manifest.json`, and only then uploads.

**Assets of every release** (names are fixed; `X.Y.Z` has no leading `v`):

| Asset | Contents |
| --- | --- |
| `vetload-corpus-X.Y.Z.tar.gz` | One top directory, `vetload-corpus-X.Y.Z/`, containing `manifest.json`, `files/<path>`, `LICENSES/`, `NOTICE.md` and `README.md` |
| `manifest.json` | The same manifest, so a consumer can read it without downloading everything |
| `manifest.schema.json` | The JSON Schema 2020-12 the manifest validates against |
| `expectations-diff.json` | Entries added, and expectations changed or removed, since the previous release (from `0.2.0`) |
| `SHA256SUMS` | GNU coreutils format: one line per other asset, `<64 lowercase hex><two spaces><asset name>`, sorted by name, LF line endings. Checked with `sha256sum -c SHA256SUMS` |
| `vetload-corpus-av-X.Y.Z.tar.gz` | From P1: EICAR antivirus test files only, kept apart so security tools do not quarantine the main archive |
| `benchmark-X.Y.Z.json` | From P1: detector benchmark results for the website (C19) |

**Reproducible archive.** Entries are sorted by path. Only regular files and directories are allowed, with no links or devices. Modes are 0644 and 0755, uid and gid are 0 with empty names, and mtime is `SOURCE_DATE_EPOCH`, the tagged commit's time. The gzip header has mtime 0 and no file name. Rebuilding from the tag gives an identical `.tar.gz`, and CI checks this. gzip is chosen over zstd because decompression is built into .NET, Node and Python without extra packages, and the `tar` command is on every CI runner.

**Provenance.** The release workflow attests every asset with `actions/attest-build-provenance` (v4.2.2 was the latest release on 2026-09-24). Anyone can verify with `gh attestation verify <asset> -R Vetload/vetload-corpus`. We ask the founder to turn on immutable releases for this repository, so that published assets and tags cannot be replaced.

**How consumers use a release.** This is the rule for C05, C06 and every later consumer:

1. Pin the version and the SHA-256 of `vetload-corpus-X.Y.Z.tar.gz` in the consumer's repository. C06 chooses the file, for example `engine/tests/corpus.lock`.
2. Download `https://github.com/Vetload/vetload-corpus/releases/download/vX.Y.Z/vetload-corpus-X.Y.Z.tar.gz`.
3. Check the archive against **the pin**. `SHA256SUMS` from the same release only proves the download was not damaged; it does not prove the archive is the one you pinned.
4. Extract safely: reject absolute paths, `..` segments, links and device entries.
5. Check each file's `sha256` and `bytes` against `manifest.json`. Fail if a listed file is missing, and fail if a file is present that is not listed.
6. Keep a positive control in the consumer's tests: flip one byte in a copy and assert that step 5 fails.
7. Cache by the pinned hash, for example as an `actions/cache` key.
8. Run the engine or library with the manifest's `test_limits`, apply the capability gating and flag subsetting from ADR-0002, and compare.

C05 compares kind, MIME and stored dimensions only, for both its C# library and its browser package. C06 compares every expected field and keeps its full golden documents keyed by `path`.

## Options considered

| Option | For | Against |
| --- | --- | --- |
| GitHub Releases, tar.gz, `SHA256SUMS`, attestations (chosen) | Free with no bandwidth charge. Supported by every runtime. Verifiable | 2 GiB per asset. Fine for a corpus of a few hundred MB |
| Files served from S3 and CloudFront | Per-file HTTP access for the sandbox | The brief puts serving with C10 and C02. It costs money. It can still be fed from these releases later |
| One asset per file | Consumers can download a subset | More than 1,000 assets breaks the per-release limit at P1 scale. Slow uploads |
| zstd or zip | Better ratio; zip has random access | zstd needs extra packages on some runtimes. zip metadata is harder to make reproducible |
| A git submodule or LFS | Simple | Violates the rule against depending on git references. LFS bandwidth is metered |

## Consequences

- Easier: one pinned hash per consumer makes the whole corpus tamper-evident, and upgrades are ordinary pull requests.
- Harder: consumers need a small fetch-and-verify step. We will publish a reference script in this repository (`tools/fetch.py`, CC0) so every consumer does not rewrite it.
- **Cost:** $0 at idle and at scale. Expected size is under 50 MB compressed for the seed and under 300 MB at P1. Neither is measured yet.

## Revisit when

- An asset nears 1 GiB. Split it by category into several archives.
- Customers need per-file HTTP URLs for the sandbox. C10 and C02 then mirror a release to the CDN, and this format stays the source.
