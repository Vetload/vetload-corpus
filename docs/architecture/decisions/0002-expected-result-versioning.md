# 0002. How expected results are versioned

- **Status:** proposed
- **Date:** 2026-09-24
- **Component:** C04

## Context

The brief asks how expected results are versioned when the engine deliberately changes a fact, for example when a better colour-space classifier arrives. Three facts shape the answer:

- The brief puts only **stable truths** in the corpus: kind, MIME, mismatch, outcome, reason, dimensions and risk flags. Full golden result documents live next to the engine in `engine/tests/golden/` in the platform repository, owned by C06, because they change with every engine improvement.
- **The engine gains capabilities in phases.** Text is a P1 kind, and archives, SVG and PSD are P3. A CSV is a well-formed text file whatever phase we are in. Yet a P0 engine must answer `unsupported_type` for it.
- **Some outcomes depend on limits**, such as `file_too_large` and `image_too_large`. The corpus is public and must not carry plan limits or roadmap dates.

## Decision

The `expected` block of each entry is C01's `CorpusExpectedResult` (`contracts/schemas/corpus/expected-result.schema.json`), vendored by checksum. Every outcome, reason, kind and risk-flag value comes from C01's registries, and the corpus defines none of its own. The corpus-only `facts` and `revision` sit beside `expected`, not inside it ([ADR-0049](https://github.com/Vetload/vetload-platform/blob/main/docs/architecture/decisions/0049-wave-1-alignment.md) A1, A4).

1. **Expectations describe the file, not an engine.** `expected.outcome` is what a fully capable engine returns when it supports the verified format. A consumer whose engine does not support that format expects `unsupported_type` with reason `recognised_unsupported` instead, and works that out from its own capability list. For the engine, that list is C06's capabilities export. Formats that Vetload never decodes, such as GIMP `.xcf`, carry `unsupported_type` directly.
2. **Risk flags are an exact set.** A consumer asserts `actual flags == expected flags ∩ codes my engine implements`. Flags that are not implemented yet are skipped, not failed. Any extra flag fails.
3. **Limit-dependent outcomes** are stated against the manifest's `test_limits`. Consumers configure those limits when they run the corpus.
4. **When the corpus is wrong,** the fix is a corpus change. The entry's `revision`, which covers `expected` and `facts`, goes up by one. Each release ships `expectations-diff.json`, computed by diffing against the previous manifest, so nobody has to maintain it by hand.
5. **When the engine is wrong or not finished,** the corpus does not change. The consumer records a known deviation next to its own tests. For the engine that is `engine/tests/golden/`. The deviation is removed when the engine catches up. Known deviations never live in this public repository.
6. **When the engine improves a fact the corpus does not carry,** such as a finer colour-space label, only C06's golden documents change. This is why the expected block stays small.
7. **A released file's bytes never change.** A path is immutable. To change the bytes, add a new path. A path is removed only in a major release.
8. **Version numbers** follow semver, as set out in [ADR-0003](0003-release-format-and-consumption.md): a patch release changes no bytes and no expectations; a minor release adds files or fields, or changes expectations with a revision bump; a major release breaks the schema or removes paths. Until 1.0.0, a minor release may also break the schema, and the release notes say so.
9. **Consumers pin one exact corpus version.** Upgrading is a pull request in the consumer's repository that shows the expectation diff and any new known deviations.

## Options considered

| Option | For | Against |
| --- | --- | --- |
| File truths, capability gating by the consumer, deviations kept by the consumer (chosen) | The corpus never follows engine releases. The public repository reveals no roadmap. Customers and the benchmark get one answer per file | Each consumer needs a small gating step and a deviation list |
| Engine-version ranges per expectation in the manifest | Explicit | Ties a public dataset to private engine versions and release timing. The manifest grows with every engine release |
| One expectation profile per phase (P0, P1, and so on) | No gating logic | Publishes the roadmap. Multiplies expectations. Stale after the last phase |
| No expectations in the corpus, only engine goldens | Nothing to keep in step | The SDK sandbox (C18), the CLI's `vetload corpus run` (C24) and the benchmark have no ground truth, which defeats the corpus's purpose |

## Consequences

- Easier: a corpus release never has to wait for an engine release, and the other way round. Customers read the same truth the engine is tested against.
- Harder: C06 and C05 each implement gating (a few lines) and keep a deviation list. C01 must keep reason and flag codes stable. A renamed code reaches the corpus as a new vendored `vetload-corpus-schema` version, and every affected entry gets a revision bump.
- **Cost:** none. It is data and a diff script.

## Revisit when

- A consumer's deviation list grows past about 10 percent of entries for longer than one phase. That would mean the corpus is stating things that are not stable truths.
- C01 introduces outcome semantics that depend on the request, such as policy decisions, rather than on the file.
