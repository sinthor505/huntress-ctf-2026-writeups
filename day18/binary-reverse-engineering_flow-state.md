# Flow State

| | |
|---|---|
| Day / Lane | 18 / Binary Reverse Engineering |
| Challenge ID | `binary-reverse-engineering-day-18-control-flow-reconstruction` |
| Technique | cumulative recap bundle: static ELF/PE/JAR/WASM parsing, HCFG record recovery and storage classification |
| Single Solve | Solved (journal-recorded) |
| Time Trial | Solved (journal-recorded) |
| Mastery | Solved (journal-recorded) |

## Summary

The prompt promised call-graph reconstruction and an injection hunt. None of the 216 questions asked about
control flow. They were the previous days' per-specimen fact blocks plus one new block for Day 18, all about
file structure and an embedded XOR-encoded `HCFG` configuration record. We re-used the validated Day 15
mapper, added a WebAssembly branch for Day 16 and a small set of new field readers, and each mode went
CORRECT on its first submission. The leaf holds no `result.json`; the CORRECT outcomes are recorded in
the journals, `progress.md` and the attempt logs.

## The challenge

Prompt: "Follow the calls and the program tells you its story. Reconstruct the call graph, read the API
intent at each edge, and prove no injection hides in the flow." The wording was misleading. A grep of the
Single Solve question text for "call graph", "control flow" and "injection" found nothing.

- **Single Solve:** one AES ZIP (password `infected`), about 32.8 MB, 21 members, untimed. It carried
  d01-d15 specimens (ELF, a .NET DLL with its runtimeconfig, two JARs), two WebAssembly modules for d16 and
  one ELF each for d17 and d18. The d01-d15 members matched the Day 15 and Day 17 archives byte for byte.
  216 answer IDs, 12 per day for d01-d18. Formats: 80 pattern, 74 exact_text, 28 enum, 26 integer, 8 json.
- **Time Trial:** same layout and the same 216 IDs, different `variant-NNN` builds (for example d17 and d18).
- **Mastery:** four deliveries, each with the same layout, 864 IDs prefixed `set1.` to `set4.`.

The question kinds were mostly `*_config_{campaign,domain,mutex,url,crc32_hex}`, `*_hcfg_version`, `*_hcfg_offset`,
`*_xor_key_hex`, `*_format`, machine/architecture, and `artifact_sha256_N`. The d01-d14 blocks also had
older-format facts such as ELF section names, symbols, a disassembly, JAR compression methods and C++/Go/Rust
presence predicates.

## Solution

1. **Recon.** We downloaded the archive, checked its SHA-256 against the start response and extracted it
   with `pyzipper`. We kept the central-directory order and a per-member hash manifest. Several questions
   depend on that order.
2. **Starting point.** The question text matched Day 15's apart from two points: d03 added a "use the unique
   `.dll`" preamble, and 30 d01-d15 fields were new (`.huntress` marker, `recover_code`, mangled and typeinfo
   symbols, HCFG version/xor/offset/storage, section names, JAR class facts, Go/Rust/coroutine predicates).
   We copied the Day 15 mapper and extended it.
3. **HCFG recovery.** An HCFG record has a 12-byte header (version, one-byte XOR key, little-endian payload
   length, CRC32) followed by an XOR-encoded JSON payload. The rule was to take the first record whose bounds,
   CRC32 and JSON all validate, so a stray `HCFG` byte pattern with a bad CRC is skipped.
   - d17 and d18 (ELF): the record is a literal in the raw file, and `hcfg_offset` is the file offset of the magic.
   - d16 (WebAssembly): the record is a literal inside a data segment. The plain literal scanner found it.
     Format token `wasm`, architecture `wasm32`.
   - d08 to d14: some records exist only after decoding a long Base64 run, and the offset is that of the
     run's first character, checked to be a run boundary.
4. **`hcfg_storage` classification.** This enum was the part with no obvious definition. We applied the
   rules in order, and the validator accepted them as written:
   - `base64` if the winning record exists only after decoding a Base64 run. A CRC-invalid literal hit, as in
     d08, is a decoy.
   - `raw_hcfg_section` if the record starts in a section named exactly `.hcfg`.
   - `raw_const_array` if it sits in another mapped section and some code operand (imm32 or RIP-relative
     displacement) resolves to the exact record start. Pascal d12 was this case: the record is in `.data`
     and a `movabs` references it.
   - otherwise `raw`.
