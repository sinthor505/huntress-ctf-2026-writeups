# Crystal Clear

| | |
|---|---|
| Day / Lane | 13 / Binary Reverse Engineering |
| Challenge ID | `binary-reverse-engineering-day-13-crystal-dns-configuration` |
| Technique | static ELF/PE/JAR/CLR parsing; XOR+CRC32 "HCFG" config record hidden in a Base64 run inside a Crystal ELF |
| Single Solve | Independent (journal plus saved CORRECT response; 2 submission calls, the first rejected for wire format) |
| Time Trial | Independent (saved CORRECT response, 1 call; two later ranked reruns also CORRECT) |
| Mastery | Independent (saved CORRECT response on call 2; two later ranked reruns CORRECT, 1 call each) |

## Summary

Despite the title, this is not a Crystal reversing puzzle. The Day 13 delivery is a cumulative
recap: one archive holding the specimens from every earlier binary day (d01-d13), with 10
questions per day, 130 in all. The new block is d13, a Crystal-built ELF whose configuration record
is not stored as raw bytes but inside a long Base64 string, so a plain scan for the `HCFG` magic
finds nothing valid. We answered everything offline with static parsers; all three modes went
CORRECT. The only misses were a wire-format error (Single Solve) and a container label on one
Mastery set.

## The challenge

Prompt: "Crystal compiles to native code with a distinctive shine. Fingerprint the runtime, then
follow the DNS labels - this one has somewhere it wants to resolve." The "DNS labels" are the
config's `domain` field (`node-000.day13.example.invalid`, a name that can never resolve).

Delivered: an AES-encrypted `evidence.zip` (password `infected`, the one given in the guidance) with 14 members in
central-directory order: d01 through d13 plus d03's `.runtimeconfig.json`. d01, d02, d04, d05 and
d08-d13 are ELF, d03 is a managed PE32 DLL, d06 and d07 are JARs. The d08 member alone is 67 MB and
d13 is 749,208 bytes. Question IDs are prefixed by day (`d13_*`), and the 130 answers split
into 49 `exact_text`, 60 `pattern`, 9 `integer`, 7 `enum` and 5 `json`.

The ten d13 questions are generic recap fields: SHA-256 of the member, `architecture`, `machine`,
`format` (an enum: `elf`, `pe`, `pe_cli`, `jar`, `wasm`, `neko_bytecode`), the HCFG `campaign`,
`domain`, `mutex`, `url` and `crc32_hex`, and `hcfg_offset`. None of them ask for anything
Crystal-specific.

## Solution

1. **Verify and extract.** The archive hash matched the start response; an AES-capable ZIP reader
   (stdlib `zipfile` cannot do WinZip AE-2) wrote each member and its SHA-256 in central-directory
   order. Nothing was executed.
2. **Structural fields.** d13 is ELF64 little-endian, machine x86-64; `format` is `elf`. The other
   days' fields (ELF section names and `.symtab` names in table order, d01's `recover_code` stub
   `mov eax,0x1fa4; ret`, d03's CLI machine `0x14c` and metadata version `v4.0.30319`, the two JARs'
   compression methods `[0,8]`) came from the same kind of header parsing.
3. **Find the config record.** The question defines the record layout exactly: `HCFG` magic, a
   version byte, a one-byte XOR key, a little-endian length, a little-endian CRC32 of the *decoded*
   payload, then the XOR-encoded JSON. For d13 it also says how to find it: take every
   non-overlapping Latin-1 run matching `[A-Za-z0-9+/]{100,}={0,2}`, Base64-decode it, and accept the
   first decoded string that begins with a CRC32-valid record.

   ```python
   pattern = re.compile(rb"[A-Za-z0-9+/]{100,}={0,2}")
   for match in pattern.finditer(data):
       try:
           decoded = base64.b64decode(match.group(), validate=True)
       except (ValueError, binascii.Error):
           continue
       for item in literal_hcfg_candidates(decoded):   # HCFG magic + XOR + CRC32 + JSON check
           item["source_offset"] = match.start()
           found.append(item)
   ```

   The validity check (bounds, XOR, CRC32, JSON parse) is what separates a real record from a stray
   `HCFG` byte sequence.
4. **Read the fields.** The record was at the very start of the Base64 run, which began at byte
   offset 560188 of the d13 member. Decoded, it gave campaign `BR13-000`, domain
   `node-000.day13.example.invalid`, mutex `Global\HuntressBR13_000`, URL
   `https://node-000.day13.example.invalid/api/BR13-000`, and header CRC32 `0xa69b7e61`.
5. **Submit.** The mapper keyed its output by the exact `answer_ids` from the start response, with
   every value serialized as a string. The 130-field map was accepted on the second call (the first was a wire-format
   rejection, see the gotchas). The Single Solve returned the flag `b97709a9710c17f3a5e6cd1efb3c14a1`.

The journal also records that the library's independent HCFG and JAR extractors were run over
d02-d13; every record passed CRC32 and they matched the mapper's fields.

