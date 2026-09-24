# Seed corpus: the first files and their expected results

**Status: proposal for review with C06 (engine core) and C05 (identification).** This is the list for corpus release `v0.1.0`, the gate G1 deliverable. Nothing here is generated yet; the numbers are what the generators will be written to produce. The tables and the coverage counts were produced from a single list by a throwaway script, so they agree. In milestone M1 that list becomes the generator specs, and this page is generated from them.

## How to read the table

- **Path** is the file's stable ID inside `files/` of the release. Its basename is the claimed file name. Unless the row says otherwise, the harness sends the claimed MIME conventionally paired with the extension, and no claimed MIME when there is no extension.
- **Outcome** is what a fully capable engine returns when it supports the verified MIME ([ADR-0002](architecture/decisions/0002-expected-result-versioning.md)). The reason follows the slash.
- **†** marks an expectation that is a judgement call. It needs agreement from C06 and the owning kind component (C11 images, C12 video and audio, C13 documents) before `v0.1.0` is tagged. If they disagree, the file stays and its expectation changes; nothing is dropped silently.
- **‡** marks a file whose verified MIME is not in the launch kind set. An engine that does not support it yet must return `unsupported_type` with reason `recognised_unsupported`, and the consumer derives that from its own capability list. The file still carries the full truth for later phases.
- **Kind and MIME** are `null` when nothing matched ([ADR-0049](https://github.com/Vetload/vetload-platform/blob/main/docs/architecture/decisions/0049-wave-1-alignment.md) A3).
- **Mismatch** compares C05 format IDs, so an audio-only MP4 named `.mp4` and an APNG named `.png` are not mismatches. It is `null` when the claim carries no type information, such as no extension and no claimed MIME, or when nothing could be identified.
- Outcome, reason, kind and risk-flag values are C01's (`contracts/codes/*.yaml`, ADR-0049 A1 and A2); a zero-byte file is `unsupported_type` with reason `empty_file`.
- **Test limits** are part of the manifest: `max_file_bytes` 10,485,760, `max_image_pixels` 100,000,000, and `max_image_width` and `max_image_height` 65,535. A value exactly at a limit passes. They are test values chosen to keep the release small, not plan limits. A consumer configures its engine with them when it runs the corpus.
- Dimensions are 1024x768 for still images, 640x360 for video and 3 seconds for media, unless the row says otherwise.

## Coverage

Total: **167 files**, of which 25 marked † and 12 marked ‡.

| Outcome | Files |
| --- | --- |
| `corrupt_media` | 23 |
| `encrypted` | 4 |
| `file_too_large` | 1 |
| `image_too_large` | 7 |
| `success` | 125 |
| `unsupported_type` | 7 |

| Kind | Files |
| --- | --- |
| archive | 1 |
| audio | 12 |
| document | 35 |
| image | 89 |
| null | 5 |
| text | 8 |
| video | 17 |

| Reason | Files |
| --- | --- |
| `dimension_limit` | 1 |
| `empty_file` | 2 |
| `malformed` | 9 |
| `password_required` | 4 |
| `pixel_limit` | 6 |
| `recognised_unsupported` | 2 |
| `size_limit` | 1 |
| `truncated` | 14 |
| `unrecognised` | 3 |

Risk flag codes used: `csv_formula_injection`, `office_dde`, `office_external_references`, `office_macros`, `office_ole_objects`, `pdf_embedded_files`, `pdf_javascript`, `polyglot`, `trailing_data`.

Verified MIME values used: `application/pdf`, `application/vnd.ms-excel.sheet.macroEnabled.12`, `application/vnd.ms-word.document.macroEnabled.12`, `application/vnd.openxmlformats-officedocument.presentationml.presentation`, `application/vnd.openxmlformats-officedocument.spreadsheetml.sheet`, `application/vnd.openxmlformats-officedocument.wordprocessingml.document`, `application/zip`, `audio/flac`, `audio/mp4`, `audio/mpeg`, `audio/ogg`, `audio/wav`, `image/avif`, `image/bmp`, `image/gif`, `image/heic`, `image/jpeg`, `image/jxl`, `image/png`, `image/svg+xml`, `image/tiff`, `image/vnd.adobe.photoshop`, `image/webp`, `image/x-xcf`, `text/csv`, `text/plain`, `video/mp4`, `video/quicktime`, `video/webm`, `video/x-matroska`. The verified MIME of the two encrypted OOXML packages is the value C05's `formats.json` assigns to a CFB-encrypted package, still pending. Secondary signatures in facts also use `application/java-archive` and `text/html`.

## Files

### Baseline: one plain file per launch format

| # | Path | What it is | Outcome | Kind | Verified MIME | Mismatch | Key facts and flags |
| --- | --- | --- | --- | --- | --- | --- | --- |
| 1 | `baseline/jpeg-baseline.jpg` | Baseline JPEG, 4:2:0, sRGB, 1024x768 | `success` | image | `image/jpeg` | no | 1024x768; progressive=false; rgb 8-bit |
| 2 | `baseline/jpeg-progressive.jpg` | Progressive JPEG | `success` | image | `image/jpeg` | no | 1024x768; progressive=true |
| 3 | `baseline/png-rgb8.png` | PNG truecolour 8-bit | `success` | image | `image/png` | no | 1024x768; rgb 8-bit; alpha=false |
| 4 | `baseline/png-rgba8.png` | PNG with alpha | `success` | image | `image/png` | no | 1024x768; alpha=true |
| 5 | `baseline/png-palette.png` | Palette PNG | `success` | image | `image/png` | no | 1024x768; color_model=palette |
| 6 | `baseline/gif-static.gif` | Single-frame GIF | `success` | image | `image/gif` | no | 320x240; frames=1; animated=false |
| 7 | `baseline/webp-lossy.webp` | Lossy WebP (VP8) | `success` | image | `image/webp` | no | 1024x768 |
| 8 | `baseline/webp-lossless-alpha.webp` | Lossless WebP with alpha (VP8L) | `success` | image | `image/webp` | no | 1024x768; alpha=true |
| 9 | `baseline/tiff-rgb-deflate.tif` | TIFF, RGB, Deflate | `success` | image | `image/tiff` | no | 1024x768 |
| 10 | `baseline/bmp-24bit.bmp` | 24-bit BMP | `success` | image | `image/bmp` | no | 1024x768 |
| 11 | `baseline/heic-still.heic` | HEIC still image, 8-bit | `success` | image | `image/heic` | no | 1024x768; bit_depth=8 |
| 12 | `baseline/avif-still.avif` | AVIF still image, 8-bit | `success` | image | `image/avif` | no | 1024x768; bit_depth=8 |
| 13 | `baseline/mp4-h264-aac.mp4` | MP4, H.264 + AAC, moov first, 3 s | `success` | video | `video/mp4` | no | 640x360; fast_start=true; h264/aac; duration_ms=3000 |
| 14 | `baseline/mov-h264-aac.mov` | QuickTime, H.264 + AAC, 3 s | `success` | video | `video/quicktime` | no | 640x360; h264/aac |
| 15 | `baseline/webm-vp9-opus.webm` | WebM, VP9 + Opus, 3 s | `success` | video | `video/webm` | no | 640x360; vp9/opus |
| 16 | `baseline/mp3-cbr-128k.mp3` | MP3, CBR 128 kbit/s, 3 s | `success` | audio | `audio/mpeg` | no | 44100 Hz; 2 ch; duration_ms=3000 |
| 17 | `baseline/m4a-aac.m4a` | AAC in MP4 (M4A), 3 s | `success` | audio | `audio/mp4` | no | 44100 Hz; 2 ch; aac |
| 18 | `baseline/flac-16bit.flac` | FLAC, 16-bit, 3 s | `success` | audio | `audio/flac` | no | 44100 Hz; bit_depth=16 |
| 19 | `baseline/wav-pcm16.wav` | WAV, PCM 16-bit, 3 s | `success` | audio | `audio/wav` | no | 44100 Hz; bit_depth=16 |
| 20 | `baseline/ogg-vorbis.ogg` | Ogg Vorbis, 3 s | `success` | audio | `audio/ogg` | no | 44100 Hz; vorbis |
| 21 | `baseline/pdf-one-page.pdf` | PDF 1.7, one A4 page with text | `success` | document | `application/pdf` | no | page_count=1 |
| 22 | `baseline/pdf-three-pages-mixed-sizes.pdf` | PDF, three pages: A4 portrait, Letter, A4 landscape | `success` | document | `application/pdf` | no | page_count=3 |
| 23 | `baseline/docx-two-pages.docx` | DOCX, two pages, app.xml page count set | `success` | document | `application/vnd.openxmlformats-officedocument.wordprocessingml.document` | no | page_count=2 |
| 24 | `baseline/xlsx-two-sheets.xlsx` | XLSX, two sheets | `success` | document | `application/vnd.openxmlformats-officedocument.spreadsheetml.sheet` | no |  |
| 25 | `baseline/pptx-three-slides.pptx` | PPTX, three slides | `success` | document | `application/vnd.openxmlformats-officedocument.presentationml.presentation` | no | page_count=3 |

### Mislabelled and misleading names

| # | Path | What it is | Outcome | Kind | Verified MIME | Mismatch | Key facts and flags |
| --- | --- | --- | --- | --- | --- | --- | --- |
| 26 | `mislabelled/png-named-jpg.jpg` | PNG sent as .jpg with image/jpeg | `success` | image | `image/png` | yes |  |
| 27 | `mislabelled/jpeg-named-png.png` | JPEG sent as .png | `success` | image | `image/jpeg` | yes |  |
| 28 | `mislabelled/heic-named-jpg.jpg` | HEIC sent as .jpg, the everyday iPhone case | `success` | image | `image/heic` | yes |  |
| 29 | `mislabelled/jpeg-named-heic.HEIC` | HEIC already converted to JPEG, name kept | `success` | image | `image/jpeg` | yes |  |
| 30 | `mislabelled/webp-named-jpg.jpg` | WebP sent as .jpg | `success` | image | `image/webp` | yes |  |
| 31 | `mislabelled/pdf-named-docx.docx` | PDF sent as .docx | `success` | document | `application/pdf` | yes |  |
| 32 | `mislabelled/docx-named-pdf.pdf` | DOCX sent as .pdf | `success` | document | `application/vnd.openxmlformats-officedocument.wordprocessingml.document` | yes |  |
| 33 | `mislabelled/webm-named-mp4.mp4` | WebM sent as .mp4 | `success` | video | `video/webm` | yes |  |
| 34 | `mislabelled/mp3-named-wav.wav` | MP3 sent as .wav | `success` | audio | `audio/mpeg` | yes |  |
| 35 | `mislabelled/jpeg-no-extension` | JPEG with no extension and no claimed MIME | `success` | image | `image/jpeg` | null |  |
| 36 | `mislabelled/pdf-no-extension` | PDF with no extension, claimed application/octet-stream | `success` † | document | `application/pdf` | null |  |
| 37 | `mislabelled/PHOTO-UPPERCASE.JPG` | JPEG with an upper-case extension | `success` | image | `image/jpeg` | no |  |
| 38 | `mislabelled/jpeg-alias-extension.jpeg` | JPEG with the .jpeg alias | `success` | image | `image/jpeg` | no |  |
| 39 | `mislabelled/invoice.pdf.exe` | PDF content behind a double extension ending .exe | `success` | document | `application/pdf` | yes |  |
| 40 | `mislabelled/png-right-name-wrong-mime.png` | PNG with the right name but claimed image/jpeg | `success` | image | `image/png` | yes |  |
| 41 | `mislabelled/docm-named-docx.docx` | Macro-enabled document sent as .docx | `success` | document | `application/vnd.ms-word.document.macroEnabled.12` | yes | flags: office_macros |

### EXIF and container orientation

| # | Path | What it is | Outcome | Kind | Verified MIME | Mismatch | Key facts and flags |
| --- | --- | --- | --- | --- | --- | --- | --- |
| 42 | `orientation/jpeg-orientation-1.jpg` | JPEG 640x480 with EXIF orientation 1 | `success` | image | `image/jpeg` | no | orientation=1; stored 640x480; display 640x480 |
| 43 | `orientation/jpeg-orientation-2.jpg` | JPEG 640x480 with EXIF orientation 2 | `success` | image | `image/jpeg` | no | orientation=2; stored 640x480; display 640x480 |
| 44 | `orientation/jpeg-orientation-3.jpg` | JPEG 640x480 with EXIF orientation 3 | `success` | image | `image/jpeg` | no | orientation=3; stored 640x480; display 640x480 |
| 45 | `orientation/jpeg-orientation-4.jpg` | JPEG 640x480 with EXIF orientation 4 | `success` | image | `image/jpeg` | no | orientation=4; stored 640x480; display 640x480 |
| 46 | `orientation/jpeg-orientation-5.jpg` | JPEG 640x480 with EXIF orientation 5 | `success` | image | `image/jpeg` | no | orientation=5; stored 640x480; display 480x640 |
| 47 | `orientation/jpeg-orientation-6.jpg` | JPEG 640x480 with EXIF orientation 6 | `success` | image | `image/jpeg` | no | orientation=6; stored 640x480; display 480x640 |
| 48 | `orientation/jpeg-orientation-7.jpg` | JPEG 640x480 with EXIF orientation 7 | `success` | image | `image/jpeg` | no | orientation=7; stored 640x480; display 480x640 |
| 49 | `orientation/jpeg-orientation-8.jpg` | JPEG 640x480 with EXIF orientation 8 | `success` | image | `image/jpeg` | no | orientation=8; stored 640x480; display 480x640 |
| 50 | `orientation/jpeg-orientation-invalid-9.jpg` | EXIF orientation value 9, outside the spec | `success` † | image | `image/jpeg` | no | orientation=null; display = stored |
| 51 | `orientation/heic-irot-90.heic` | HEIC rotated by an irot property, no EXIF orientation | `success` † | image | `image/heic` | no | stored 640x480; display 480x640 |
| 52 | `orientation/png-exif-orientation-6.png` | PNG with an eXIf chunk, orientation 6 | `success` † | image | `image/png` | no | orientation=6; display 480x640 |
| 53 | `orientation/tiff-orientation-6.tif` | TIFF with Orientation tag 6 | `success` | image | `image/tiff` | no | orientation=6; display 480x640 |

### Colour

| # | Path | What it is | Outcome | Kind | Verified MIME | Mismatch | Key facts and flags |
| --- | --- | --- | --- | --- | --- | --- | --- |
| 54 | `colour/jpeg-cmyk.jpg` | CMYK JPEG | `success` | image | `image/jpeg` | no | color_model=cmyk |
| 55 | `colour/jpeg-cmyk-adobe-inverted.jpg` | CMYK JPEG with Adobe APP14 inverted values | `success` | image | `image/jpeg` | no | color_model=cmyk |
| 56 | `colour/tiff-cmyk.tif` | CMYK TIFF | `success` | image | `image/tiff` | no | color_model=cmyk |
| 57 | `colour/jpeg-display-p3.jpg` | JPEG tagged with a Display P3 profile (CC0 compact profile) | `success` | image | `image/jpeg` | no | icc_profile set |
| 58 | `colour/png-display-p3-iccp.png` | PNG with a Display P3 iCCP chunk | `success` | image | `image/png` | no | icc_profile set |
| 59 | `colour/jpeg-icc-split-across-app2.jpg` | JPEG whose ICC profile spans several APP2 segments | `success` | image | `image/jpeg` | no | icc_profile set |
| 60 | `colour/jpeg-grayscale.jpg` | Greyscale JPEG | `success` | image | `image/jpeg` | no | color_model=gray |
| 61 | `colour/png-rgb16.png` | 16-bit truecolour PNG | `success` | image | `image/png` | no | bit_depth=16 |
| 62 | `colour/png-gray16.png` | 16-bit greyscale PNG | `success` | image | `image/png` | no | color_model=gray; bit_depth=16 |
| 63 | `colour/heic-10bit.heic` | 10-bit HEIC | `success` | image | `image/heic` | no | bit_depth=10 |
| 64 | `colour/avif-10bit-alpha.avif` | 10-bit AVIF with alpha | `success` | image | `image/avif` | no | bit_depth=10; alpha=true |

### Animation

| # | Path | What it is | Outcome | Kind | Verified MIME | Mismatch | Key facts and flags |
| --- | --- | --- | --- | --- | --- | --- | --- |
| 65 | `animation/gif-animated-loop-forever.gif` | Animated GIF, 12 frames, NETSCAPE loop 0 | `success` | image | `image/gif` | no | frames=12; animated=true; loop_count=0 |
| 66 | `animation/gif-zero-delay-frames.gif` | Animated GIF with zero frame delays | `success` | image | `image/gif` | no | frames=5; animated=true |
| 67 | `animation/webp-animated.webp` | Animated WebP, 10 frames | `success` | image | `image/webp` | no | frames=10; animated=true |
| 68 | `animation/apng-animated.png` | APNG, 8 frames, named .png; not a mismatch (same C05 format); MIME follows C05 | `success` † | image | `image/png` | no | frames=8; animated=true |
| 69 | `animation/gif-single-frame-with-loop-ext.gif` | One frame, but with a NETSCAPE loop extension | `success` | image | `image/gif` | no | frames=1; animated=false |

### Limits (test limits: 10 MiB, 100,000,000 pixels, 65,535 per side)

| # | Path | What it is | Outcome | Kind | Verified MIME | Mismatch | Key facts and flags |
| --- | --- | --- | --- | --- | --- | --- | --- |
| 70 | `limits/png-header-60000x60000.png` | IHDR declares 3.6 gigapixels, tiny IDAT; not a live bomb | `image_too_large / pixel_limit` | image | `image/png` | no | stored 60000x60000 |
| 71 | `limits/png-100000x10-too-wide.png` | 100,000 pixels wide, 1 MP: over the side limit only | `image_too_large / dimension_limit` | image | `image/png` | no | stored 100000x10 |
| 72 | `limits/jpeg-sof-65500x65500.jpg` | SOF declares 65500x65500, almost no scan data | `image_too_large / pixel_limit` | image | `image/jpeg` | no | stored 65500x65500 |
| 73 | `limits/gif-screen-65535x65535.gif` | GIF logical screen and frame of 65535x65535, exactly at the side limit | `image_too_large / pixel_limit` † | image | `image/gif` | no | stored 65535x65535 |
| 74 | `limits/webp-vp8x-canvas-16383x16383.webp` | VP8X canvas 16383x16383 (268 MP) | `image_too_large / pixel_limit` | image | `image/webp` | no | stored 16383x16383 |
| 75 | `limits/tiff-header-60000x60000.tif` | TIFF IFD declares 60000x60000 | `image_too_large / pixel_limit` | image | `image/tiff` | no | stored 60000x60000 |
| 76 | `limits/heic-ispe-65535x65535.heic` | HEIC ispe declares 65535x65535 | `image_too_large / pixel_limit` † | image | `image/heic` | no | stored 65535x65535 |
| 77 | `limits/png-10000x10000-at-limit.png` | Exactly 100,000,000 pixels, solid colour | `success` † | image | `image/png` | no | stored 10000x10000 |
| 78 | `limits/jpeg-40000x100-wide.jpg` | 40,000-pixel-wide panorama, 4 MP | `success` | image | `image/jpeg` | no | stored 40000x100 |
| 79 | `limits/bmp-12mib-over-size-limit.bmp` | 2048x2048 BMP, 12,582,966 bytes, over the 10 MiB test limit | `file_too_large / size_limit` | image | `image/bmp` | no |  |

### Empty and tiny

| # | Path | What it is | Outcome | Kind | Verified MIME | Mismatch | Key facts and flags |
| --- | --- | --- | --- | --- | --- | --- | --- |
| 80 | `empty/zero-bytes.jpg` | Zero bytes, named .jpg | `unsupported_type / empty_file` | `null` | `null` | null |  |
| 81 | `empty/upload` | Zero bytes, no extension, no claimed MIME | `unsupported_type / empty_file` | `null` | `null` | null |  |
| 82 | `empty/one-byte.png` | One byte, 0x89 | `unsupported_type / unrecognised` † | `null` | `null` | null |  |
| 83 | `empty/jpeg-soi-only.jpg` | FF D8 FF and nothing else | `corrupt_media / truncated` † | image | `image/jpeg` | no |  |
| 84 | `empty/png-signature-only.png` | PNG signature, no chunks | `corrupt_media / truncated` | image | `image/png` | no |  |
| 85 | `empty/pdf-header-only.pdf` | %PDF-1.7 header line only | `corrupt_media / truncated` | document | `application/pdf` | no |  |

### Truncated

| # | Path | What it is | Outcome | Kind | Verified MIME | Mismatch | Key facts and flags |
| --- | --- | --- | --- | --- | --- | --- | --- |
| 86 | `truncated/jpeg-cut-at-50-percent.jpg` | Baseline JPEG cut at half its length | `corrupt_media / truncated` | image | `image/jpeg` | no | stored 1024x768 |
| 87 | `truncated/jpeg-cut-before-sos.jpg` | Headers complete, cut before scan data | `corrupt_media / truncated` | image | `image/jpeg` | no | stored 1024x768 |
| 88 | `truncated/png-cut-mid-idat.png` | PNG cut inside IDAT | `corrupt_media / truncated` | image | `image/png` | no | stored 1024x768 |
| 89 | `truncated/png-missing-iend.png` | All image data present, IEND missing | `success` † | image | `image/png` | no | stored 1024x768 |
| 90 | `truncated/gif-missing-trailer.gif` | Complete GIF without the 0x3B trailer | `success` † | image | `image/gif` | no |  |
| 91 | `truncated/webp-cut-at-60-percent.webp` | Lossy WebP cut at 60 percent | `corrupt_media / truncated` | image | `image/webp` | no |  |
| 92 | `truncated/heic-cut-in-mdat.heic` | HEIC cut inside mdat | `corrupt_media / truncated` | image | `image/heic` | no |  |
| 93 | `truncated/mp4-faststart-cut-in-mdat.mp4` | moov first, cut inside mdat | `corrupt_media / truncated` | video | `video/mp4` | no |  |
| 94 | `truncated/mp4-moov-at-end-missing-moov.mp4` | moov last, cut before it: no index at all | `corrupt_media / truncated` | video | `video/mp4` | no |  |
| 95 | `truncated/webm-cut-at-50-percent.webm` | WebM cut at half | `corrupt_media / truncated` | video | `video/webm` | no |  |
| 96 | `truncated/pdf-missing-xref-and-eof.pdf` | PDF cut before xref, trailer and %%EOF | `corrupt_media / truncated` | document | `application/pdf` | no |  |
| 97 | `truncated/docx-missing-central-directory.docx` | DOCX cut before the ZIP central directory | `corrupt_media / truncated` † | document | `application/vnd.openxmlformats-officedocument.wordprocessingml.document` | no |  |
| 98 | `truncated/mp3-vbr-xing-cut.mp3` | VBR MP3 whose Xing frame count exceeds the frames present | `corrupt_media / truncated` † | audio | `audio/mpeg` | no |  |

### Malformed

| # | Path | What it is | Outcome | Kind | Verified MIME | Mismatch | Key facts and flags |
| --- | --- | --- | --- | --- | --- | --- | --- |
| 99 | `malformed/jpeg-bad-huffman-table.jpg` | DHT with an impossible code-length set | `corrupt_media / malformed` | image | `image/jpeg` | no |  |
| 100 | `malformed/png-ihdr-bad-crc.png` | IHDR CRC wrong | `corrupt_media / malformed` † | image | `image/png` | no |  |
| 101 | `malformed/png-idat-zlib-corrupt.png` | IDAT zlib stream corrupted, CRCs fixed up | `corrupt_media / malformed` | image | `image/png` | no |  |
| 102 | `malformed/gif-invalid-lzw-min-code-size.gif` | LZW minimum code size 12 | `corrupt_media / malformed` | image | `image/gif` | no |  |
| 103 | `malformed/webp-vp8-bad-frame-header.webp` | VP8 frame header with a bad start code | `corrupt_media / malformed` | image | `image/webp` | no |  |
| 104 | `malformed/mp4-child-box-larger-than-parent.mp4` | trak box larger than its moov | `corrupt_media / malformed` | video | `video/mp4` | no |  |
| 105 | `malformed/pdf-garbage-after-header.pdf` | %PDF header followed by random bytes | `corrupt_media / malformed` | document | `application/pdf` | no |  |
| 106 | `malformed/pdf-broken-xref-offsets.pdf` | Every xref offset wrong; objects intact and recoverable | `success` † | document | `application/pdf` | no | page_count=1 |
| 107 | `malformed/docx-invalid-document-xml.docx` | word/document.xml is not well-formed XML | `corrupt_media / malformed` | document | `application/vnd.openxmlformats-officedocument.wordprocessingml.document` | no |  |
| 108 | `malformed/docx-zip-crc-mismatch.docx` | CRC-32 of word/document.xml wrong | `corrupt_media / malformed` † | document | `application/vnd.openxmlformats-officedocument.wordprocessingml.document` | no |  |

### Encrypted

| # | Path | What it is | Outcome | Kind | Verified MIME | Mismatch | Key facts and flags |
| --- | --- | --- | --- | --- | --- | --- | --- |
| 109 | `encrypted/pdf-aes256-user-password.pdf` | AES-256 with a user password | `encrypted / password_required` | document | `application/pdf` | no |  |
| 110 | `encrypted/pdf-rc4-128-user-password.pdf` | RC4 128-bit with a user password | `encrypted / password_required` | document | `application/pdf` | no |  |
| 111 | `encrypted/pdf-owner-password-only.pdf` | Owner password only; opens without a password | `success` | document | `application/pdf` | no | pdf_permissions_restricted=true; page_count=1 |
| 112 | `encrypted/docx-agile-encrypted.docx` | ECMA-376 agile encryption: a CFB container | `encrypted / password_required` † | document | C05 CFB MIME, pending | no |  |
| 113 | `encrypted/xlsx-agile-encrypted.xlsx` | ECMA-376 agile encryption: a CFB container | `encrypted / password_required` † | document | C05 CFB MIME, pending | no |  |

### Document risk

| # | Path | What it is | Outcome | Kind | Verified MIME | Mismatch | Key facts and flags |
| --- | --- | --- | --- | --- | --- | --- | --- |
| 114 | `document_risk/pdf-javascript-openaction.pdf` | OpenAction runs JavaScript (app.alert only) | `success` | document | `application/pdf` | no | flags: pdf_javascript |
| 115 | `document_risk/pdf-embedded-file.pdf` | One embedded text attachment | `success` | document | `application/pdf` | no | flags: pdf_embedded_files |
| 116 | `document_risk/pdf-javascript-and-embedded-file.pdf` | Both of the above | `success` | document | `application/pdf` | no | flags: pdf_javascript, pdf_embedded_files |
| 117 | `document_risk/docm-with-macro.docm` | Benign VBA project, no auto-run entry point | `success` | document | `application/vnd.ms-word.document.macroEnabled.12` | no | flags: office_macros |
| 118 | `document_risk/xlsm-with-macro.xlsm` | Benign VBA project, no auto-run entry point | `success` | document | `application/vnd.ms-excel.sheet.macroEnabled.12` | no | flags: office_macros |
| 119 | `document_risk/docx-remote-template.docx` | attachedTemplate points at https://example.invalid | `success` | document | `application/vnd.openxmlformats-officedocument.wordprocessingml.document` | no | flags: office_external_references |
| 120 | `document_risk/docx-external-image-link.docx` | Image relationship with TargetMode External | `success` | document | `application/vnd.openxmlformats-officedocument.wordprocessingml.document` | no | flags: office_external_references |
| 121 | `document_risk/xlsx-external-workbook-link.xlsx` | externalLink part to another workbook | `success` | document | `application/vnd.openxmlformats-officedocument.spreadsheetml.sheet` | no | flags: office_external_references |
| 122 | `document_risk/docx-ddeauto-field.docx` | DDEAUTO field with an inert command | `success` | document | `application/vnd.openxmlformats-officedocument.wordprocessingml.document` | no | flags: office_dde |
| 123 | `document_risk/xlsx-dde-formula.xlsx` | Cell formula using DDE, inert | `success` | document | `application/vnd.openxmlformats-officedocument.spreadsheetml.sheet` | no | flags: office_dde |
| 124 | `document_risk/docx-embedded-ole-object.docx` | Embedded OLE object holding plain text | `success` | document | `application/vnd.openxmlformats-officedocument.wordprocessingml.document` | no | flags: office_ole_objects |

### Polyglots and trailing data

| # | Path | What it is | Outcome | Kind | Verified MIME | Mismatch | Key facts and flags |
| --- | --- | --- | --- | --- | --- | --- | --- |
| 125 | `polyglot/gif-plus-jar.gif` | GIF followed by a JAR whose central directory resolves (GIFAR) | `success` | image | `image/gif` | no | flags: polyglot, trailing_data; secondary application/java-archive |
| 126 | `polyglot/pdf-plus-zip.pdf` | PDF with a valid ZIP after %%EOF | `success` | document | `application/pdf` | no | flags: polyglot, trailing_data; secondary application/zip |
| 127 | `polyglot/jpeg-plus-zip.jpg` | JPEG with a ZIP appended after EOI | `success` | image | `image/jpeg` | no | flags: polyglot, trailing_data; secondary application/zip |
| 128 | `polyglot/png-plus-zip.png` | PNG with a ZIP appended after IEND | `success` | image | `image/png` | no | flags: polyglot, trailing_data; secondary application/zip |
| 129 | `trailing_data/jpeg-trailing-html-script.jpg` | HTML with a script tag after EOI | `success` | image | `image/jpeg` | no | flags: trailing_data; secondary text/html; trailing_bytes exact |
| 130 | `trailing_data/png-trailing-random-bytes.png` | 4 KiB of seeded random bytes after IEND | `success` | image | `image/png` | no | flags: trailing_data; trailing_bytes=4096 |
| 131 | `trailing_data/pdf-bytes-after-eof.pdf` | Binary bytes after the final %%EOF | `success` | document | `application/pdf` | no | flags: trailing_data |
| 132 | `trailing_data/mp4-trailing-garbage.mp4` | Bytes after the last top-level box | `success` | video | `video/mp4` | no | flags: trailing_data |
| 133 | `trailing_data/jpeg-zero-padding-after-eoi.jpg` | Control: 16 zero bytes of padding after EOI | `success` † | image | `image/jpeg` | no | no flags; trailing_bytes=16 |

### Video

| # | Path | What it is | Outcome | Kind | Verified MIME | Mismatch | Key facts and flags |
| --- | --- | --- | --- | --- | --- | --- | --- |
| 134 | `video/mp4-moov-at-end.mp4` | Index after media data | `success` | video | `video/mp4` | no | fast_start=false |
| 135 | `video/mp4-fragmented.mp4` | Fragmented MP4 (moof boxes) | `success` | video | `video/mp4` | no | fragmented=true |
| 136 | `video/mov-rotated-90.mov` | Display matrix rotates 90 degrees | `success` | video | `video/quicktime` | no | video_rotation=90; stored 640x360; display 360x640 |
| 137 | `video/mp4-vfr.mp4` | Variable frame rate | `success` | video | `video/mp4` | no | vfr=true |
| 138 | `video/mp4-audio-only.mp4` | Audio-only MP4 named .mp4; not a mismatch (same C05 format) | `success` | audio | `audio/mp4` | no |  |
| 139 | `video/mp4-hevc-hvc1.mp4` | HEVC (hvc1) + AAC | `success` | video | `video/mp4` | no | video_codec=hevc |
| 140 | `video/mp4-two-audio-tracks-and-subtitles.mp4` | Two audio tracks and a tx3g subtitle track | `success` | video | `video/mp4` | no | audio_streams=2; subtitle_streams=1 |
| 141 | `video/mp4-video-only.mp4` | Video track only | `success` | video | `video/mp4` | no | audio_streams=0 |

### Audio

| # | Path | What it is | Outcome | Kind | Verified MIME | Mismatch | Key facts and flags |
| --- | --- | --- | --- | --- | --- | --- | --- |
| 142 | `audio/mp3-id3v2-cover-art.mp3` | ID3v2.4 tags with an embedded cover image | `success` | audio | `audio/mpeg` | no |  |
| 143 | `audio/wav-float32.wav` | WAV, 32-bit float | `success` | audio | `audio/wav` | no | bit_depth=32 |
| 144 | `audio/flac-24bit-96k.flac` | FLAC, 24-bit, 96 kHz | `success` | audio | `audio/flac` | no | bit_depth=24; 96000 Hz |
| 145 | `audio/ogg-opus.opus` | Ogg Opus with the .opus extension | `success` | audio | `audio/ogg` | no | audio_codec=opus |

### Text (text kind arrives in P1)

| # | Path | What it is | Outcome | Kind | Verified MIME | Mismatch | Key facts and flags |
| --- | --- | --- | --- | --- | --- | --- | --- |
| 146 | `text/csv-utf8.csv` | Plain UTF-8 CSV | `success` ‡ | text | `text/csv` | no | utf-8; delimiter , |
| 147 | `text/csv-utf8-bom.csv` | UTF-8 CSV with a BOM | `success` ‡ | text | `text/csv` | no | utf-8; bom=true |
| 148 | `text/csv-formula-injection.csv` | =HYPERLINK cell | `success` ‡ | text | `text/csv` | no | flags: csv_formula_injection |
| 149 | `text/csv-formula-injection-fullwidth.csv` | Cell starting with U+FF1D FULLWIDTH EQUALS SIGN | `success` ‡ | text | `text/csv` | no | flags: csv_formula_injection |
| 150 | `text/csv-windows-1252-claimed-utf8.csv` | Windows-1252 bytes, claimed text/csv; charset=utf-8 | `success` ‡ | text | `text/csv` | no | text_encoding=windows-1252 |
| 151 | `text/txt-utf16le-bom.txt` | UTF-16LE with a BOM | `success` ‡ | text | `text/plain` | no | text_encoding=utf-16le; bom=true |
| 152 | `text/txt-shift-jis.txt` | Shift_JIS Japanese text | `success` ‡ | text | `text/plain` | no | text_encoding=shift_jis |
| 153 | `text/txt-utf8-with-nul-bytes.txt` | UTF-8 text containing NUL bytes | `success` † ‡ | text | `text/plain` | no | text_encoding=utf-8 |

### Other kinds: never decoded, or not yet

| # | Path | What it is | Outcome | Kind | Verified MIME | Mismatch | Key facts and flags |
| --- | --- | --- | --- | --- | --- | --- | --- |
| 154 | `other/gimp-image.xcf` | GIMP XCF: recognised, never decoded | `unsupported_type / recognised_unsupported` † | image | `image/x-xcf` | no |  |
| 155 | `other/jpeg-xl.jxl` | JPEG XL: recognised, not on the kind roadmap | `unsupported_type / recognised_unsupported` † | image | `image/jxl` | no |  |
| 156 | `other/random-bytes.bin` | 4 KiB of seeded random bytes | `unsupported_type / unrecognised` | `null` | `null` | null |  |
| 157 | `other/random-bytes-named-jpg.jpg` | The same bytes named .jpg | `unsupported_type / unrecognised` † | `null` | `null` | null |  |
| 158 | `other/zip-archive.zip` | ZIP with three small text files | `success` ‡ | archive | `application/zip` | no |  |
| 159 | `other/matroska-h264.mkv` | Matroska, H.264 + AAC | `success` ‡ | video | `video/x-matroska` | no |  |
| 160 | `other/svg-simple.svg` | Plain SVG, no script | `success` ‡ | image | `image/svg+xml` | no |  |
| 161 | `other/psd-two-layers.psd` | Photoshop file with two layers and a composite | `success` ‡ | image | `image/vnd.adobe.photoshop` | no |  |

### JPEG and TIFF structure

| # | Path | What it is | Outcome | Kind | Verified MIME | Mismatch | Key facts and flags |
| --- | --- | --- | --- | --- | --- | --- | --- |
| 162 | `jpeg/jpeg-app-segments-over-64kib-before-sof.jpg` | 70 KiB of APP1/APP2 before SOF | `success` | image | `image/jpeg` | no | stored 1024x768 |
| 163 | `jpeg/jpeg-exif-gps-synthetic.jpg` | Synthetic GPS at 0,0 | `success` | image | `image/jpeg` | no | exif_gps_present=true |
| 164 | `jpeg/jpeg-arithmetic-coding.jpg` | Arithmetic-coded JPEG | `success` † | image | `image/jpeg` | no |  |
| 165 | `tiff/tiff-ifd-after-image-data.tif` | IFD written after the strips | `success` | image | `image/tiff` | no | stored 1024x768 |
| 166 | `tiff/tiff-multipage-3.tif` | Three pages | `success` | image | `image/tiff` | no | frames=3 |
| 167 | `tiff/bigtiff.tif` | BigTIFF header (version 43) | `success` | image | `image/tiff` | no |  |

## How the files are made

| Group | Method | Tools inside the generator image |
| --- | --- | --- |
| Byte-level structure: empty, limits, truncations, most malformed files, polyglots, trailing data, orientation and ICC patches, text, CSV | Pure Python writing bytes directly (`struct`, `zlib`, `zipfile` with fixed timestamps) from a base file | Python only |
| Still images | Pillow and libvips encode from a synthetic, seeded test pattern | Pillow, libvips, libheif with its encoder plugins, libavif |
| Video and audio | ffmpeg from `lavfi` test sources with bit-exact flags and metadata removed | ffmpeg |
| PDF | Hand-written PDF object syntax for risky and broken files; qpdf for encryption | qpdf |
| Office | `zipfile` with fixed timestamps and hand-written XML; a small first-party compound-file writer for VBA projects and for encrypted packages | Python only |

Colour-managed files embed a Display P3 profile from [Compact-ICC-Profiles](https://github.com/saucecontrol/Compact-ICC-Profiles), which is CC0-1.0; that is the only third-party input in the seed. No file contains real personal data, real malware or copyrighted media. The GPS fixture uses the coordinates 0,0. Macros are benign and have no auto-run entry point. External references point at `example.invalid`, which can never resolve.

## Deliberately not in the seed

| What | Why | When |
| --- | --- | --- |
| Files that make the engine time out or crash (`processing_timeout`, `processing_crash`) | A correct engine never crashes on a file, and a timeout depends on hardware and deadlines rather than on the file. C06 covers both with fault injection. See the brief conflict in `ARCHITECTURE.md` | Not planned as files |
| EICAR antivirus fixtures | Security tools in consumers' CI quarantine any archive that contains them. They ship as a separate release asset | P1 |
| Legacy `.doc`, `.xls`, `.ppt` | Needs the compound-file writer first | P0, corpus `0.2.0` |
| Animated AVIF, HDR gain maps, Live Photos, C2PA, hidden text, unapplied redactions | Later brief rows | P1, P2 |
| Camera RAW, DICOM with synthetic PHI, fonts, 3D, archives with bombs | New kinds | P3 |
