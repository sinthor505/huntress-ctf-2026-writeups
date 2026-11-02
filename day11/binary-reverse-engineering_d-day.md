# D-Day

| | |
|---|---|
| Day / Lane | 11 / Binary Reverse Engineering |
| Challenge ID | `binary-reverse-engineering-day-11-d-collection-planner` |
| Technique | static triage of a cumulative d01-d11 specimen bundle; HCFG config records (XOR + CRC32) found in a long Base64 run |
| Single Solve | Solved (journal-recorded) |
| Time Trial | Solved (journal-recorded) |
| Mastery | Solved (journal-recorded) |

All three modes were graded CORRECT on the Game Master according to our own journals and `progress.md`,
but no `result.json` or ranked folder was saved for this challenge. The ranked rerun of this day's
Binary Time Trial and Mastery was not possible (see "Time Trial and Mastery").

## Summary

D-Day is the Day 11 entry in the Binary lane's running series. The prompt talks about D-language
template expansions, but the 110 questions are a recap of every earlier day's specimens (d01 to d11):
hashes, header facts and the decoded `HCFG` configuration record of each file. Nothing in the
questions required executing or decompiling anything; the whole thing was static parsing, and the
only new carrier on Day 11 was an `HCFG` record hiding inside a long Base64 run in an ELF. All three
modes were solved offline with one extractor that was extended, not rewritten, between modes.

## The challenge

Prompt: "D templates generate code the way a printing press generates pages ... Trace the runtime's
template expansions back to the collection config they built." Single Solve was untimed.

Delivery: one AES-encrypted ZIP (`evidence.zip`, password `infected`) with 12 members in
central-directory order: `d01` ELF, `d02` ELF, `d03` .NET DLL plus its `runtimeconfig.json`, then one
specimen each for d04 to d11 (ELF for d04, d05, d08 to d11; JAR for d06 and d07). 110 answer ids, 10
per day-block, each a JSON string in the format the start response specified (`pattern`, `enum`,
`json`, `exact_text` or `integer` kinds; for the Single Solve 52 pattern, 5 enum, 5 json, 41
exact_text, 7 integer).

The per-block questions differ a little, but most blocks share a core: the SHA-256 of the specimen,
its container format and architecture, and `config_campaign` / `config_domain` / `config_mutex` /
`config_url` / `config_crc32_hex` from the decoded `HCFG` record. Block-specific extras:

- d01: the full ELF header and section picture (class, type, endianness, entry point, section names,
  every `.symtab` name in order, `has_symtab`) and the disassembly of the `recover_code` function.
- d02: XOR key, C++ typeinfo/vtable presence (a `DiscoveryEngine` class).
- d03: both member hashes, PE machine, CLR metadata version string.
- d06 / d07: ZIP compression methods of the JAR entries.
- d09 / d11: byte offset of the HCFG record; d10: whether a section named `.hcfg` exists.

## Solution

1. **Verify and unpack.** The archive SHA-256 matched the start response; members were read with
   `pyzipper` and never executed.
2. **Ordinary header facts.** A small ELF parser read the header, section table, name table and
   `.symtab`; GNU `readelf`/`objdump` were used as independent checks. The d03 PE/COFF header and CLR
   metadata root were parsed statically for the machine and the `v4.0.30319` version string.
3. **`HCFG` records.** The record layout is the one defined in each question: `HCFG` magic, version,
   one-byte XOR key, little-endian length, little-endian CRC32 of the decoded payload, then the
   payload. The extractor accepts a candidate only when the bounds fit, the CRC32 of the XOR-decoded
   payload matches, and the payload parses as a JSON object:

   ```python
   payload = bytes(b ^ key for b in record[12:12 + length])
   if zlib.crc32(payload) & 0xFFFFFFFF != expected_crc:
       return None
   decoded_json = json.loads(payload.decode("utf-8"))
   ```

4. **Carrier per day.** Where the record lives changes by block: the start of a section named
   `.hcfg` (d02), a raw literal marker (several blocks), a Base64-encoded string constant in the
   `Main-Class` class file's constant pool for the JARs (d06, d07; classes were parsed as data), and
   a long Base64 run in raw bytes for d08, d09 and d11. For d11 the scan for runs of 100+ Base64
   characters found exactly one CRC-valid record, starting at source offset 368272, which is also the
   value submitted for `d11_analysis_hcfg_offset`.
5. **d01 disassembly.** `recover_code` is a six-byte `FUNC` in the symbol table: `mov eax, 0x1fa4`
   then `ret`. The question accepts Intel or AT&T spelling; the Single Solve map used the `objdump`
   AT&T text (`mov $0x1fa4,%eax`) and was accepted.
6. **Submit.** The candidate map had exactly the 110 answer ids, all strings (arrays as compact JSON,
   integers in base 10, booleans lowercase). The Single Solve went CORRECT on its first submission.

Spellings that were accepted in this challenge: machine and architecture as `x86-64` for the ELF
specimens, `little endian` for d01 endianness, `ET_EXEC` for the ELF type, `PE-CLI` and `0x014c` for
the d03 format and machine, `JAR` for the JAR format, `[0,8]` for compression methods. The d03
PE32+ labels in Mastery were `PE32+`.

## Time Trial and Mastery