## Time Trial and Mastery

- **Time Trial** was a fresh `variant-000` archive with the same layout and an 80 s window. The
  same mapper, run on the new bytes, was accepted on the first submission with the string-serialized
  payload (130 fields).
- **Mastery** delivered three fresh archives under one 200 s clock (390 `setN.`-prefixed answers).
  Each set had to be mapped from its own bytes: campaigns differed per set (`BR13-007`, `BR13-002`,
  `BR13-005` for sets 1-3), although the d13 Base64 run was again at offset 560188 in each.
  The first complete submission came back `incorrect`; the second was CORRECT (see gotchas).
- Later ranked reruns (bragging rights, not the prize ranking) used a general solver built on the
  library's question-facts engine; Time Trial was CORRECT in one call (80 s window, about 10-11 s
  end to end) and Mastery in one call with five sets and 650 answers under a 320 s window. The live
  ranked fixtures were larger than the first Mastery attempt's, so the solver had to be per-set
  and manifest-driven rather than tied to three archives.

## Dead ends and gotchas

- **Wire format.** The Single Solve's first call returned HTTP 400 `invalid_answers`: the map held
  native JSON arrays and integers. Every value must be a JSON string (arrays as compact JSON text,
  integers as base-10 text). The evidence values were unchanged; nothing was graded. (This is also in
  the day overview.)
- **Mixed formats in Mastery.** In set 2 the d02, d04 and d10 specimens were PE32+ executables,
  while sets 1 and 3 had ELF64 for the same members. The first 390-field submission was
  rejected as a batch with no per-field feedback; the journal records only that the d10 container and
  architecture labels were corrected, to `PE32+` and the COFF Machine as `x86-64`. The exact label
  that failed is not recorded. Classify each member from its magic bytes, per set.
- **Offset means the Base64 source offset.** For Base64 carriers `hcfg_offset` is the offset of
  the first Base64 character of the winning run in the member (560188), not the offset of `HCFG`
  inside the decoded bytes (0). The question text says this; it is easy to answer with the decoded
  position.
- **d13's record is only reachable through Base64.** The mapper takes d13's candidates from the
  Base64-run scan alone, and the same carrier shape appeared in d08, d09 and d11. We did not record
  whether d13 also held a literal CRC-invalid `HCFG` decoy.
- **Spelling is soft for patterns, exact for enums.** `x86-64` was accepted for every ELF and PE
  machine/architecture field, and `format` for d12/d13 is the lowercase enum (`elf`).
- **A Crystal-specific field came later.** A symbol-count field for the Crystal member (`Crystal`,
  `crystal`, `__crystal`, `GC_` matches) shows up in later days' recap blocks, not this one; there the
  `.symtab`-only count was accepted and a `.symtab`+`.dynsym` count was rejected (progress row for
  Day 29 Fraternal Twins).

## Takeaways

Technique names: **cumulative recap bundle** and **HCFG Base64 run carrier**. When a binary-lane
delivery lists a member per earlier day, treat it as a regression test of every earlier parser and
keep one manifest-driven mapper rather than per-day scripts. The config record is validated by its
own structure (magic, XOR key, length, CRC32 of the decoded bytes), so scan for the format, not for a
string: if a literal scan finds nothing, try long Base64 runs. Before the first submission, dry-run
the key set against `answer_ids`, serialize everything as strings, and check any mixed-format
member (ELF/PE swaps across Mastery sets) from magic bytes.

## Evidence

Leaf base path: `challenges/day13/binary-reverse-engineering/binary-reverse-engineering-day-13-crystal-dns-configuration/`

- `single-solve/journal.md`, `single-solve/start_response.json`, `single-solve/artifacts/answer_map.py`
  (the mapper; `base64_run_candidates` and `valid_hcfg_at`), `single-solve/artifacts/hcfg-crosscheck.json`,
  `single-solve/artifacts/attempt-001-response.json` (rejected) and `attempt-002-response.json` (CORRECT),
  `single-solve/artifacts/submission-attempt-log.json`
- `time-trial/journal.md`, `time-trial/artifacts/attempt-001-response.json`,
  `time-trial/ranked-1/result.json`, `time-trial/ranked-2/result.json`
- `mastery/journal.md`, `mastery/artifacts/attempt-001-response.json` (incorrect),
  `mastery/artifacts/attempt-002-response.json` (CORRECT), `mastery/artifacts/candidate-answers.json`,
  `mastery/ranked-1/result.json`, `mastery/ranked-2/result.json`
- `tools/ranked/solvers/binary-reverse-engineering-day-13-crystal-dns-configuration/solve.py` (ranked-rerun solver)
- Library entries: BRE-HCFG-001, BRE-HCFG-BASE64-001, BRE-JAR-HCFG-001, BRE-PE-CLR-001,
  BRE-QFACTS-B107, SHARED-AESZIP-001 (`library/binary-reverse-engineering/cumulative-recap-bundles.md`)
- `progress.md` (grep `^| 13 .*crystal-dns`)
