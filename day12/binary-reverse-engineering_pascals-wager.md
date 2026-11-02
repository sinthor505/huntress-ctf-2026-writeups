# Pascal's Wager

| | |
|---|---|
| Day / Lane | 12 / Binary Reverse Engineering |
| Challenge ID | `binary-reverse-engineering-day-12-pascal-archive-stager` |
| Technique | static extraction of an XOR-obfuscated, CRC-checked `HCFG` config record from a stripped Free Pascal ELF (plus a cumulative recap of Days 1-11) |
| Single Solve | Solved (journal-recorded) |
| Time Trial | Solved (journal-recorded); a later ranked rerun was `practice_only` |
| Mastery | Solved (journal-recorded) |

## Summary

Pascal's Wager is a 120-field Binary RE challenge delivered as one AES-encrypted ZIP holding a
specimen for each earlier day (d01-d12). The new piece on Day 12 is a stripped Free Pascal ELF
whose configuration is a literal `HCFG` record sitting in `.data`. No code needs to run: scan for
the marker, XOR-decode the payload, check the CRC32, parse the JSON. The same offline extractor and
answer mapper went CORRECT on the first full-map submission in Single Solve and Time Trial, and
Mastery (three deliveries, 360 fields) completed after two submissions expired.

## The challenge

Prompt: "Pascal is betting you won't read its record layouts. Take the wager: unpack the archive
stager's structures and read the configuration straight from memory's own filing cabinet."

`start` returned a download of about 31.9 MB, an AES (WinZip AE-2) ZIP, password `infected`, with
13 central-directory members: d01, d02, d03 (a .NET PE plus its `.runtimeconfig.json`), and d04-d12.
There were 120 questions, ten per day d01-d12, all string answers keyed by IDs like
`d12_analysis_config_domain`. Formats were a mix of exact patterns (64-hex SHA-256), enums
(`ELF32`/`ELF64`, `true`/`false`, the d12 `format` token), JSON arrays (section names, symbol
names, disassembly, ZIP compression methods) and integers.

Each day's block re-asks that day's field set against that day's specimen (ELF header fields and
symbols for d01, an HCFG record in an ELF `.hcfg` section for d02, a CLR version for d03, and so
on). d12 is the day's own block:

- `d12_artifact_sha256_1`, `architecture`, `machine`, `format` (enum `elf`/`pe`/`pe_cli`/`jar`/...)
- `config_campaign`, `config_domain`, `config_mutex`, `config_url`, `config_crc32_hex`
- `hcfg_offset` (the file offset of the record, as an integer string)

The question text spells out the record layout: bytes 0..3 `HCFG`, byte 4 version, byte 5 XOR key,
bytes 6..7 little-endian payload length, bytes 8..11 little-endian CRC32 of the decoded payload,
payload from byte 12; accept the first marker whose bounds, CRC and decoded JSON all validate.

## Solution

1. **Verify and unpack.** Check the download against the `sha256` in the start response, then read
   the AES ZIP with `pyzipper` (stdlib `zipfile` and the system `unzip` can list but not extract AES
   members). Write each member out as raw bytes and record a SHA-256 manifest in central-directory
   order; the `artifact_sha256_N` questions are literally "hash member N in that order".
2. **Triage d12.** The d12 member is a stripped ELF64 x86-64 (`ET_EXEC`) with the sections
   `.note.ABI-tag .text .rodata .data .fpcdata .bss .shstrtab`. `.fpcdata` is the Free Pascal
   runtime marker and is where the "Pascal" in the title comes from; it carries no answer. There
   is no `.symtab` and no `.hcfg` section, so unlike d02 the config has to be found by content.
3. **Scan for the marker.** Search every `HCFG` occurrence and validate each as a record. The
   validator is short:

   ```python
   def parse_hcfg(record):
       if len(record) < 12 or record[:4] != b"HCFG":
           return None
       version, key = record[4], record[5]
       length = struct.unpack_from("<H", record, 6)[0]
       crc = struct.unpack_from("<I", record, 8)[0]
       if 12 + length > len(record):
           return None
       decoded = bytes(b ^ key for b in record[12:12 + length])
       if binascii.crc32(decoded) & 0xFFFFFFFF != crc:
           return None
       cfg = json.loads(decoded.decode("utf-8"))       # failures -> None in the real code
       return cfg if isinstance(cfg, dict) else None
   ```

   Bounds, CRC and JSON are all checked, so incidental `HCFG` bytes elsewhere in the file are
   rejected rather than trusted.
4. **Read the record.** The accepted record sat at file offset 82928, inside `.data` (section
   offset 82624, so 304 bytes in), with XOR key `0x81`, version 1, a 368-byte payload and CRC32
   `0xe4fc81f8`. The decoded JSON had `campaign`, `domain`, `ipv4`, `url`, `mutex`, `windows_path`,
   `linux_path`, `service` and `host_model`. The answers were the string values, e.g. campaign
   `BR12-000`, mutex `Global\HuntressBR12_000`, URL `https://node-000.day12.example.invalid/api/BR12-000`.
