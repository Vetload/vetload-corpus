# Vetload corpus

The upload torture corpus: awkward, real-shaped files with the results a truthful file inspector must return. Mislabelled images, sideways phone photos, truncated video, encrypted and booby-trapped documents, polyglots and more, each generated reproducibly or carefully licensed.

**Status:** being set up. Generators, the manifest and the first release arrive in the first build wave.

## What you will find

- Generators that rebuild every file deterministically, verified by checksum in CI.
- `manifest.json` with the expected kind, type, outcome and key facts for every file.
- Releases with checksums, free to use in your own CI.
- A benchmark of common open-source file-type detectors against the corpus.

No real malware and no real personal data: antivirus fixtures use the standard EICAR test string, and decompression-bomb fixtures only declare huge sizes.

## Licence

Generated files are dedicated to the public domain under CC0-1.0 (see `LICENSE`). Third-party samples keep their own licence, recorded per file in the manifest.
