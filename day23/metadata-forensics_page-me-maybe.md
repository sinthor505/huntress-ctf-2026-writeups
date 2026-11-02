# Page Me Maybe

| | |
|---|---|
| Day / Lane | 23 / Metadata Forensics |
| Challenge ID | `metadata-forensics-day-23-page-me-maybe` |
| Technique | SQLite header parsing inside a cumulative evidence bundle; set-valued fields that want sorted order |
| Single Solve | Solved (journal-recorded; the attempt log reads CORRECT, no `result.json` saved) |
| Time Trial | Independent |
| Mastery | Independent |

## Summary

"Page Me Maybe" introduced SQLite databases to the metadata-forensics lane. The Single Solve was one
header field (`page_size_bytes`) read straight from the 100-byte SQLite header. Time Trial and
Mastery were the lane's usual cumulative recap bundle (28 fields per evidence set) with the SQLite
field folded in. What cost submissions were two fields where the validator wanted **ascending
sorted order** instead of the file's natural order, and a mislabeled audio file whose codec had to
be identified from the real bytes.

## The challenge

Prompt: "Application databases have been copied, truncated, embedded, and mislabeled. Validate page
geometry, connect databases to parent applications, and find the impossible claim." No live
target; each mode delivered a ZIP.

- **Single Solve:** a 536-byte ZIP holding `database-001.sqlite` (12,288 bytes). One question,
  `d23-sqlite-001-sqlite-page-size-bytes`, `integer`, base-10, no units.
- **Time Trial:** `evidence.zip` (654 KB), 28 questions. Nineteen named members: note, roster and
  note-copy text files, JPEG, PNG, PDF, a nested ZIP, PE exe and dll, MSI, LNK, a DOCX, EML, an
  `audio-001.wav`, an MP4, ELF, fat Mach-O, APK, JAR and the SQLite file, plus one cross-file
  SHA-256-ordering question.
- **Mastery:** `evidence-bundle.zip` (4.7 MB) with seven nested `evidence-0N.zip` sets, 196 answer
  ids (7 x 28, `setN.<suffix>`), the same 20-member manifest in every set.

The prompt's "truncated" and "impossible claim" themes never showed up as questions in any mode we
saw: every SQLite specimen we checked had a header page count times page size equal to the file size.

## Solution (Single Solve)

1. **Recon.** Verified the download against the start response's SHA-256 and the `PK` magic. The
   file magic `SQLite format 3\x00` was already recognized by the library's `identify()`, but it had
   no SQLite reader.
2. **The field.** Per the SQLite file-format spec, the page size is a big-endian u16 at offset 16,
   with the raw value `1` meaning 65536. The raw value here was `0x1000`, so 4096.
3. **Cross-check.** The header's `database_size_pages` was 3 and 3 x 4096 = 12,288, the actual file
   size, so the header was self-consistent.
4. **Submit.** A stdlib-only extractor emitted `{"d23-sqlite-001-sqlite-page-size-bytes": "4096"}`
   (a string, per the lane's string-only rule). CORRECT on attempt 1.

```python
page_size_raw = struct.unpack_from(">H", data, 16)[0]
page_size = 65536 if page_size_raw == 1 else page_size_raw
```

While there we also parsed `sqlite_master` off page 1 (two tables, `evidence` and `timeline`) and
recorded `application_id`, in case a later mode asked about "parent applications". No mode did.

## Time Trial and Mastery

**Time Trial** (28 fields). The SQLite reader was reused as-is. Everything else came from the
lane's existing readers plus two ported adapters (binary AndroidManifest and JAR manifest, both
already validator-confirmed on Days 21 and 22). Two new things had to be built:

- `audio-001.wav` was really an MP4/M4A container (`ftyp M4A `). The codec name we submitted, `aac`,
  came from the `esds` box: `ES_Descriptor`, then `DecoderConfigDescriptor`, whose
  `objectTypeIndication` `0x40` means AAC. This was accepted, so the ffprobe-style name was the
  right convention.
- The nested ZIP's `zip.central_directory_member_names` had to be sorted (see below).

**Mastery** (196 fields, 7 sets). The Time Trial extractor and mapper were reused, pointed at the
bundle. The new wrinkle was that `audio-001.wav` was a different real format per set: FLAC (sets 1, 4,
6), Ogg/Vorbis (sets 2, 3, 7) and a genuine WAV (set 5). FLAC was already handled; an Ogg reader was
added that keys on the first page's identification packet:

```python
if b"\x01vorbis" in head: return "vorbis"
if b"OpusHead"   in head: return "opus"
if b"\x80theora" in head: return "theora"
```

Observed SQLite page sizes across the sets ran from 512 to 65536. The per-set roster/note-copy hash
fields varied by set, so the extractor resolved the target file from the question id rather than
naming a member.

## Dead ends and gotchas

- **Natural order was wrong for two set-valued fields (the day's real trap).** Time Trial's first
  full submission came back `incorrect`, and so did two reduced batches. A plain `incorrect` gives no
  per-field signal, but a correct subset comes back `accepted` and persists, so bisection worked.
  Submitting singletons showed that `zip.central_directory_member_names` in central-directory
  order, the convention accepted on Days 8-22, was rejected, while the same five names sorted
  ascending were accepted. The 15 remaining fields then went through together as CORRECT (13
  submissions in total).
- **Same thing for `macho.cpu_type_set` in Mastery.** The first 196-field submission was
  `incorrect`. Submitting each set separately accepted sets 1, 6 and 7 and rejected sets 2-5. Bisecting
  set 2 down to a singleton showed that fat-header table order was rejected and the ascending
  lexical sort of the cpu-type hex strings was accepted. Applying the sort to all sets then went
  CORRECT (16 submissions in total). Sets 1, 6 and 7 passed under table order only because their
  table order happened to equal sorted order. A later census in the journal found that all earlier
  two-member ZIPs and Mach-O sets also had natural order equal to sorted order, so Day 23 simply
  contained the first specimens where the two orders differ. That reading is the journal's, not
  something the validator stated.
- **Per-set isolation beats flat bisection on Mastery.** Seven set-level calls ruled out three sets
  at once and left a small shared cause.
- **The mislabeled audio file hides the codec.** The `.wav` extension said nothing reliable. Trust
  magic bytes (`ftyp`, `OggS`, `fLaC`, `RIFF`), then read the codec from the container's own
  descriptor, not from the sample-entry fourcc.
- The ranked rerun of 2026-10-31 hit both ordering traps again on fresh fixtures: a solver that
  predated the sorted-order finding failed on `zip.central_directory_member_names`, then on
  `macho.cpu_type_set`, before a third try went CORRECT in one call. That mapper had not yet applied
  the sorted rule from the library notes (see the Day 23 overview).

## Takeaways

- **SQLite header recipe:** magic at 0-15, page size u16 BE at 16 (`1` means 65536), page count u32 BE
  at 28, schema cookie at 40, user_version at 60, `application_id` at 68. Compare page count x page
  size to the file size to detect a truncation or "impossible" size claim.
- **Try sorted order first for unordered set fields.** When a field is a JSON array of identifiers
  with no inherent order (ZIP member names, Mach-O cpu types, OOXML targets), sort ascending. The
  Where natural order is already sorted, sorting changes nothing, so it is a safe default (our
  inference from the two Day 23 findings, not a validator statement).
- **Isolating a bad field:** a plain `incorrect` is batch-level only, but a correct subset persists
  as `accepted`. Bisect by set first, then by halves, down to singletons.
- **Media codec names:** use the container's decoder config (`esds` object type, Ogg identification
  header, FLAC streaminfo) and the ffprobe-style short name (`aac`, `vorbis`, `flac`).

## Evidence

Leaf: `challenges/day23/metadata-forensics/metadata-forensics-day-23-page-me-maybe/`

- `single-solve/journal.md`, `single-solve/start_response.json`,
  `single-solve/artifacts/extract_facts.py`, `single-solve/artifacts/submission-attempt-log.json`
- `time-trial/journal.md`, `time-trial/artifacts/accepted_map.json`,
  `time-trial/artifacts/submission-attempt-log.json`, `time-trial/ranked-2/result.json`
- `mastery/journal.md`, `mastery/artifacts/accepted_map.json`,
  `mastery/artifacts/extract_facts.py` (`ogg_codec_name`), `mastery/ranked-1/accepted_plan.json`,
  `mastery/ranked-1/result.json`
- `progress.md` (grep `^| 23 .*page-me-maybe`); library entries META-BUNDLE-TOOLKIT-B101 and
  META-GOTCHAS-B110 (SQLite reader promoted into the toolkit afterwards).