5. **Fill the other 110 fields from the same parsers.** ELF header fields, section-name and symbol
   lists, the d01 `recover_code` disassembly (a single `mov eax,0x1fa4; ret`, read with `objdump`
   without running anything), the d03 PE/CLI metadata version (`v4.0.30319`), the d06/d07 JAR
   `Main-Class` constant-pool HCFG and ZIP compression methods (`[0,8]`), and the d08/d09/d11
   Base64-carried HCFG records, each reported with its source offset. Every HCFG candidate was
   accepted only after the same bounds / XOR / CRC32 / JSON checks.
6. **Submit once.** The mapper keyed 120 string values by the exact answer IDs (arrays as compact
   JSON text) and one full-map submission returned `CORRECT` for all 120 fields.

## Time Trial and Mastery

- **Time Trial** gave a fresh 31.9 MB archive (new bytes, new hashes) and an 80 s window. The
  same extractor and mapper were rerun on the fresh bytes (the Single Solve answer values were not
  reused); the full 120-field map was `CORRECT` on attempt 1.
- **Mastery** delivered three archives (one per `setN.` prefix, 360 IDs) under a 200 s window, with
  different specimen mixes per set. Set 1 matched Single Solve's mix. Set 2 had PE specimens for
  d02 (C++), d04 (Go) and d10, so the d02 type-info/vtable predicates were answered from the PE
  COFF symbol table and its `.hcfg` section; set 3 used ELF for d02/d04/d10 again. The extractor
  was run on each delivery independently.
- Submission sequence (from the attempt log): set 1 `accepted`; sets 2 and 3 `expired` (the 200 s
  clock had run out); a fresh untimed continuation was then obtained, sets 2 and 3 were accepted
  and set 1 resubmitted, returning `CORRECT` for 360 of 360 on attempt 6. The timed attempt thus
  did not finish in its window, so the Mastery completion is the untimed continuation.
- A later ranked rerun of the Time Trial (legacy sweep) returned a spent-pool `practice_only`
  payload and issued nothing, so there is no ranked result for this challenge.

## Dead ends and gotchas

- No answer was rejected in any mode; nothing needed an alternate spelling. Several fields are
  pattern-based and accept equivalent standard spellings (the d12 `machine` question lists a
  normalized name, a tool label or a numeric code as acceptable), while the d12 `format` is an
  exact enum token (`elf` for this specimen).
- The title and the `.fpcdata` section suggest Pascal-specific analysis (RTTI, ShortStrings).
  None was needed: the question names the record layout outright and the record is plain bytes.
- `hcfg_offset` is the offset of the `HCFG` marker within the member file, not a virtual address and
  not an offset within `.data`. For d02 the offset of the record coincided with the `.hcfg`
  section's start, which served as a cross-check there.
- Day 12 is cumulative: 110 of the 120 fields re-ask d01-d11 shapes. Because a d12-only extractor
  covers just ten of them, the work is mostly a general ELF/PE/JAR reader. For the Day 12 delivery
  the ZIP also has two d03 members, so "member 1" per day must be resolved by the `dNN--` prefix,
  not by absolute position.
- The Mastery timer ran out before sets 2 and 3 were submitted. See the Day 12 overview for the
  process side of that.

## Takeaways

- Technique name: **embedded HCFG record carving**. Find a fixed magic, treat the following header
  as untrusted, bound the length, decode with the stated XOR key, and confirm with a CRC32 and
  JSON parse before believing it. A CRC-valid record is a far better discriminator than the magic
  alone, and it makes stripped binaries (no symbols, no named section) as easy as unstripped ones.
- For "bundle of earlier-day specimens" challenges, build one reader per container type (ELF, PE
  incl. CLR metadata, JAR constant pools) and drive it from the ZIP manifest instead of from
  filenames.
- Faster next time: do not read the language runtime for hints; read the question's record layout
  first. For Mastery, run all deliveries concurrently and submit set by set as each finishes, before
  the single shared timer expires.

## Evidence

Leaf: `challenges/day12/binary-reverse-engineering/binary-reverse-engineering-day-12-pascal-archive-stager/`

- `single-solve/journal.md`, `single-solve/start_response.json`
- `single-solve/artifacts/extract_archive.py`, `extract_facts.py`, `answer_map.py`,
  `candidate-answers.json`, `facts.json`, `submission-attempt-log.json` (attempt 1 `CORRECT`)
- `time-trial/journal.md`, `time-trial/artifacts/submission-attempt-log.json` (attempt 1 `CORRECT`),
  `time-trial/ranked-1/result.json` (`practice_only`)
- `mastery/journal.md`, `mastery/artifacts/submission-attempt-log.json` (six attempts, last `CORRECT`),
  `mastery/artifacts/candidate-set1.json`, `mastery/mode-sets/set2`, `mastery/mode-sets/set3`
- `tools/ranked/solvers/binary-reverse-engineering-day-12-pascal-archive-stager/solve.py`
  (later generalized solver; records the spellings that were CORRECT in all three modes)
- `library/binary-reverse-engineering/hcfg-records/` (BRE-HCFG-001), `progress.md`
  (grep `^| 12 .*pascal-archive-stager`)