5. **d01 disassembly without objdump.** The Windows host had no `objdump`. The fallback decoded
   `recover_code` with capstone and rendered it objdump-style, with no space after the commas. The result
   matched the value Day 15 had already had accepted. This needs `python -P`, not `-I`, so the user-site
   capstone import works.
6. **Submission.** The key set of the candidate map was checked against `answer_ids`, and every value was a
   string. All 216 IDs went in one call and returned CORRECT.

## Time Trial and Mastery

- **Time Trial:** `start` returned an 80 s server window although `run_for_time` was not requested. The
  unchanged mapper re-ran in about 5 s on the new variants. The values were re-derived from the new bytes,
  not copied. The journal records submission about 17 s after start, CORRECT on call 1.
- **Mastery:** four deliveries inside a window of about 265 s. The artifact endpoint helper returns only the
  first delivery, so all four were downloaded from the `artifacts[]` list. Sets 2 and 4 contained PE members
  (the d02, d04 and d10 `.exe` builds), and the mapper's ELF-only readers failed on set 2. We added a native
  COFF/PE reader inside the window: primary symbols only, long names through the string table, aux records
  skipped, section-definition symbols kept, `/N` section names resolved, plus a PE storage classifier. The
  mapper ran per set on unprefixed copies of the questions, and the results were merged under the
  `setN.` prefixes (864 of 864 IDs). The single 864-ID submission returned CORRECT about 110 s after start.

## Dead ends and gotchas

- **Title and prompt vs questions.** Nothing here concerns call graphs or injection. Read the question IDs
  (`dNN_analysis_*`) before choosing an approach.
- **Decoy HCFG hits.** A raw `HCFG` match is not enough. In d08 the literal hit had a bad CRC and the real
  record was inside a Base64 run.
- **Format depends on bytes, not extension.** `analysis_format` is classified from magic and parsed
  container. Mastery mixed PE and ELF builds of the same day, so the extractor has to dispatch on magic.
- **Unrequested clock.** Time Trial and Mastery both returned deadlines although we asked for untimed play,
  so the download, map and submit steps were scripted into a single pass.
- No answer was rejected. All three modes were accepted on the first submission, so we have no validator
  evidence about alternate spellings for these fields.

## Takeaways

- **Cumulative recap bundle.** From about Day 10 the binary lane re-asks every earlier day's fact block
  plus a new one. Start from the previous day's validated mapper, run it on the new bytes, diff the output
  shape, and write only the new block.
- Keep a native ELF64/PE/COFF/classfile reader in the toolbox so a missing `objdump` or `readelf` does not
  block the run. The HCFG scan plus the storage classifier covers most of the lane.
- For the Mastery layout, map each delivery on its own and merge with the prefix afterwards, asserting the
  merged key set equals `answer_ids`.
- Library entries (by name): BRE-HCFG-001, BRE-HCFG-BASE64-001, BRE-JAR-HCFG-001, BRE-FLOWSTATE-001,
  superseded by BRE-RECAP-B105 and BRE-INSPECT-B101.

## Evidence

Leaf base: `challenges/day18/binary-reverse-engineering/binary-reverse-engineering-day-18-control-flow-reconstruction/`

- `single-solve/journal.md`, `single-solve/start_response.json`,
  `single-solve/artifacts/answer_map.py`, `single-solve/artifacts/extra_fields.py` (new-field readers and
  `classify_storage`), `single-solve/artifacts/submission-attempt-log.json`
- `time-trial/journal.md`, `time-trial/start_response.json`
- `mastery/journal.md`, `mastery/start_response.json`, `mastery/artifacts/run_sets.py`
- `progress.md` (grep `^| 18 .*control-flow-reconstruction`), `challenges/day18/submission-attempts-binary.md`
- No `result.json`, `accepted_plan.json` or `manual_solve.json` in any mode directory, and no `ranked-N/`
  folder. The per-mode CORRECT outcomes rest on the journals, the progress rows and the attempt log.
