# Wasm Up

| | |
|---|---|
| Day / Lane | 16 / Binary Reverse Engineering |
| Challenge ID | `binary-reverse-engineering-day-16-webassembly-optimization-equivalence` |
| Technique | cumulative recap archive (d01-d16) plus a new WebAssembly block; HCFG record in a wasm data segment |
| Single Solve | Independent (journal-recorded CORRECT on attempt 1; saved submission response reads CORRECT) |
| Time Trial | Independent (untimed attempt CORRECT on attempt 1; five later ranked attempts also CORRECT) |
| Mastery | Independent (untimed attempt CORRECT on attempt 1; three later ranked attempts also CORRECT) |

## Summary

The prompt and the ID (`webassembly-optimization-equivalence`) promise a WebAssembly puzzle about
imports, memory and indirect calls. In practice the challenge was a cumulative review: one block of
static-fact questions per earlier day (d01-d15, the same field set as Day 15) plus a small new d16
block for two `.wasm` modules. We copied the previous day's offline mapper, added the d16 block, and
all three modes went CORRECT on the first submission (160, 160 and 800 fields).

## The challenge

Prompt: "WebAssembly runs in a sandbox, but the sandbox has doors: imports, memory, indirect calls.
Map every door in this module and inventory what passes through."

`start` delivered an AES-encrypted `evidence.zip` (password `infected`, SHA-256 checked against the
start response) with 19 members named `d01--...` to `d16--...`; d03, d15 and d16 each have two
members. Single Solve and Time Trial asked 160 questions, ten per day with a few extras (the exact
count per day varies). Mastery asked 800: five deliveries of the same 160-question shape, with IDs
prefixed `setN.`. Formats mixed `pattern`, `exact_text`, `enum`, integer and JSON, and every answer
is a JSON string.

The d16 questions were only eight facts about two modules, `...-direct.wasm` and `...-compact.wasm`:
- `d16_artifact_sha256_1` / `_2`: SHA-256 of each member in ZIP central-directory order.
- `d16_analysis_{direct,compact}_architecture`: enum `x86-64` or `wasm32`.
- `d16_analysis_{direct,compact}_format`: enum of container labels (`elf`, `pe`, `pe_cli`, `jar`, `wasm`, `neko_bytecode`).
- `d16_analysis_compact_config_{campaign,domain,mutex,url}`: fields of the HCFG record in the compact module.

No question asked about imports, exports, memory or indirect calls; the prompt text was thematic only.

## Solution

1. **Recon.** The question list showed d01-d15 repeating the Day 15 field set and d16 as the only new
   block. The Day 15 `answer_map.py` was the starting point.
2. **Reuse, then add d16.** We extended the mapper's day range to 1..16 and added a d16 block:
   SHA-256 of both members, check the 8-byte magic `00 61 73 6d 01 00 00 00`, answer `wasm32` and
   `wasm` for both modules.
3. **The HCFG record.** The compact module's config record is stored literally (no base64, no extra
   encoding) in a data-section payload: bytes `HCFG`, version, one-byte XOR key, little-endian
   length, CRC32, then the XOR-decoded JSON. A plain scan for `HCFG` with CRC32 validation found it.
   Because the question says to parse the section stream, the mapper also walks the sections with
   LEB128 sizes and asserts the record's offset lies inside section id 11 (data):

   ```python
   for sec_id, start, end in wasm_sections(compact_raw):
       if sec_id == 11 and start <= compact_record["offset"] < end:
           in_data = True
   if not in_data:
       raise ValueError("compact HCFG record is not inside a data section")
   ```

   Single Solve values were campaign `BR16-000`, mutex `Global\HuntressBR16_000`, with the domain and
   URL following the same `-000` pattern.
4. **No `objdump` on the workstation.** The Day 15 mapper had used `objdump` for `recover_code`
   disassembly. We added offline fallbacks: decode the constant-return stub directly (`b8 imm32 c3`
   becomes `mov eax,0x...` / `ret`) and read PE COFF symbol tables by hand. These reproduced the
   Day 15 outputs exactly (the d01-d15 part of the Single Solve map was identical to the accepted
   Day 15 candidate file), which is how we trusted them before submitting.
5. **Submission.** One call with all 160 answers; CORRECT. Submission was a separate, authorized
   step from the offline mapping.

