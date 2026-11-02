# Event Horizon

| | |
|---|---|
| Day / Lane | 24 / Metadata Forensics |
| Challenge ID | `metadata-forensics-day-24-event-horizon` |
| Technique | EVTX structure parsing (ElfFile / ElfChnk record walk), manifest-driven evidence-bundle extraction |
| Single Solve | Independent |
| Time Trial | Independent |
| Mastery | Independent |

## Summary

Day 24's metadata challenge introduced one new field, `evtx.record_count`, the number of records in a
Windows event log. The prompt hinted at traps (duplicated, truncated records), but on every specimen we
saw, a plain walk of the EVTX chunk structure agreed with the header's own counter, and that value was
accepted. The real work was in Time Trial and Mastery, which re-asked 20 older metadata fields beside it
and needed member lookup to cope with a new file-naming pattern. All three modes went CORRECT, one
submission each.

## The challenge

Prompt: "Windows event logs from several hosts have been duplicated, truncated, and bundled with
reports. Count records accurately and find the selectively shortened timeline."

- **Single Solve:** one ZIP (`metadata-day-24-evidence.zip`) and one question, `Determine
  evtx.record_count for events-001.evtx`, answer kind `integer` (base-10, no units or separators), `trim`
  normalization.
- **Time Trial:** one `evidence.zip` and 29 questions. One was the EVTX count; the rest were recap fields
  from earlier days' formats (JPEG SOF dimensions, PNG IHDR, PDF header version, ZIP central directory,
  PE entry-point RVA / machine / original filename, .NET MVID, MSI ProductCode, LNK target path, OOXML
  external targets, email Message-ID, audio and video codec facts, ELF e_machine, Mach-O CPU types, APK
  package name, JAR main class, SQLite page size) plus `roster-*` and `evidence-note-*` hash/size/filename
  questions and a `metadata-day-24-synthesis-sha256-order` field.
- **Mastery:** one bundle holding eight sets, each with its own evidence ZIP (`evidence-01.zip` ...),
  232 questions in total: the same 29-field family per set, with the `roster-*` and `evidence-note-*`
  questions varying in which ones appeared per set.

Neither the "selectively shortened timeline" nor any question about event content was asked in any mode;
only the record count was.

## Solution

1. **Recon.** The Single Solve ZIP was checked against the start response's SHA-256 and `events-001.evtx`
   extracted. The existing evidence-bundle toolkit already had an `evtx` reader, but it exposed no record
   count, so the field had no rule.
2. **Reading the format.** An EVTX file is a 4 KiB `ElfFile` header followed by 64 KiB `ElfChnk` chunks,
   each holding `**\x00\x00` records. We defined the answer as the number of records actually walked: for
   each chunk, read records from chunk offset 512, stepping by each record's size field. Four independent
   readings were computed as cross-checks:
   - `next_record_id - 1` from the file header (u64 at offset 24);
   - the walk stopped at the chunk's free-space offset (u32 at chunk+48);
   - the sum of `last - first + 1` of the chunk headers' event record numbers;
   - the count of distinct record IDs (a duplicated record would make this smaller).
3. **Result.** On the Single Solve specimen all five readings agreed (3 records), so there was no ambiguity
   to resolve and only one answer was submitted: `3`, accepted on the first call.
4. **Wiring.** The reader (`read_evtx` with the counts, plus a `question_facts` handler for the new field)
   went into the shared metadata toolkit, and the solver for the day is a thin wrapper around that tool's
   contract-mode entry point. The handler reports status `review` and emits the other readings as variants
   only if the cross-checks disagree; the validator never required that path.

## Time Trial and Mastery

- **Time Trial** (29 fields). The first run resolved 28 of 29: `roster-filename` failed because the roster
  member was named `roster-<hex>.csv` rather than a bare `roster` stem, so an exact-stem lookup missed it.
  Adding a hex-tagged unique-stem match to `member_by_stem` fixed it; the next run resolved 29 of 29 and went
  CORRECT in one call. The failed run was not submitted.
- **Mastery** (8 sets x 29 = 232 fields, one call). Each set has its own copy of the file names
  (`events-001.evtx`, `roster-<hex>.csv`, and so on), and which `evidence-note-*` / `roster-*` questions
  appear varies per set. The extraction therefore had to be driven by each set's manifest, resolving every
  target file from its question ID inside that set's own container, never by base name. It was accepted on
  the first submission. EVTX record counts ranged from 3 to 105 across all modes' specimens.
- All 19 EVTX members we saw (1 Single Solve, 1 + 1 Time Trial, 8 + 8 Mastery, regular and ranked) were
  69,632 bytes: a 4,096-byte header plus one 64 KiB chunk, version 3.2, not dirty, not full, with contiguous
  record IDs. We never saw a multi-chunk or dirty file, so the cross-checks stayed theoretical on live data.

## Dead ends and gotchas

- **The prompt promised traps that never appeared.** "Duplicated, truncated" suggested the header count might
  lie or records might repeat. In all 19 real specimens the walked count, header counter, free-space-bounded
  count, chunk span and unique-ID count were identical. Which reading would win if they disagreed was
  never tested against the validator; the synthetic two-chunk test we built (walked 7, header 6) only shows
  that our code flags it.
- **Member lookup is per container, not per name.** In Mastery every set calls its log `events-001.evtx`. An
  alternate solver that re-read the member by base name returned set 1's file for sets 2-8 and disagreed
  with the accepted answers on 7 of 232 fields (it was never submitted). The solver that called the library
  reader per set matched the accepted answers 232/232 and 29/29.
- **File names change from this day on.** `roster-<hex>.csv` (hex-tagged) replaced the bare roster name used
  earlier, and a stem-equality lookup silently fails on it.
- Two harness-level slips (a first-round solver that did not handle the Time Trial roster name before the fix
  above) belong to the day overview, not this challenge.

## Takeaways

- **EVTX record counting:** walk the `ElfChnk` records yourself and compare with the header's
  `next_record_id - 1`; do not trust a single counter, because duplicated or stale records behind a chunk's
  free-space offset are exactly where they diverge. On a clean single-chunk file everything agrees.
- **Bundle questions are keyed by ID, files by manifest.** For multi-set bundles, resolve each target from the
  question's own artifact and ID within that set. Diff two independent extractors' output per set before
  trusting either.
- Hex-tagged member names (`roster-<hex>.csv`) need a stem *match*, not equality.
- What we would do faster: the field was a single new reader plus a regression against the previous days'
  accepted maps; the only real delay was the roster-name miss, which a name-pattern test on the saved Time
  Trial fixture would have caught in advance.

## Evidence

Leaf base: `challenges/day24/metadata-forensics/metadata-forensics-day-24-event-horizon/`

- `single-solve/journal.md` (includes the library-promotion notes for all three modes)
- `single-solve/start_response.json`, `single-solve/accepted_plan.json`
- `single-solve/artifacts/cand-adapter/solve.py` (thin wrapper over the toolkit's contract mode)
- `time-trial/regular/result.json`, `time-trial/regular/accepted_plan.json`
- `mastery/regular/result.json`, `mastery/regular/accepted_plan.json`
- `time-trial/ranked-1/result.json`, `mastery/ranked-1/result.json` (later ranked runs; bragging rights only)
- `challenges/day24/_run/lane-metadata-forensics-day-24-event-horizon.log`
- `library/metadata-forensics/evtx-record-count/README.md` (META-EVTX-RECORD-COUNT-001)
- `library/metadata-forensics/evidence-bundle-toolkit/README.md` (META-BUNDLE-TOOLKIT-B101)
- `progress.md` (grep `^| 24 .*event-horizon`)
