# Nimble

| | |
|---|---|
| Day / Lane | 9 / Binary Reverse Engineering |
| Challenge ID | `binary-reverse-engineering-day-09-nim-generated-runtime` |
| Technique | Static binary-format recap: ELF/PE headers, symbol tables, HCFG config records (XOR + CRC32), Base64-carried records, JAR constant pools |
| Single Solve | Independent (journal plus a saved `CORRECT` submission response; 1 call) |
| Time Trial | Independent (saved `CORRECT` submission response; later ranked reruns also `CORRECT`) |
| Mastery | Independent (saved `CORRECT` submission response after 8 calls; later ranked reruns also `CORRECT`) |

## Summary

The prompt talks about spotting Nim runtime fingerprints, but the 90 questions never ask about Nim. They are a cumulative
recap of the Day 1-9 binary challenges (ELF/PE/CLR/JAR header facts plus one embedded HCFG config record per specimen),
and the only genuinely new part is a Base64-carried record in a small ELF with two decoys. All 90 fields were derived
statically in one submission. Mastery repeated the recap five times and exposed one bug: a symbol predicate that
silently read only ELF symbol tables when the specimen was a PE.

## The challenge

Prompt: "Nim compiles down to C, but the generated names give it away. Learn to spot Nim's runtime fingerprints hiding
inside C-shaped clothing." We did not find anything Nim-specific in the questions, and our notes never established that
the d09 specimen is a Nim build (the only compiler-looking string we saw in the d01 symbol list was a `variant-000.c`
FILE entry). Treat the title and prompt as flavor text.

Delivery: one AES-encrypted ZIP (password `infected`; WinZip AE-2 needs `pyzipper`, not stdlib `zipfile`) of about
31 MB with 10 members named `d01--` to `d09--`: ELFs for d01, d02, d04, d05, d08 and d09, a managed PE DLL plus
`runtimeconfig.json` for d03, and JARs for d06 and d07. The d08 ELF alone is 67 MB uncompressed. The start response
lists 90 answer IDs, all string-typed, one block per earlier day:

- d01: ELF header facts, `.symtab` name list, section names, `recover_code` disassembly, entry point.
- d02/d04/d05/d06/d07: container/architecture labels and five to seven fields from an HCFG record
  (campaign, domain, mutex, URL, CRC32, XOR key, version), plus C++ typeinfo/vtable symbol predicates for d02.
- d03: compound `PE32_CLR` format token, machine, CLR metadata version, HCFG fields.
- d08/d09: HCFG fields from a Base64 run; d09 also asks for the offset of the winning carrier.
- Every member also has a SHA-256 field.

## Solution

1. Decrypt the archive with `pyzipper`, hash every member, and keep ZIP central-directory order (several questions
   say "member N in central-directory order"; d03 has two members, DLL then runtimeconfig).
2. Write one stdlib parser per format: ELF header/section/`SHT_SYMTAB` reader, PE header and CLR metadata-root reader,
   and a JAR reader. Nothing was executed; `objdump` was used only to disassemble the bytes of `recover_code`
   (`mov eax,0x1fa4; ret`).
3. HCFG record layout, as the questions spell it out: bytes 0-3 `HCFG`, byte 4 version, byte 5 XOR key, bytes 6-7
   little-endian payload length, bytes 8-11 little-endian CRC32 of the decoded payload, payload from byte 12. Decode,
   check bounds and CRC32, then require valid UTF-8 JSON:

   ```python
   def hcfg_record(record):
       if len(record) < 12 or record[:4] != b"HCFG": return None
       version, key = record[4], record[5]
       length = struct.unpack_from("<H", record, 6)[0]
       stored_crc = struct.unpack_from("<I", record, 8)[0]
       if 12 + length > len(record): return None
       payload = bytes(b ^ key for b in record[12:12 + length])
       if zlib.crc32(payload) & 0xFFFFFFFF != stored_crc: return None
       ...  # json.loads(payload) must give a dict
   ```

   Only a record that passes every check counts. A bare `HCFG` marker is not evidence.
4. d09 is where the checking matters. The ELF has two literal `HCFG` occurrences (byte offsets 30633 and 30664) that
   fail validation (the journal records an unsupported version). The question says to test every non-overlapping run
   matching `[A-Za-z0-9+/]{100,}={0,2}`, Base64-decode it, and accept the first decoded string that *begins* with a
   CRC-valid record. That run starts at byte offset 30696 (CRC32 `0x9614e1a8` in Single Solve), which is also the
   answer to `d09_analysis_hcfg_offset`: the offset of the first Base64 character of the carrier, not of the HCFG
   magic inside the decoded bytes. d08 follows the same rule but with literals tried first (its literal at byte
   90031 is invalid; the valid carrier is a run at offset 18366768).
5. d06/d07: parse the JAR, read the `Main-Class` classfile's constant pool, Base64-decode each UTF-8 constant of at
   least 80 Base64 characters and take the first CRC-valid record. The compression-method fields come from the
   central directory including directory entries (`[0,8]` here).
