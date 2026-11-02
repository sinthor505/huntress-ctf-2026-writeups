# Frame of Reference

| | |
|---|---|
| Day / Lane | 18 / Metadata Forensics |
| Challenge ID | `metadata-forensics-day-18-frame-of-reference` |
| Technique | MP4 stream geometry (`tkhd` vs sample entry vs H.264 SPS crop) inside a pooled evidence bundle |
| Single Solve | Solved (journal-recorded) |
| Time Trial | Independent (ranked rerun `result.json` CORRECT; original attempt journal-recorded) |
| Mastery | Independent (ranked rerun `result.json` CORRECT; original attempt journal-recorded) |

## Summary

Day 18 added one new evidence type to the metadata lane: an MP4 video, with one question about its
primary video dimensions. The Single Solve was just that one field. The Time Trial and Mastery were
the lane's usual pooled bundle (23 fields per set) with the MP4 field added to the template. The
trap the prompt hints at, coded size versus display size, never materialised on this specimen:
`tkhd`, the `avc1` sample entry and the SPS-cropped size all agreed on `320x180`. All three modes
went CORRECT on the first submission.

## The challenge

Prompt: "Clips, previews, and screen recordings have been mixed together. Use stream geometry to
identify originals, crops, and the recording that could not have come from the claimed device."
Read literally, that suggests rotation matrices, crops and a spoofed-device recording. The actual
questions asked for none of that on the Day 18 fixtures.

- **Single Solve.** One question, `d18-video-001-media-primary-video-dimensions` (`exact_text`,
  `trim`), for `video-001.mp4`. The download was a ZIP wrapping a single 19,775-byte MP4. The
  validator syntax text only says "exact artifact-derived value", so the `WxH` spelling had to be
  chosen from convention (see below).
- **Time Trial.** 23 questions over a 15-file bundle (626,852 bytes): note, note copy and roster text
  files, JPEG, PNG, PDF, a nested ZIP, a PE, a .NET DLL, an MSI, an LNK, a DOCX, an EML, a WAV, the
  MP4, plus one synthesis question (`metadata-day-18-synthesis-sha256-order`: sort four named files
  by full-file SHA-256 and submit the names as a compact JSON array). No ELF yet.
- **Mastery.** 115 questions: five nested `evidence-01..05.zip` sets of the same 23 fields (3,134,684
  bytes), answer IDs prefixed `setN.`.

Both Time Trial and Mastery start responses reported `deadline: null` for the original attempts.

## Solution

1. **Recon.** Checked the artifact SHA-256 against the start response (it matched), unpacked the
   single MP4 and walked the ISO BMFF boxes with a stdlib-only script. The file had two tracks: one
   H.264 video and one `mp4a` audio.
2. **Compare every source of "dimensions".** For the video track the script prints three
   independent values:
   - `tkhd` width/height (16.16 fixed point): 320 x 180
   - `avc1` sample entry in `stsd`: 320 x 180
   - the H.264 SPS inside `avcC`: coded size 320 x 192 with 6 bottom crop rows, so 320 x 180 after
     cropping

   Also noted: identity `tkhd` matrix (no rotation) and a 1:1 `pasp`. The coded 320x192 is just the
   16-pixel macroblock alignment of 180 and is not the display size.
3. **Pick the spelling.** The earlier JPEG and PNG dimension fields in the lane took `WxH`
   (width, lowercase `x`, height) with no spaces, so the video field was answered the same way.

   ```text
   d18-video-001-media-primary-video-dimensions = "320x180"
   ```

   One submission, CORRECT.

The crop calculation is the part worth showing, because it is what separates "coded" from
"displayed" size:

```python
w = (r.ue() + 1) * 16;  hm = r.ue() + 1;  fmo = r.bit()
h = (2 - fmo) * hm * 16
...
if r.bit(): crop = (r.ue(), r.ue(), r.ue(), r.ue())      # left, right, top, bottom
cux = 1 if chroma in (0, 3) else 2
cuy = (1 if chroma in (0, 3) else 2) * (2 - fmo)
cropped = (w - cux*(crop[0]+crop[1]), h - cuy*(crop[2]+crop[3]))
```

## Time Trial and Mastery

The bundle approach, not the MP4, was the work here. We reused the lane's bundle extractor
(`extract_facts.py`, with its ELF member made optional since Day 18 has no ELF) and adapted the
previous mapper: drop the ELF field, use this day's synthesis ID, and take each field's format from
conventions that earlier days had already validated:

