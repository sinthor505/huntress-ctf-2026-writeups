# Looks Can Be Deceiving

| | |
|---|---|
| Day / Lane | 2 / Metadata Forensics |
| Challenge ID | `metadata-forensics-day-02-looks-can-be-deceiving` |
| Technique | File-type identification by magic bytes (first 16 bytes as hex), nested ZIP bundles |
| Single Solve | Solved, graded result not preserved (journal records a CORRECT on 1 submission, 2026-10-20) |
| Time Trial | Independent (ranked rerun 2026-10-26 CORRECT, saved `result.json`; the 2026-10-20 live solve is journal-only) |
| Mastery | Independent (ranked rerun 2026-10-26 CORRECT, saved `result.json`; the 2026-10-20 live solve is journal-only) |

## Summary

An easy warm-up for the metadata lane. Each question asked for the first sixteen bytes of one named
file inside a ZIP, as lowercase hex. The only work was reading the right member and the right
bytes; the "disguise" was a PDF stored under a `.bin` name. All three modes were solved with a
few-line offline script and one submission each.

## The challenge

The prompt: "The extensions claim one story; the first sixteen bytes tell another. Identify what each
artifact actually is and isolate the deliberate disguise." The delivered artifact was a ZIP bundle
(`metadata-day-02-evidence.zip` in Single Solve). Each question carried an `artifact` field of the
form `bundle.zip -> member` naming the file to read. The answer format was exactly 32 hex
characters, no `0x` prefix, no spaces, with `trim` normalization.

| Mode | Questions in the first (live) attempt | Member asked about |
|---|---|---|
| Single Solve | 1 | `evidence-note.txt` |
| Time Trial | 1 | `roster.csv` |
| Mastery | 2 (one per inner ZIP, `set1.` / `set2.` prefixed IDs) | `evidence-note-copy.txt`, `disguised-report.bin` |

## Solution

1. Download the bundle through the Game Master and check its SHA-256 against the start response. The
   copy first saved locally was damaged (see gotchas); the re-download matched.
2. Read the question's `artifact` field to find the member. It is not always the same file from
   mode to mode.
3. Take `member[:16].hex()`. The whole extractor for Single Solve was:

   ```python
   z = zipfile.ZipFile(sys.argv[1])
   d = z.read("evidence-note.txt")
   print(json.dumps({"evidence-note-magic-prefix-hex-16": d[:16].hex()}))
   ```

4. Submit. Single Solve answer: `48756e7472657373204d657461646174`, which is ASCII `Huntress Metadat`,
   the start of a plain text note, so the `.txt` extension was honest there.

## Time Trial and Mastery

- **Time Trial** (live attempt): the member was `roster.csv`; the answer was
  `616e616c7973745f69642c636173655f` (`analyst_id,case_`, a CSV header). The mapper became generic: it
  reads the member name from each question's `artifact` field instead of hard-coding it. The server
  set a short deadline (about 50 s) even though we asked for an untimed attempt, so download, map and
  submit ran as one script.
- **Mastery** (live attempt): an outer ZIP holding two inner ZIPs, one question each. The mapper
  opens the inner ZIP named left of `->` and reads the member on the right. Set 2 is where the
  title pays off: `disguised-report.bin` starts `255044462d312e370a2548756e7472`, i.e. `%PDF-1.7\n%Hunter`,
  a PDF behind a `.bin` extension. The window was about 80 s; one submission covered both IDs.
- **Ranked reruns (2026-10-26):** the fixtures were much larger than the ones saved from the live
  attempts: Time Trial asked 6 questions (filename, size in bytes, MD5, SHA-1, SHA-256, and the
  16-byte prefix of `evidence-note.txt`) and Mastery asked 12 (the same six for each of two sets).
  Both went CORRECT in one call each, with total pass time around 1.7 s. We did not keep the
  solver's source in this leaf, so how it computed the extra fields is not documented here; the
  fields are standard digests of the named member.

## Dead ends and gotchas

- The locally saved `bundle.zip` for Single Solve was corrupt (bad ZIP header); its hash differed
  from the API's. A re-download matched the published SHA-256. A later review found the cause: the
  saved copy had been through LF to CRLF translation, and undoing the CRs restored the server hash
  exactly.
- Time Trial and Mastery use different members than Single Solve, and the member can change between
  attempts of the same mode, so the member must come from the question, not from memory.
- Time Trial and Mastery use `<id>/time_trial` and `<id>/mastery` as the challenge ID for start and
  submit; the Time Trial deadline was server-set and short despite requesting no timer.
- Hex normalization is trim-only: lowercase, 32 characters, no separators.

## Takeaways

- For "magic bytes" questions, trust the bytes and ignore the extension; report the first N bytes
  literally (`data[:16].hex()`) rather than a decoded file type. Check the decoded ASCII afterwards
  as a sanity check (`%PDF`, `PK`, plain text).
- Drive the extractor from the question's `artifact` field and support nested ZIPs, so the same
  script works across Single Solve, Time Trial and Mastery.
- Verify the downloaded archive's SHA-256 before reading it; line-ending damage looks like a
  corrupt ZIP.

## Evidence

Leaf base path: `challenges/day02/metadata-forensics/metadata-forensics-day-02-looks-can-be-deceiving/`

- `single-solve/journal.md`, `single-solve/challenge.json`, `single-solve/artifacts/answer_map.py`, `single-solve/artifacts/answers.json`
- `time-trial/journal.md`, `time-trial/start_response.json`, `time-trial/artifacts/answer_map.py`, `time-trial/ranked-1/result.json`, `time-trial/ranked-1/accepted_plan.json`
- `mastery/journal.md`, `mastery/start_response.json`, `mastery/artifacts/answer_map.py`, `mastery/ranked-1/result.json`, `mastery/ranked-1/accepted_plan.json`
- Provenance note: `progress.md` marks the original solves as 2026-10-18 "Player-directed local mark"; the
  2026-10-20 audit found no preserved graded result and the modes were re-solved that day. The
  saved per-mode answer files and the ranked `result.json` files are the local evidence; the
  Single Solve, Time Trial and Mastery live-attempt CORRECT results are recorded in the journals and
  `progress.md`, while only the ranked reruns have a preserved `result.json`.