6. Assemble 90 string answers keyed by the exact IDs from the start response (key-set audit: 90 expected, 90 present,
   none extra) and submit once. Single Solve returned `CORRECT` with flag `93dac172a6494824bfba220bdf7169ac`.

## Time Trial and Mastery

- **Time Trial:** same 90 IDs, a fresh archive. Values differ from Single Solve (for example the d09 CRC32 became
  `0xa10eee2b`) so nothing was reused; the Single Solve mapper was copied and re-run on the new bytes. The attempt
  was untimed (`run_for_time: false`, no deadline). One call, `CORRECT`.
- **Mastery:** 450 IDs (`set1.` to `set5.` prefixes) over five archives with a 320 s window (started 05:14:05,
  `CORRECT` at 05:17:21 per the journal). Each archive is a variant of the same ten-member layout, but the d02 C++
  specimen is a PE in sets 2 and 4 instead of an ELF. Submission calls, in order: full 450 batch `incorrect`;
  set 1 `accepted`; set 2 `incorrect`; set 3 `accepted`; set 4 `incorrect`; set 5 `accepted`; set 2 corrected
  `accepted`; set 4 corrected `CORRECT`. (The first full batch tells you only that something is wrong; splitting by
  set located it.)
- Later reruns: the archive holds ranked-attempt result files for Time Trial (windows of 80 s, 5-10 s total per
  pass, one call each) and Mastery (320 s window, 8-22 s total per pass, one call each), all `CORRECT` except two
  passes that did not run because the solver environment lacked `pyzipper` (a tooling issue, not a challenge issue;
  see the day overview).

## Dead ends and gotchas

- **PE symbol tables.** The d02 predicates are `/ZTI.*DiscoveryEngine/` and `/ZTV.*DiscoveryEngine/` over "a nonempty
  symbol name". The first mapper fell back to `false` for any PE because it only knew ELF `SHT_SYMTAB`. PE files have
  a COFF symbol table (`PointerToSymbolTable`/`NumberOfSymbols` in the COFF header). Both PE d02 specimens contain
  `_ZTI15DiscoveryEngine` and `_ZTV15DiscoveryEngine` (also in `.rdata$...` entries), so the right answer is `true`
  for both. Parsing it requires 18-byte records, skipping each record's auxiliary entries (`record[17]` of them), and
  resolving names whose first four bytes are zero through the string table. This cost sets 2 and 4 their first
  submission and was the only substantive miss.
- **Decoy HCFG markers.** In d08 and d09 the literal `HCFG` strings are traps; validate bounds, CRC32 and JSON, and
  respect the order the question states (literals first for d08, Base64 runs only for d09).
- **Offset semantics.** `d09_analysis_hcfg_offset` is the source offset of the Base64 run, not the position of the
  record inside the decoded bytes.
- **Accepted spellings.** Values that were accepted: d01 endianness `little endian`, d03 format `PE32_CLR`, d03 machine
  `Intel 80386`, d04/d05 format `ELF` for ELF64 specimens. Other pattern fields accept several equivalent labels, but
  enum and exact-text fields are exact.
- **Download/hash caution.** In Mastery, use each delivery's own download link and verify its SHA-256; the generic
  artifact operation can return a different sample.
- **`d01_analysis_symbols`** was not a problem here, but it is the field that cost many attempts on other days:
  submit every nonempty `.symtab` name in table order, FILE entries and duplicates included.

## Takeaways

- Read the questions before the prompt: a themed title ("Nim") can sit on top of a pure recap bundle.
- For embedded configs, never trust a magic number; validate length, XOR, CRC32 and JSON, and reproduce the order the
  question specifies. A one-file scanner that also tests Base64 runs found the same carrier and CRC as our mapper.
- A predicate that talks about "symbol names" applies to whichever container the specimen turns out to be. Test it
  against every container variant in a Mastery bundle, not only the one seen in Single Solve.
- When a big batch is `incorrect` with no per-field feedback, submit per set (or per family) to localize the failure.
  Searchable names: HCFG record, Base64 carrier run, COFF symbol table, ELF `SHT_SYMTAB`, JAR constant pool.

## Evidence

Leaf base path: `challenges/day09/binary-reverse-engineering/binary-reverse-engineering-day-09-nim-generated-runtime/`

- Single Solve: `single-solve/journal.md`, `single-solve/start_response.json`, `single-solve/artifacts/answer_map.py`,
  `single-solve/artifacts/submission-attempt-01-response.json` (`CORRECT`).
- Time Trial: `time-trial/journal.md`, `time-trial/artifacts/answer_map.py`,
  `time-trial/artifacts/submission-01-response.json` (`CORRECT`), `time-trial/ranked-N/result.json`.
- Mastery: `mastery/journal.md`, `mastery/artifacts/answer_map.py` (`coff_symbols`),
  `mastery/artifacts/submission-*-response.json` (8 calls, final `CORRECT`), `mastery/ranked-N/result.json`.
- Later per-challenge solver: `tools/ranked/solvers/binary-reverse-engineering-day-09-nim-generated-runtime/solve.py`.
- Days 1-10 have no `_run/` folder; the Single Solve journal states the start time as 2026-10-11 (from the Game
  Master response) and the Time Trial and Mastery records as 2026-10-20.
