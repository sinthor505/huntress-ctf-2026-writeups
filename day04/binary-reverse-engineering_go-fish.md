# Go Fish

| | |
|---|---|
| Day / Lane | 4 / Binary Reverse Engineering |
| Challenge ID | `binary-reverse-engineering-day-04-go-runtime-fingerprints` |
| Technique | Static triage of a cumulative evidence bundle: ELF/PE header and symbol tables, XOR-encoded `HCFG` config records, a stripped Go specimen |
| Single Solve | Solved (journal-recorded): the journal records our own `CORRECT` response; no `result.json` |
| Time Trial | Independent (original attempt `CORRECT`, plus a ranked re-run with `result.json` `CORRECT`) |
| Mastery | Independent (`mastery/artifacts/submission-response.json` reads `CORRECT`; no `result.json`, and no ranked rerun because the Mastery pool was exhausted) |

## Summary

The title and prompt point at Go build info, but the 32 questions are a cumulative recap of Days 1-4: hashes and
header facts for four earlier specimens, plus the decoded `HCFG` configuration of the Day 2, 3 and 4 binaries. The
Go specimen is only ever asked about through its hash, header and `HCFG` record. A stdlib-plus-`pyzipper` parser
answered every field; the only real work was making it handle ELF and PE variants in Mastery.

## The challenge

Prompt: "Go binaries are chatty: they ship with build info, function names, and file paths baked right in. Fish
them out and map what this one was built to do."

- Delivery: one `evidence.zip` (about 850 KB), WinZip AES-encrypted with the password `infected`. Members:
  `d01--variant-000.elf` (a small C ELF), `d02--*` (C++ specimen), `d03--*.dll` plus its
  `.runtimeconfig.json` (a .NET assembly), and `d04--*` (the roughly 1.9 MB statically linked Go binary).
- Answers: 32 string fields (`d01_*` through `d04_*`). Formats were mostly `exact_text` or `pattern`, with a few
  JSON arrays (section names, symbols, disassembly).
- Field families:
  - Hashes of members.
  - Header facts: ELF class, machine, `e_type`, endianness, PE format, PE machine, .NET metadata version.
  - Day 1 `.symtab` names and the disassembly of its `recover_code` function.
  - For the Day 2-4 specimens, the decoded `HCFG` fields: campaign, domain, mutex, URL, stored CRC32, and for Day 2
    the XOR key.

## Solution

1. **Open the bundle.** The archive is AES-encrypted ZIP (extra field `0x9901`), so stdlib `zipfile` is not enough;
   `pyzipper` with `infected` opens it. The archive comment (`huntress:day04-single`) and member names already
   identify which dN each file belongs to. Nothing was executed.
2. **Headers.** Hand-written `struct` parsers read the ELF header and section table (class, machine, `e_type`,
   `EI_DATA`, section names in table order) and the PE/COFF header (machine, `PE32` vs `PE32+`). The .NET
   metadata version comes from the `BSJB` signature in the DLL (`v4.0.30319`).
3. **Day 1 symbols and disassembly.** `d01_analysis_symbols` was answered as every non-empty name in `.symtab`,
   in table order, including `FILE` entries and duplicates (`crtstuff.c` appears twice). `recover_code` was a
   two-instruction stub, `mov $0x1fa4,%eax; ret`, disassembled with GNU `objdump`.
4. **HCFG records.** Each of the d02, d03 and d04 binaries carries a record found by searching for the ASCII
   marker `HCFG`:

   ```python
   start = data.find(b"HCFG")
   version, key = data[start + 4], data[start + 5]
   length = struct.unpack_from("<H", data, start + 6)[0]
   expected_crc = struct.unpack_from("<I", data, start + 8)[0]
   plain = bytes(v ^ key for v in data[start + 12:start + 12 + length])
   assert zlib.crc32(plain) & 0xffffffff == expected_crc   # validates the key and offset
   config = json.loads(plain)   # campaign, domain, mutex, url
   ```

   The stored CRC32 and the XOR key are themselves answer fields, and the CRC over the decoded JSON matched in
   every specimen, which confirmed the decode. In the Single Solve archive the keys were `0x31` (d02), `0x41` (d03)
   and `0x51` (d04). The d04 record sat at file offset `0x1baa60`, after `.go.buildinfo` at `0x1b9000`.
5. **Go build info (confirmation only).** The d04 ELF is stripped, but `.go.buildinfo` and `strings` showed
   `go1.25.1`, build path `_/work/variant-000` and `GOARCH=amd64`. No question asked for these. They only
   confirmed that `x86-64` from the ELF header and the HCFG record belonged to the Go specimen.
