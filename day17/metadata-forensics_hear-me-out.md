# Hear Me Out

| | |
|---|---|
| Day / Lane | 17 / Metadata Forensics |
| Challenge ID | `metadata-forensics-day-17-hear-me-out` |
| Technique | RIFF/WAV `fmt ` chunk to ffprobe-style codec name, plus a standard evidence-bundle field sweep |
| Single Solve | Solved (journal-recorded; the saved `candidate-answers.json` is the submitted value, no `result.json` was kept) |
| Time Trial | Independent (untimed original: journal-recorded; a later ranked rerun has `ranked-1/result.json` CORRECT) |
| Mastery | Independent (untimed original: journal-recorded; later ranked reruns have `ranked-1/2/3/result.json` CORRECT) |

## Summary

The prompt talks about voice notes that were "transcoded, renamed, and embedded in mail" and about
connecting equivalent recordings and finding the stream that breaks a sender's story. None of that
turned out to be a puzzle. The only audio question in the day asks for one field of one file,
`media.primary_audio_codec_name` of `audio-001.wav`, and the answer is the ffprobe-style name of the
encoding read from the RIFF `fmt ` chunk: `pcm_s16le`. Time Trial and Mastery wrap that same field in
a 22-field evidence bundle (one copy for Time Trial, five nested copies for Mastery) that the
existing bundle extractor already handled. All three modes were CORRECT on the first submission.

## The challenge

Single Solve delivered a 1,323-byte ZIP containing only `audio-001.wav` and asked for one value,
`exact_text`. The answer ID was `d17-audio-001-media-primary-audio-codec-name`.

Time Trial delivered a 14-member evidence ZIP (about 608 KB) and 22 answer fields: a note file
(`evidence-note.txt`), its renamed copy, a `roster.csv`, a JPEG, a PNG, a PDF, a nested ZIP, a PE
executable, a .NET DLL, an MSI, a LNK, a DOCX, an EML and the WAV. Mastery delivered a ~3 MB bundle of
five nested `evidence-NN.zip` files and 110 answer fields (5 sets x 22), with IDs prefixed `setN.`.

The 22 fields were:

- note, copy and roster attributes (filename, size, MD5, SHA-1, SHA-256), the note's first 16 bytes as
  hex and its UTF-8 BOM flag;
- one field per media or document member (JPEG SOF dimensions, PNG IHDR dimensions, PDF header
  version, ZIP EOCD entry count and central-directory names, PE machine / entry-point RVA / version
  `OriginalFilename`, .NET MVID, MSI ProductCode, LNK target path, DOCX external relationship targets,
  EML Message-ID, WAV codec);
- one synthesis question: SHA-256 each of four named files and return their names in ascending digest
  order (`evidence-note.txt`, `panel-001.png`, `managed-001.dll`, `audio-001.wav` in the Time Trial).

## Solution

1. **Recon.** The Single Solve ZIP held a single WAV. We checked the bundle's SHA-256 against the
   start response before parsing it.
2. **Read the fmt chunk, not the extension.** The RIFF chunks were walked from offset 12, honoring the
   even-length pad byte, until `fmt ` was found. It held format tag 1 (PCM), 1 channel, 16000 Hz,
   16 bits per sample.
3. **Map to the ffprobe name.** The expected answer convention is ffprobe's codec name, not the
   container (`wav`) and not the numeric format tag:

   ```python
   names = {(1, 8): "pcm_u8", (1, 16): "pcm_s16le", (1, 24): "pcm_s24le",
            (1, 32): "pcm_s32le", (3, 32): "pcm_f32le",
            (6, 8): "pcm_alaw", (7, 8): "pcm_mulaw"}
   codec = names[(tag, bits)]      # (1, 16) -> "pcm_s16le"
   ```

4. **Submit.** `pcm_s16le` was accepted on the first call (flag `021756850aceba345d175a0a1ba77edd`).

## Time Trial and Mastery

Both modes reused the library's mixed-evidence-bundle extractor. Two changes were needed:

- **Optional members.** The extractor was written for earlier days whose bundles contained a
  `video-001.mp4` and a `collector-001.elf`. Day 17's bundle has neither, so those two readers were made
  optional (read only if the member is present).