- **Time Trial.** The same 110 ids and the same SHA-256 as Single Solve (the journal notes the
  member names changed from `*-variant-000.*` to `*-sample.*`, which did not matter because the
  mapper groups members by their `dNN` prefix). The first attempt came with an 80 s window and
  `unique_attempts_remaining: 0` from the start; it expired while the mapper was being adapted.
  `retry` then issued an untimed fallback attempt on the same artifact, and the map went CORRECT
  there. The one adaptation: `objdump` was not installed on that host, so the `recover_code` bytes
  were read through the symbol table and disassembled with `capstone` (Intel syntax, with a space after
  the comma), which the question accepts.
- **Mastery.** 330 ids = 3 deliveries x 110, each its own `evidence.zip`. Again a 200 s first
  attempt that expired, then an untimed fallback with the same artifact hashes. Set 1 and set 3 had
  the Single Solve's layout. **Set 2 swapped d02, d04 and d10 to PE32+ executables** while the rest
  stayed as before, so the mapper was rewritten to classify each specimen by its magic bytes (`MZ`
  versus ELF) and to add a minimal PE section-table reader; the d02 record is read from the start of the
  section literally named `.hcfg`, which needs the file offset and raw size from the PE section
  header, not the virtual ones. The Game Master client had to be extended to fetch every delivery of
  a multi-artifact bundle (it had only fetched the first). Mastery went CORRECT on the third
  submission call; the two earlier calls were transient 503s.
- Neither timed attempt was Hall-of-Fame eligible in the form that was graded, because both were the
  untimed fallbacks after the timed window expired. A later ranked sweep of Days 11 to 15 found this
  day's Binary Time Trial and Mastery pools spent (`retry_variants_unavailable`), so no ranked
  timing exists for either.

## Dead ends and gotchas

- **The bundle is a recap, not a single-language challenge.** The title and prompt talk about D
  templates, but no question asks about the D toolchain. The facts needed come from the same
  parsers used on Days 1 to 10. We did not establish from the saved material what language the d11
  specimen was built with.
- **Container format is not a constant across Mastery sets.** Set 2's d02/d04/d10 were PE32+, set 1
  and 3's were ELF. An extractor that assumes ELF breaks silently. This was noted from earlier days
  and confirmed by magic-byte checks before mapping.
- **`.symtab` names mean every non-empty entry in table order, including duplicates and FILE entries**
  (`crt1.o`, `variant-000.c`, ...). The Single Solve answer for `d01_analysis_symbols` used that reading
  and was accepted in full.
- **Identical specimens across Mastery sets.** The d01 and d03 member-2 specimens hash the same in all
  three sets; this is expected and the answer-lint's "same SHA-256 for several ids" warning on them
  was not a bug.
- **Section parsing detail.** The first PE parse of Mastery set 2 unpacked the section header fields in
  the wrong order and pointed the `.hcfg` read at garbage; the CRC check caught it on a dry run before
  any submission (our own code bug, not a challenge trap).
- Time Trial/Mastery windows and transport errors (503s, a `download_required` after `retry`) cost
  calls but no wrong guesses. Lifecycle details are in the day overview.

## Takeaways

- **HCFG record scanning**: validate every candidate by length bounds, XOR decode, CRC32 and JSON
  parse; ignore anything that fails. A literal `HCFG` marker proves nothing, and the carrier changes
  by block (raw, named section, Base64 run, JAR constant pool).
- **Base64-run carrier**: scan for long (100+) Base64 character runs, strictly decode, then test the
  decoded bytes for the `HCFG` magic; report the run's source offset when a question asks for an offset.
- **Classify by magic bytes, not by day or file name**, because Mastery sets mix ELF and PE across the
  same challenge.
- For a cumulative recap bundle, build the mapper to produce exactly the start response's answer ids and
  lint it against the answer formats before submitting.
- Faster next time: keep one `objdump`-free disassembly path (symbol table plus `capstone`) ready, and
  have the mapper fetch all deliveries of a Mastery bundle from the start.

## Evidence

Leaf: `challenges/day11/binary-reverse-engineering/binary-reverse-engineering-day-11-d-collection-planner/`

- `single-solve/journal.md` (records the Single Solve submission as CORRECT on attempt 1; there is no
  saved `start_response.json` for this mode), `single-solve/artifacts/answer_map.py`,
  `single-solve/artifacts/candidates.json`
- `time-trial/journal.md`, `time-trial/start_response.json`, `time-trial/retry_response.json`,
  `time-trial/artifacts/answer_map.py`
- `mastery/journal.md`, `mastery/start_response.json`, `mastery/retry_response.json`,
  `mastery/artifacts/answer_map.py` (magic-byte classifier plus PE parser)
- No `result.json`, `accepted_plan.json` or `ranked-N/` folder exists for this challenge; the CORRECT
  results rest on the journals and `progress.md` (grep `^| 11 .*d-collection-planner`).
- Library entries named in the journal: BRE-HCFG-001, BRE-HCFG-BASE64-001, BRE-JAR-HCFG-001,
  BRE-TECHNIQUE-001 (ELF symbol inventory), META-ZIP-MEMBER-PATHS-001, META-ZIP-COMPRESSION-001.
- Later ranked-solver notes (not a graded record for this challenge): `tools/ranked/solvers/binary-reverse-engineering-day-11-d-collection-planner/solve.py`