6. **Submit.** The 32 values were submitted in one call under the exact `answer_ids` from the start response. The
   Single Solve journal records `{"result":"CORRECT"}`. Example values: `d04_analysis_config_campaign` =
   `BR04-000`, `d04_analysis_config_domain` = `node-000.day04.example.invalid`,
   `d04_analysis_config_crc32_hex` = `0x2b747600`.

## Time Trial and Mastery

- **Time Trial.** Same 32 ids, but a fresh archive with new variants: the d03 DLL changed (campaign `BR03-001`)
  and d04 became variant 006 (campaign `BR04-006`, CRC `0xdae5ff06`); d01 and d02 were byte-identical to the
  Single Solve ones. The window was 50 s, so the Single Solve parser was wrapped into `solve_attempt.py`. It reads
  the live start response, derives values from the new archive, and asserts that its key set equals `answer_ids`.
  Submitted once, `CORRECT`. A later ranked re-run through a per-challenge solver also went `CORRECT` in one call
  (about 1.75 s end to end, parse about 0.1 s).
- **Mastery.** Two deliveries, so 64 ids with `set1.` and `set2.` prefixes, under an 80 s deadline. Set 1 was the
  ELF chain (d04 `BR04-006`). Set 2 delivered d02 and d04 as **PE32+** executables instead of ELF
  (d04 campaign `BR04-002`). The first solver run stopped at that point because it assumed ELF. The fix classifies
  each specimen by magic bytes (`\x7fELF` or `MZ`) and takes architecture and format from whichever header is
  present. After that all 64 values were submitted together and the response was `CORRECT`. The Mastery start
  reported no unique attempts remaining, so there was no second try.

## Dead ends and gotchas

- **The title is a decoy.** Nothing asks for Go function names, source paths or the pclntab. Time spent on Go
  internals would have been wasted. `GOARCH` and the ELF machine field are separate facts. The question asks for
  the header's machine.
- **Mastery mixes ELF and PE for the same logical specimen.** Which d02 and d04 you get as PE varies by set, and
  the question text for `d04_analysis_format` accepts ELF or PE labels. Dispatch on magic bytes, not on file
  extension or on set 1's shape. A Go PE has no COFF symbol names, and its HCFG literal lives in `.data` rather
  than `.noptrdata`; neither mattered for these questions.
- **Accepted spellings.** Only the spellings the validator took were replayed later: `PE32_CLR` for the compound
  d03 container (the pattern also allows CLI/.NET), `I386` for the PE machine, `little-endian` for endianness,
  and AT&T `mov $0x1fa4,%eax` for the disassembly (Intel was also permitted by the format text). We did not
  determine whether other spellings would have passed.
- **Cumulative fields change with the variant.** d03 hashes and campaigns, and the d04 hash, campaign, domain,
  mutex, URL and CRC, differ per attempt. d01 and d02 stayed fixed in the saved TT and SS archives. Never reuse an
  earlier mode's answer map.
- **ZIP corruption in the local copies.** Some saved archives were later found damaged by line-ending conversion.
  Fresh downloads and SHA-256 checks against the start response avoid that.

## Takeaways

- Read the question list before the prompt. A "language fingerprint" title can hide a generic cumulative recap
  whose real content is `HCFG` XOR-config decoding (technique: XOR-key record with CRC32 self-check) and header
  parsing.
- Build one parser that is keyed by the start response's `answer_ids` and asserts the key sets match; reuse it
  unchanged across Single Solve, Time Trial and Mastery, and expect only the container type (ELF/PE) to vary.
- When a fixed-size window is 50-80 s, script download, decode and submit in one pass. Parsing took under a second
  once written; most of the clock is network.
- Go specifics worth keeping for other challenges: a stripped Go ELF keeps `.go.buildinfo`, and Go PEs keep their
  build info in `.data` (reusable `go_buildinfo` parser: "Go build info parser").

## Evidence

Leaf: `challenges/day04/binary-reverse-engineering/binary-reverse-engineering-day-04-go-runtime-fingerprints/`

- `single-solve/journal.md`, `single-solve/artifacts/solve_evidence.py` (parsers), `solve_attempt.py` (start-response
  mapper), `submit.json`, `answer-map.json`
- `time-trial/journal.md`, `time-trial/artifacts/submission-response.json` (CORRECT),
  `time-trial/ranked-1/result.json` and `accepted_plan.json`
- `mastery/journal.md`, `mastery/questions.md`, `mastery/artifacts/submission-response.json` (CORRECT)
- `tools/ranked/solvers/binary-reverse-engineering-day-04-go-runtime-fingerprints/solve.py` (later per-challenge solver)
- `progress.md` rows for Day 4 `binary-reverse-engineering-day-04-go-runtime-fingerprints`: all three modes marked Solved.