## Time Trial and Mastery

- **Time Trial** used the same 160 IDs against a fresh archive with different variants for d01 and
  d03-d07 and d09-d15. The mapper output was diffed against the Single Solve map as a sanity check
  only. The d01 specimen was stripped (no `.symtab`), so its symbol and disassembly answers are `[]`
  and `has_symtab` is false, per the question text. The d16 modules were the same as in Single Solve
  (campaign `BR16-000`). CORRECT on attempt 1.
- **Mastery** delivered five evidence archives. `gm.py artifact` only fetches the first, so we wrote
  a small fetcher that downloaded every capability listed in `start_response.json["artifacts"]` and
  checked each SHA-256. Each set was extracted and mapped separately with a `setN.` answer prefix, then
  merged into one 800-ID map. d16's compact campaign was `BR16-001` to `BR16-005` across the sets.
  Deliveries 1, 3 and 5 carried PE `.exe` specimens for d02 (C++ typeinfo/vtable via COFF symbols),
  d04 (Go) and d10 (Zig); no new answer rules were needed. CORRECT on attempt 1.
- **Ranked reruns.** Later we ran five ranked Time Trial attempts (80 s window, 7.7-11.8 s total
  each) and three ranked Mastery attempts (320 s window, 21.8-26.6 s total each), all CORRECT on the
  first submission, using a wrapper that rebuilds the member manifest from each fresh zip and runs
  the same mapper unchanged. These were speed-board attempts, not part of the original solve.

## Dead ends and gotchas

- **The title and prompt are not the questions.** Nothing about wasm imports, memory or indirect
  calls was asked. Reading the question list before planning saved time.
- **Specimens vary per mode, not rules.** Time Trial and Mastery changed specimen variants (a stripped
  d01 ELF, PE builds for d02/d04/d10 in some Mastery deliveries); the rule set stayed the Day 15 one.
  The mapper has to be manifest-driven rather than tied to one member filename.
- **Mastery needs all deliveries.** Only the first archive downloads by default; the other four have
  to be fetched from the `artifacts` list.
- Per the journals we did not need any diagnostic submissions; there were no rejected answers in
  any mode, so we have no validator quirks to report for this challenge.

## Takeaways

- For a "cumulative review" binary challenge, diff the question IDs against the previous day first;
  the new day is usually a small block on top of an already-validated mapper.
- WebAssembly HCFG: the config record is a literal in a data-section segment, CRC32 valid at the
  `HCFG` marker; confirm the section id (11) rather than assuming offsets, since sizes are LEB128.
- Without `objdump`, a constant-return stub (`mov eax,imm32; ret`) and COFF symbol tables are easy to
  decode by hand, but validate the fallback against a previously accepted answer set.
- We did not determine from the evidence what the `direct` versus `compact` (size-optimized) builds
  differ in beyond the code section; the library note says they carried the same campaign and only
  the code differed, and no question asked us to compare them.

## Evidence

Leaf base: `challenges/day16/binary-reverse-engineering/binary-reverse-engineering-day-16-webassembly-optimization-equivalence/`

- `single-solve/journal.md`, `single-solve/start_response.json`, `single-solve/artifacts/answer_map.py`,
  `single-solve/artifacts/submission-attempt-log.json`, `single-solve/artifacts/submission-attempt-01-response.txt`
- `time-trial/journal.md`, `time-trial/artifacts/answer_map.py`, `time-trial/artifacts/submission-attempt-01-response.txt`,
  `time-trial/ranked-1/result.json` ... `time-trial/ranked-5/result.json`
- `mastery/journal.md`, `mastery/artifacts/fetch_deliveries.py`, `mastery/artifacts/candidate-answers.json`,
  `mastery/artifacts/submission-attempt-01-response.txt`, `mastery/ranked-1/result.json` ... `mastery/ranked-3/result.json`
- `tools/ranked/solvers/binary-reverse-engineering-day-16-webassembly-optimization-equivalence/solve.py`
  (ranked wrapper around the leaf mapper)
- Library: BRE-WASM-HCFG-001 (WebAssembly HCFG and cumulative review archives), BRE-INSPECT-B101
- `progress.md` (grep `^| 16 .*webassembly`)
