# Little Endian on the Prairie

| | |
|---|---|
| Day / Lane | 19 / Metadata Forensics |
| Challenge ID | `metadata-forensics-day-19-little-endian-on-the-prairie` |
| Technique | ELF header parsing (`e_machine` honouring `EI_DATA`), mixed-format evidence-bundle metadata extraction |
| Single Solve | Solved (journal-recorded) |
| Time Trial | Independent (`ranked-1/result.json` CORRECT; original untimed attempt journal-recorded) |
| Mastery | Independent (`ranked-1` and `ranked-2` `result.json` CORRECT; original untimed attempt journal-recorded) |

## Summary

Day 19's new field was the ELF `e_machine` value, asked as a lowercase hex string. The Single Solve
was exactly that one field from one ELF. The only trap was width: the minimal spelling `0x3e` was
rejected and the zero-padded four-digit `0x003e` was accepted on the second submission. Time Trial
(24 fields) and Mastery (120 fields, five sets) re-asked every metadata field from Days 4-19 about a
16-member evidence bundle, and one mapper built on the earlier days' validator-confirmed formats got
both accepted on the first complete submission.

## The challenge

Prompt: "Linux binaries from several systems have been pooled with Windows executables and renamed
data. Determine architecture directly from the ELF header and rebuild deployment groups."

- **Single Solve.** A 1,849-byte ZIP holding `collector-001.elf`. One question,
  `d19-elf-001-elf-e-machine-u16-hex`, answer format "lowercase hexadecimal with a 0x prefix and no
  explanatory text", `trim` normalization.
- **Time Trial.** A 628,573-byte ZIP of 16 members (text note and a renamed copy, `roster.csv`, JPEG,
  PNG, PDF, a nested ZIP, a PE32 exe, a managed .NET DLL, an MSI, an LNK, a DOCX, an EML, a WAV, an
  MP4 and the ELF64). 24 questions: note/roster filename, size and digests, plus one question per
  format family (JPEG SOF dimensions, PNG IHDR dimensions, PDF header version, ZIP EOCD entry count and
  member names, PE machine/entry RVA/original filename, .NET MVID, MSI ProductCode, LNK target path,
  DOCX external relationship targets, email Message-ID, audio codec, video dimensions, ELF
  `e_machine`) and one "synthesis" question (sort four named files by full-file SHA-256).
- **Mastery.** A 3.1 MB outer ZIP of `evidence-01.zip` .. `evidence-05.zip`, 120 questions =
  5 sets x 24, with `setN.`-prefixed IDs. Formats: 45 `exact_text`, 45 `pattern`, 15 `json`,
  10 `integer`, 5 `enum`.

## Solution

1. **Read the header, not the filename.** The prompt stresses reading architecture "directly from the
   ELF header". `e_machine` is a `u16` at offset 18, and its byte order is set by `EI_DATA` at offset 5
   (1 = little, 2 = big). The extractor picks the struct prefix from that byte first:

   ```python
   e = "<" if d[5] == 1 else ">"
   return "0x%04x" % struct.unpack_from(e + "H", d, 18)[0]
   ```

   The sample was ELF64 little-endian with `EM_X86_64` (62), so the raw value is `0x3e`.
2. **Width.** The question text names the field `e_machine_u16_hex`. We first submitted the minimal
   `0x3e` and got `incorrect`; `0x003e` (four hex digits for a u16) was `CORRECT` on attempt 2. A note
   from earlier days (PE machine and entry RVA were accepted zero-padded) had pointed that way, which
   is why the second guess was cheap. The same rule held for every ELF machine we saw later in the
   event (Day 23 reused it for eight different machines, per the journal).