- **Question-ID-driven note fields.** Instead of hard-coding which note/copy/roster attribute is asked,
  the mapper parses the ID with `(evidence-note-copy|evidence-note|roster)-(filename|size-bytes|md5|sha1|sha256)`
  and looks the value up in per-member hashes computed from the real bytes. This matters because the
  asked attribute varies by set and by mode: in the Mastery start response, set 1 asks for the copy's
  filename and the roster's size, set 3 asks for the roster's filename, set 5 asks for the note's
  filename and size and the roster's MD5, and so on.

For Mastery each nested `evidence-NN.zip` was extracted to its own facts file and mapped with a `setN.`
prefix; the merged answer map was checked to have exactly the start response's key set. JPEG and PNG
dimensions differed per set. The other members looked identical across sets to the journal author, but a
later library note found that the PE member's SHA-256 does differ per set (about 10 embedded bytes),
which is why every set's hashes were recomputed rather than copied. The synthesis answer is just the
named files sorted by their full-file SHA-256.

Both modes went CORRECT in one submission each (22 and 110 fields). Those original attempts were
untimed (`deadline: null`). We later re-ran Time Trial once and Mastery three times as timed attempts
with a scripted solver; each was CORRECT on its first submission, with a local total of about 1.5 s
(Time Trial) and 2.6-4.6 s (Mastery). Those ranked runs are speed-board attempts, not part of the
prize scoring, and the fixtures differed from the originals (the ranked Time Trial's answer IDs include
`roster-sha256`, for instance), so the extractor's ID-driven design was exercised again.

## Dead ends and gotchas

- **The prompt is flavor text.** "Connect equivalent recordings" and "the stream that breaks the
  sender's story" suggest cross-file audio comparison or a hidden second stream. The questions never ask
  for that: there is one audio file and one codec field. Reading the question list first saves the
  time.
- **Name the encoding.** The accepted form is `pcm_s16le`: not `wav`, not `PCM`, and not format number
  `1`. This was only validator-confirmed for the 16-bit PCM case; the other names in the table are
  ffprobe's convention and were not tested here.
- **Note/roster fields vary per set.** A Time Trial-style mapper that hard-codes
  `evidence-note-md5`/`roster-sha1`/etc. breaks on Mastery, where the attribute asked changes with every
  set (see above), and where the copy's filename itself can be a question.
- **The `.wav` name stopped meaning WAV later.** Per the post-hoc library note, on Day 23 a file named
  `audio-001.wav` could be FLAC, Ogg Vorbis or M4A, and the accepted answer was always the real
  container's codec. Day 17's specimens were genuine PCM WAVs, so reading the `fmt ` chunk was enough
  here, but parsing the actual container is the safer habit.

## Takeaways

- Technique to search for: **WAV `fmt ` chunk to ffprobe codec name** (tag + bits per sample), and
  more generally map an `exact_text` codec question to the tool-style name, not the container.
- For multi-field metadata bundles, derive per-set asked attributes **from the question ID** and compute
  hashes from the actual members of each nested ZIP. Assert that the mapper's key set equals the start
  response's `answer_ids` before submitting.
- Skim the full question list before reading the prompt for hints; here the narrative was misleading and
  the real task was small.

## Evidence

Leaf base path: `challenges/day17/metadata-forensics/metadata-forensics-day-17-hear-me-out/`

- `single-solve/journal.md`, `single-solve/start_response.json`, `single-solve/artifacts/answer_map.py`,
  `single-solve/artifacts/candidate-answers.json`
- `time-trial/journal.md`, `time-trial/start_response.json`, `time-trial/artifacts/extract_facts.py`,
  `time-trial/artifacts/answer_map.py`, `time-trial/ranked-1/result.json`,
  `time-trial/ranked-1/accepted_plan.json`
- `mastery/journal.md`, `mastery/start_response.json`, `mastery/artifacts/answer_map.py`,
  `mastery/ranked-1/result.json`, `mastery/ranked-2/result.json`, `mastery/ranked-3/result.json`
- `library/metadata-forensics/mixed-evidence-bundle/day17-hear-me-out-notes.md` (META-BUNDLE-D17-001),
  `library/metadata-forensics/wav-audio-codec.md` (META-WAV-CODEC-B107)
- `progress.md` (grep `^| 17 .*hear-me-out`)
- Per-challenge ranked solver: `tools/ranked/solvers/metadata-forensics-day-17-hear-me-out/solve.py`
  (not read in detail for this page)
