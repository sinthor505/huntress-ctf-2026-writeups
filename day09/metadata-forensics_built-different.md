# Built Different

| | |
|---|---|
| Day / Lane | 9 / Metadata Forensics |
| Challenge ID | `metadata-forensics-day-09-built-different` |
| Technique | Byte-level metadata reads from a nested evidence ZIP (hashes, JPEG SOF, PNG IHDR, PDF header, ZIP EOCD, PE COFF Machine) |
| Single Solve | Independent (`submission-01-response.json` reads CORRECT; no `result.json` was saved) |
| Time Trial | Independent (`open-1/result.json` CORRECT, plus four ranked reruns, all CORRECT) |
| Mastery | Independent (`artifacts/submission-01-response.json` CORRECT, plus `ranked-1/result.json` CORRECT) |

## Summary

The prompt promised a software drop with executables for several architectures and decoys with
executable extensions, but the 13 questions only asked for plain header and hash facts about six files in
one ZIP. Every field was readable with a few dozen lines of standard-library Python, so we read each
value from the real bytes and checked it against a second tool. All three modes went CORRECT on the first
submission.

## The challenge

Prompt: "A software drop contains executables for several architectures, plus decoys wearing executable
extensions. Determine what can actually run where."

Single Solve delivered `metadata-day-09-evidence.zip` and 13 questions. The ZIP holds six members, in
this order: `evidence-note.txt` (33 bytes), `camera-001.jpg`, `panel-001.png`, `document-001.pdf`,
`evidence-drop-001.zip` (nested) and `telemetry-agent-001.exe` (about 1.26 MB). The fields:

- For `evidence-note.txt`: filename, size, MD5, SHA-1, SHA-256, the first 16 bytes as hex, and whether a
  UTF-8 BOM is present.
- Image, PDF and PE headers: the JPEG SOF dimensions, the PNG IHDR dimensions, the PDF header version,
  and `pe.coff_machine_u16_hex` for the `.exe` (a `0x`-prefixed 4-digit lowercase hex value).
- The nested ZIP: the EOCD total-entries count, and the central-directory member names as a compact JSON
  array in stored order.

All answers are strings. The enum field takes `true` or `false`, the integers are base-10, and the member
list is JSON text. Time Trial reused the same 13 IDs. Mastery delivered five such archives in one bundle,
with answer IDs prefixed `set1.` to `set5.` (65 fields).

## Solution

1. Verified the download against the start response's SHA-256 and treated every member as data. Nothing
   was executed.
2. Read each member into memory with `zipfile`, failing if a member name was missing or duplicated:

   ```python
   def read_member(archive, name):
       matches = [i for i in archive.infolist() if i.filename == name]
       if len(matches) != 1:
           raise ValueError(f"expected exactly one member named {name!r}, found {len(matches)}")
       return archive.read(matches[0])
   ```

3. Note fields came straight from the bytes: `len`, `hashlib.md5/sha1/sha256`, `note[:16].hex()`, and
   `note.startswith(b"\xef\xbb\xbf")` for the BOM.
4. JPEG: walked the marker segments, skipping fill bytes and standalone markers, and read height then
   width from the first SOF segment (`C0`-`CF` excluding `C4`, `C8` and `CC`). PNG: checked the 8-byte
   signature and that `IHDR` is the first chunk, then read two big-endian `uint32` values. PDF: regex
   `%PDF-(\d\.\d)` over the first 1024 bytes.
5. Nested ZIP: the EOCD count came from finding `PK\x05\x06` whose declared comment length ends exactly
   at end-of-file, then reading the `uint16` at offset 10. Member names came from the central directory
   via `zipfile`, which keeps stored order.
6. PE: confirmed `MZ`, followed `e_lfanew` (offset `0x3C`) to the `PE\0\0` signature, bounds-checked, and
   read the little-endian `uint16` Machine field. The result was `0x014c` (i386), matching
   `file`'s "PE32 executable ... Intel 80386".
7. Cross-checked with independent tools: the library's JPEG SOF, PNG chunk-CRC and ZIP EOCD inspectors,
   `xxd` for the PDF header, `unzip -Z1` for member order, and `md5sum`/`sha1sum`/`sha256sum`/`wc -c` for
   the note. All agreed.