3. **Bundle modes.** One offline extractor (`extract_facts.py`) opens the evidence ZIP, reads each
   member by its fixed name and produces a facts dictionary; a separate mapper turns facts into the
   per-question answers. It reused, rather than re-derived, the formats the validator had confirmed on
   earlier days:
   - dimensions as `WxH`; PE machine as 4-digit and entry RVA as 8-digit zero-padded hex; ELF the same
     4-digit form;
   - MSI ProductCode as stored (braces, upper case); .NET MVID bare lowercase (via the library's
     MVID reader); DOCX external targets in ascending lexical order; LNK local base path;
   - synthesis question: the filenames from the question's own `Files:` list sorted by full-file
     SHA-256, emitted as compact JSON.
4. **New formats, guessed once.** Three fields had no earlier validator evidence: the email
   Message-ID (kept with its angle brackets), the audio codec (`pcm_s16le`, the ffprobe-style name
   derived from WAV format tag and bit depth) and video dimensions (`320x180`, tkhd and `avc1`
   sample entry agreed). All three were accepted in the first Time Trial submission.

## Time Trial and Mastery

- **Time Trial.** 24 fields in one call: `CORRECT` on attempt 1. Both original start responses came
  back untimed (`deadline: null`). A later timed rerun (`ranked-1`) was also `CORRECT` in one call,
  1.56 s total locally with a 50 s window.
- **Mastery.** The mapper extracts each nested `evidence-NN.zip` with the same `extract_facts` code and
  fills the 120 IDs per set. Per-set questions vary (which of note, copy and roster is asked about,
  and whether filename, size or a digest), so the mapper resolves them with one generic rule,
  `(evidence-note-copy|evidence-note|roster)-(filename|size-bytes|md5|sha1|sha256)`, instead of
  hard-coding set 1. The question counts confirm the variation: `roster-sha1` appears in all five
  sets, `evidence-note-md5` in four, `roster-size-bytes` and `roster-md5` in one each. The journal
  notes the specimens were largely identical across sets (same MVID, ProductCode, PE) with different
  note, roster and dimension values. One complete 120-field submission was `CORRECT` on attempt 1; two
  timed reruns (`ranked-1`, `ranked-2`) each went `CORRECT` in one call.

## Dead ends and gotchas

- **Unpadded hex is rejected.** `0x3e` returned `incorrect` although the stated format ("lowercase
  hexadecimal with a 0x prefix") does not mention padding. The accepted form is the full width of the
  field: `0x003e`.
- **Question text names the field, not the encoding.** `u16` in the ID is the width hint.
- **Per-set variation in Mastery.** A mapper that resolves "the note" or "the roster" by a hard-coded
  member name can read the wrong file in some set; the member and the asked property change per set.

## Takeaways

- Parse ELF `e_machine` with the `EI_DATA` byte order and zero-pad to the field width (`0x%04x`).
  Searchable names: `e_machine`, `EI_DATA`, fixed-width hex answers.
- When a day's new field is a single ELF value, spend the effort on the answer's *format* by looking
  at what earlier days' validators accepted, not on the parsing.
- For mixed-evidence bundles, keep extraction (facts as JSON) and mapping (facts to answer IDs)
  separate and drive Mastery from the nested-ZIP manifest per set; one extractor then serves
  Single Solve, Time Trial and Mastery.

## Evidence

Leaf: `challenges/day19/metadata-forensics/metadata-forensics-day-19-little-endian-on-the-prairie/`

- `single-solve/journal.md`, `single-solve/artifacts/extract.py`, `single-solve/artifacts/submission-attempt-log.json`
  (attempt 1 `incorrect`, attempt 2 `CORRECT`; no graded result file was saved), `single-solve/artifacts/cand2.json`
- `time-trial/journal.md`, `time-trial/artifacts/extract_facts.py`, `time-trial/artifacts/answer_map.py`,
  `time-trial/ranked-1/result.json` (CORRECT)
- `mastery/journal.md`, `mastery/artifacts/answer_map.py`, `mastery/ranked-1/result.json` and
  `mastery/ranked-2/result.json` (both CORRECT)
- `progress.md` rows for Day 19 (all three modes recorded as solved on the original attempts)
- Library technique: mixed-evidence-bundle (META-BUNDLE-001)
