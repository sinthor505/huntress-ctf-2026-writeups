# Routine Check

| | |
|---|---|
| Day / Lane | 7 / Binary Reverse Engineering |
| Challenge ID | `binary-reverse-engineering-day-07-kotlin-staging-logic` |
| Technique | Static extraction of an XOR/CRC-protected config record (HCFG) from a Java classfile constant pool, plus ELF/PE/CLR header facts |
| Single Solve | Solved (journal and `progress.md` record CORRECT on one 56-field submission); no graded response file was preserved in the leaf |
| Time Trial | Independent (own CORRECT response saved; a later ranked rerun also CORRECT) |
| Mastery | Independent (own CORRECT response saved); no ranked rerun possible (`rotation_exhausted`) |

## Summary

The title and prompt point at Kotlin coroutine state machines, but nothing in the answer set needed
the Kotlin code. Every field is a static fact about a bundle of specimens from Days 1-7, and the new
Day 7 specimen was a ~5 MB Kotlin fat JAR whose only interesting content was a Base64 string in the
`Main-Class` constant pool. Decoding it gave a CRC-valid HCFG record, and the solve was an offline
extractor run over an encrypted ZIP. All three modes were answered from the same extractor, and the
two scored attempts we have saved responses for were accepted on their first valid submission.

## The challenge

Prompt: "Kotlin coroutines don't run - they suspend, resume, and hop between states. Reconstruct
the state machine from the metadata and figure out what this one was staging."

Single Solve delivered one AES-encrypted `evidence.zip` (eight members). The start response asked
for 56 fields, which is the whole cumulative recap: eight fields for each of Days 1 through 7
(`d01_...` to `d07_...`), not just the new Day 7 block. Day 7's eight were:

- `d07_artifact_sha256_1`: SHA-256 of the d07 member (the d07 members, in ZIP central-directory
  order, member 1)
- `d07_analysis_format`: `JAR`
- `d07_analysis_hcfg_version`: integer, as a string
- `d07_analysis_config_campaign` / `_domain` / `_mutex` / `_url` / `_crc32_hex`: fields of the
  embedded config

The older blocks asked ELF class/machine/section names/symbol list/disassembly of `recover_code`
(Day 1), config fields, CRC and XOR key (Days 2, 4, 5, 6), and PE32/CLR machine and metadata
version (Day 3). Every answer value must be a JSON string, including the integer and the three
JSON-array fields.

## Solution

1. **Open the archive.** The ZIP is AES-encrypted (WinZip AE). Its archive comment
   (`huntress:day07-single`) was tried and was not the password; the event's bundle password
   `infected` worked. Members were decrypted in memory with `pyzipper` and never executed. The d07
   member is a single `d07--variant-000.jar`.
2. **Find the entry point.** `META-INF/MANIFEST.MF` names `Main-Class: Day07Variant000`, whose
   classfile is only 7,704 bytes inside a 5 MB JAR (the rest is bundled Kotlin runtime).
3. **Parse the constant pool.** A small bounds-checked classfile parser walks the constant pool and
   keeps `CONSTANT_Utf8` entries of at least 80 Base64 characters. Exactly one qualified here, 508
   characters long. It decoded to bytes starting with `HCFG`.
4. **Validate the HCFG record.** The 12-byte header holds version (1), XOR key (`0x31`), payload
   length (368) and a little-endian CRC32. XOR the declared payload, check the CRC, then parse the
   UTF-8 JSON. The CRC (`0xcf3ea232`) matched, which is the evidence that the right constant had
   been found and decoded correctly:

   ```python
   version, key = decoded[4], decoded[5]
   payload_len = struct.unpack_from("<H", decoded, 6)[0]
   expected_crc = struct.unpack_from("<I", decoded, 8)[0]
   payload = bytes(b ^ key for b in decoded[12:12 + payload_len])
   if zlib.crc32(payload) & 0xFFFFFFFF != expected_crc:
       continue          # not the real record (decoys exist in other days' specimens)
   config = json.loads(payload.decode("utf-8"))
   ```

   Day 7 values: campaign `BR07-000`, domain `node-000.day07.example.invalid`, mutex
   `Global\HuntressBR07_000`, URL `https://node-000.day07.example.invalid/api/BR07-000`.
5. **Hash the JAR.** SHA-256 of the exact decrypted member bytes:
   `13238011183076ed4d2c689f4e46d51a95c895b5c45bd2a374dcaed75de7959b`.
6. **Answer Days 1-6 from the same bundle.** The ELF parts used `readelf`/`objdump`-style static
   reading: Day 1 is ELF64, little endian, `ET_EXEC`, x86-64, 28 section names, 33 non-empty
   `.symtab` names kept in table order (including duplicates), and `recover_code` disassembling to
   `mov eax,0x1fa4; ret`. Day 2 carries its record in an `.hcfg` section; Days 3-5 carry a CRC-valid
   record as a literal in the binary (a .NET DLL with CLR metadata `v4.0.30319` on Day 3); Day 6 is
   the same Java Base64-in-constant-pool layout as Day 7.
7. **Submit.** A mapper (`answer_map.py`) emitted all 56 values and its key set was asserted equal to
   the start response's `answer_ids`. One submission of all 56 went CORRECT.

## Time Trial and Mastery