8. A small `answer_map.py` mapped those facts onto the exact answer IDs from the start response and
   printed one JSON object. All 13 fields went in one submission and came back CORRECT.

Single Solve values: JPEG `257x146`, PNG `269x153`, PDF `1.4`, nested ZIP EOCD count `2` with members
`["room-01/item-1.txt","room-02/item-1.txt"]`, PE machine `0x014c`, note size `33`, BOM `false`. The note's
first 16 bytes were `43617365203342413933333736383441`, ASCII `Case 3BA9337684A`.

## Time Trial and Mastery

- **Time Trial.** The start response reported `run_for_time: false` and no deadline, so the first attempt
  ran untimed. We copied the Single Solve extractor and mapper unchanged and the 13 values were CORRECT in
  one call. The only fields that changed were the note's digests and prefix (here `Case A778BE0A8B2...`);
  the image, PDF, nested ZIP and PE facts were identical to Single Solve. The first question's text carried
  an appended `[assigned-artifact-sha256:<hex>]` tag, so questions were classified by ID tail, not exact
  text.
- **Mastery.** The bundle was five per-set ZIPs (about 600 KB each). Running the same extractor on each set
  showed that sets 1-5 all had JPEG `257x146`, PNG `269x153`, PDF `1.4`, EOCD count 2, the same two member
  names, PE `0x014c`, note size 33 and no BOM. Only the note digests and 16-byte prefixes differed, so they
  had to be recomputed per set rather than copied. The timed window was 170 s, and the one 65-field
  submission went in about 32 s after start.
- **Ranked reruns (later audit).** A per-challenge solver in the ranked tooling re-solved the Time Trial
  four times and the Mastery once, each CORRECT in a single call. The ranked Time Trial window was 50 s
  and totals were 1.5-2.9 s (solve step 0.04-0.24 s). The ranked Mastery window was 170 s with a 2.4 s
  total. These are separate fresh attempts, each backed by its own `result.json`.

## Dead ends and gotchas

- The prompt's "several architectures" and "decoys wearing executable extensions" never appeared in the
  questions. There was exactly one `.exe`, and it was a real PE32 i386 image. The only architecture
  question was the COFF Machine value, read from the header and not inferred from the name. If a variant
  had shipped a decoy there, the `MZ`/`PE\0\0` checks would have rejected it. We saw no such variant.
- Output formats were strict. The PE field wants the `0x` prefix with four lowercase hex digits, the
  member list wants compact JSON (no spaces) in stored order, and the BOM answer is the string `false`.
  We got these right the first time, so we do not know which spellings the validator would have
  accepted.
- Mastery IDs are prefixed `setN.`, and the per-set evidence-note digests differ even though everything
  else is constant. A solver that cached set 1's note values would fail on the other four.

## Takeaways

- This is a "read the header, don't trust the name" challenge: bounds-checked `zipfile`, `struct`, and
  `hashlib` were enough. A technique list to search: ZIP EOCD scan with comment-length validation, JPEG
  SOF marker walk, PNG IHDR, PE COFF Machine via `e_lfanew`.
- Keep a second tool for each field. The cross-checks were cheap, and they are the reason the first
  submission was trusted.
- Write the extractor so each mode only changes the answer-ID mapping. Here the same script served all
  three modes with no edits, which is why the later ranked reruns finished in about two seconds.

## Evidence

Base path: `challenges/day09/metadata-forensics/metadata-forensics-day-09-built-different/`

- Single Solve: `single-solve/journal.md`, `single-solve/artifacts/inspect_built_different.py`,
  `single-solve/artifacts/answer_map.py`, `single-solve/submission-01-response.json` (CORRECT).
- Time Trial: `time-trial/journal.md`, `time-trial/open-1/result.json` (CORRECT),
  `time-trial/submission-01-response.json`, `time-trial/ranked-1/` to `ranked-4/` (`result.json`, all
  CORRECT).
- Mastery: `mastery/journal.md`, `mastery/artifacts/answer_map.py`,
  `mastery/artifacts/submission-01-response.json` (CORRECT), `mastery/ranked-1/result.json` (CORRECT).
- Progress rows: `progress.md`, rows for Day 9 `metadata-forensics-day-09-built-different`.
- Ranked solver and fixture list: `tools/ranked/solvers/metadata-forensics-day-09-built-different/`,
  `tools/ranked/cases/metadata-d06-10.json`.