- PE machine as 4-digit padded lowercase hex (`0x....`), entry-point RVA as 8-digit padded hex.
- MSI `ProductCode` as stored in the Property table (braces, upper case), even though the prompt
  advertised a brace-free GUID.
- OOXML external relationship targets in ascending lexical order, all external entries included.
- Video dimensions as `WxH` from `tkhd`.
- Synthesis: parse the `Files:` list out of the question text itself, hash each member in full,
  sort ascending by hex digest, emit compact JSON.

The Time Trial went in as one 23-field submission, CORRECT on call 1. The Mastery mapper walks
`evidence-NN.zip` per set, resolves the note/copy/roster questions by regex over the answer ID
(the asked attribute rotates per set: filename, size, MD5, SHA-1 or SHA-256), and again took the
synthesis `Files:` list from each question. All 115 fields went in one call, CORRECT.

Both modes were later re-run as ranked (timed, Hall-of-Fame-eligible) attempts with the same
solvers: Time Trial in a 50 s window (23 answers, 1 call, 1.43 s total) and Mastery in a 170 s
window (115 answers, 1 call, 2.74 s total). Both CORRECT. Ranked times were bragging rights, not
the scoring basis.

## Dead ends and gotchas

- **The prompt oversells the difficulty.** "Crops, originals, and the recording that could not have
  come from the claimed device" implies a disagreement between `tkhd`, `stsd` and the SPS, or a
  rotation matrix. On this specimen all three agreed, so the check was cheap but still worth doing:
  H.264 coded height (192) differs from the displayed height (180), and answering with the coded
  value would have been wrong.
- **Dimension format is not stated.** The format text for this field was the generic
  "exact artifact-derived value only". `WxH` was chosen by analogy to the JPEG/PNG fields and was
  accepted first try; we did not test other spellings.
- **The synthesis list can vary per set.** The `Files:` list in the synthesis question is not fixed
  across sets in later days (a Day 15 Mastery set swapped one file name), so the mapper reads it
  from each question rather than hard-coding it. On Day 18 all five sets listed the same four files,
  so this did not bite here.
- **Constant fixtures.** Several members of the bundle (the DLL, MSI, LNK, DOCX, EML, WAV, MP4) were
  byte-identical across the Day 16-20 sets, so some answers repeated. We still recomputed from each
  set's bytes; a repeat is an observation, not a rule.
- **Tooling.** The extractor needs `pefile` and `olefile`, which an isolated `python -I` hides; run
  it without `-I`. (The MP4 script itself is stdlib-only.)

## Takeaways

- For a video "dimensions" field, read all three sources (`tkhd`, `stsd` sample entry, SPS crop) and
  report the display size; H.264 coded height is padded to a multiple of 16. If they ever disagree
  (the prompt hints at crops and rotation), look at the tkhd matrix and the question wording and
  test singleton submissions. This specimen did not exercise that case, so we do not know which
  source the validator would favour when they differ.
- In a pooled-evidence template, the per-day diff is small: one new reader and one new field. Reuse
  the validated extractor and mapper, assert that the mapper's key set equals the start response's
  `answer_ids`, and take per-question data (like the synthesis `Files:` list) from the question text.
- A stdlib reader beats an installed one for speed: the MP4 box walk plus SPS bit-reader is under 100
  lines (SPS `ue`/`se` Exp-Golomb readers, scaling-list skip). The scaling-list sizes in that first
  version were approximate and only a profile-100 stream without scaling matrices was exercised; the
  later toolkit version walks the lists correctly and uses the right crop unit for 4:2:2.

## Evidence

Leaf base: `challenges/day18/metadata-forensics/metadata-forensics-day-18-frame-of-reference/`

- `single-solve/journal.md`, `single-solve/start_response.json`, `single-solve/artifacts/mp4_geometry.py`,
  `single-solve/artifacts/candidate.json` (no `result.json` was saved; CORRECT on attempt 1 is
  recorded in the journal and the `progress.md` row)
- `time-trial/journal.md`, `time-trial/start_response.json`, `time-trial/artifacts/extract_facts.py`,
  `time-trial/artifacts/answer_map.py`, `time-trial/ranked-1/result.json`
- `mastery/journal.md`, `mastery/start_response.json`, `mastery/artifacts/answer_map.py`,
  `mastery/ranked-1/result.json`
- Library: MP4 stream geometry (META-MP4-GEOMETRY-001), evidence-bundle toolkit
  (META-BUNDLE-TOOLKIT-B101), metadata lane answer conventions and gotchas for Days 11-20
- `progress.md` (grep `^| 18 .*frame-of-reference`)