- **Time Trial.** Same 56 IDs and the same shape, delivered as a fresh archive with a 55 s server
  window. The fixed per-day variant pool meant the d03-d07 members were different variants from
  Single Solve's variant 000 (for example `d03--variant-001.dll`, `d07--variant-005.jar`), so nothing
  was reused from the SS answers; the same mapper simply ran on the new bytes. Server timestamps in the
  saved response put start-to-CORRECT at about 31 s (this included one rejected request, see below).
- **Mastery.** Two deliveries in one attempt (`artifacts[]` had two ZIPs, a 90 s window), 112 IDs
  prefixed `set1.` and `set2.`. Set 1 carried the same members as the Time Trial. Set 2 was the new
  case: its Day 2 and Day 4 specimens were Windows PE32+ executables instead of ELFs. The mapper
  had to be extended to read the `.hcfg` section and machine field from a PE, and to report the
  container as `PE32+` for those two blocks while d05 stayed `ELF` and d03 stayed `PE32_CLR`. The whole
  112-field batch went CORRECT in one call (saved server timestamps put start-to-CORRECT at about 62 s).
- **Ranked reruns.** A later ranked Time Trial rerun with the stored solver (`legacy` route, so a plain
  `retry` was the ranked attempt) was CORRECT in one call with 3.3 s wall time. A ranked Mastery rerun
  was not possible: the original Mastery start already reported `unique_attempts_remaining: 0`, and
  the 2026-11-01 sweep re-confirmed 409 `rotation_exhausted` with nothing consumed (per `progress.md`;
  we did not re-run that check for this page).

## Dead ends and gotchas

- **The Kotlin framing is a red herring for this challenge.** There is no coroutine or state-machine
  question; the specimen's Kotlin runtime is just ballast around a classfile that has one 508-char
  constant. The lane's later days do ask about it: the playbook lists `has_coroutine_runtime` and
  `method_count` fields from the Day 17 block onward, which is where a classfile/`@Metadata` reader
  becomes necessary.
- **Archive comment is not the password.** The ZIP comment looks like a hint and is not one.
- **Cumulative scope.** The prompt is about Day 7 but 48 of the 56 fields belong to Days 1-6. Missing
  that would have left most of the answer set unaddressed.
- **Spellings validated by CORRECT on this day:** endianness `little endian`, Day 3 format `PE32_CLR`,
  Day 3 machine `Intel 80386`, Days 4/5 format `ELF` for ELF members and `PE32+` for PE members,
  section-name list keeping the leading empty string, `.symtab` names in table order with duplicates.
  Spellings that vary by day (`x86-64` vs `AMD64`, `ELF64` vs `ELF`) are in the library's question-facts
  notes; do not carry them across days.
- **Request format.** The first Time Trial POST was rejected with `unknown_property` for being
  incorrectly wrapped; the corrected request was accepted. No answers were graded by the rejected
  one (a harness-side slip; see the day overview).
- **Decoys.** Other days' specimens (Rust, Nim) carry decoy Base64/config strings, so the CRC check
  is what distinguishes a real record from a lookalike. On Day 7 itself only one candidate existed.

## Takeaways

- **HCFG record handling** is a fixed recipe across this lane: header (`HCFG`, version, XOR key,
  length, CRC32), XOR, CRC check, JSON. Only the carrier changes (ELF section, literal in
  `.rodata`/`.data`, Base64 string in a classfile constant pool).
- **Java/Kotlin carrier:** read `Main-Class` from the manifest, parse its constant pool by hand
  (bounded, no JVM), and filter `Utf8` entries by Base64 shape then CRC. This is reusable as-is for any
  JAR variant.
- **Write the extractor so one run covers every delivery.** TT and Mastery were won inside 55-90 s
  windows because the mapper keyed on magic bytes and read the manifest from the start response; the
  PE32+ case in Mastery set 2 was the only generalization needed.
- **Check the answer-ID count first.** 56 (or 112) fields for a "Day 7" title is the signal that the
  recap blocks matter more than the headline specimen.

## Evidence

Leaf base: `challenges/day07/binary-reverse-engineering/binary-reverse-engineering-day-07-kotlin-staging-logic/`

- `single-solve/journal.md`, `single-solve/start-response.json`, `single-solve/artifacts/analyze_day07_jar.py`,
  `single-solve/artifacts/answer_map.py`, `single-solve/submission-request.json` (no saved graded response)
- `time-trial/journal.md`, `time-trial/start-response.json`, `time-trial/submission-response.json`
  (CORRECT), `time-trial/ranked-1/result.json` (CORRECT, 1 call, 3.26 s), `time-trial/ranked-1/pass.log`
- `mastery/journal.md`, `mastery/start-response.json`, `mastery/submission-response.json` (CORRECT),
  `mastery/artifacts/answer_map.py`, `mastery/delivery-1/`, `mastery/delivery-2/`
- Ranked-pass solver: `tools/ranked/solvers/binary-reverse-engineering-day-07-kotlin-staging-logic/solve.py`
- `progress.md` rows for Day 7, including the 2026-11-01 `rotation_exhausted` entry
- Library techniques named in the journals: HCFG records, JAR HCFG extraction, PE/CLR detection, ELF symbol
  inventory, and the BRE question-facts and lane playbook entries.
