# Resourceful

| | |
|---|---|
| Day / Lane | 20 / Binary Reverse Engineering |
| Challenge ID | `binary-reverse-engineering-day-20-resources-and-overlays` |
| Technique | cumulative recap bundle with tagged trailing overlays (PK19 packed, RS20 resource) holding an HCFG config record |
| Single Solve | Solved (journal-recorded) |
| Time Trial | Independent |
| Mastery | Independent |

## Summary

"Resourceful" is the Day 20 binary-lane recap bundle: 240 questions over 20 members (one block per day, d01 to d20), where the new material is how binaries hide data past the end of their sections. d16 hides the config in a WebAssembly data segment, d17/d18 use a plain literal record, d19 appends a scrambled `PK19` overlay and d20 appends an `RS20` resource overlay. Most of the answer set was already covered by the Day 15 mapper; the new work was the d16-d20 field blocks and, in Mastery, PE builds of three earlier specimens. All three modes were CORRECT in a single submission call each.

## The challenge

Prompt: "Binaries have attics: resources, overlays, bytes past the end of the sections. Rummage through this one's hidden storage and map every boundary." Single Solve and Time Trial each delivered one AES-encrypted ZIP (password `infected`, about 33 MB) with 23 files: d01 to d20 specimens, d16 as two WASM modules, d03 as a `.dll` plus its runtimeconfig. There were 240 answer IDs, 12 per day, in mixed formats (exact text, integer, pattern, enum, JSON). Mastery delivered five such ZIPs under one attempt, 1,200 IDs prefixed `set1.` to `set5.`. Every answer had to be a string.

The d16-d20 blocks are all variations on one record, HCFG: ASCII `HCFG`, version byte, one-byte XOR key, u16le payload length, u32le CRC32 of the decoded payload, then the XOR-encoded payload, which is a JSON object with `campaign`, `domain`, `mutex` and `url`. What changes per day is where the record is stored:

| Day | Carrier | How the record is reached |
|---|---|---|
| d16 | two WASM modules | walk the section stream, scan data-segment bytes for a CRC-valid literal record |
| d17, d18 | ELF | literal `HCFG` scan, first record whose bounds, CRC and JSON validate |
| d19 | ELF + `PK19` overlay | `plain = ror8(b ^ k2, 3) ^ k1` over the packed bytes, SHA-256 of the result checked, record at offset 0 |
| d20 | ELF + `RS20` overlay | u32 resource length at +8, u16 scratch length at +12, SHA-256 of the scratch at +16, resource at +48, scratch after it; record at resource offset 0 |

Each of d19/d20 also asked for the overlay offset, `file_length - offset`, and (d20) the resource size field.

## Solution

1. **Recon.** The ZIP's SHA-256 matched the one in the start response. The members d01 to d15 were byte-identical to the Day 15 variant-000 specimens (hash-compared against the Day 15 manifest), so 150 of the 240 IDs belonged to the Day 15 bank.
2. **Reuse the validated mapper.** The Day 15 `answer_map.py` (already graded CORRECT on that day) was reused unchanged for those 150 IDs, and its recomputed values were diffed against the Day 15 candidate answers before anything new was added.
3. **New extras for d01 to d15.** The other 90 IDs included extra fields on the old specimens: marker, `recover_code`, symbol and section names, predicates, config storage class and offsets. `recover_code` on d01 is a lone `mov eax, imm32 ; ret`; with no `objdump` on the host, the six-byte body (`b8 imm32 c3`) was decoded directly and rendered in objdump spacing, as the accepted Day 15 form was.
4. **Storage class of the config record** (literal in `.hcfg`, literal in another section, referenced constant array, Base64). The rule used was ordered: find the CRC-valid literal record, map its file offset into a non-NOBITS section, then check for a reference to its address. d12's record sat in `.data`, with its exact address appearing as a `movabs` immediate in `.text`, so it was `raw_const_array`; d14's record began at the start of `.hcfg` so it was `raw_hcfg_section`; d08/d09/d11/d13 only decoded through Base64 runs so they were `base64`.
5. **d16 to d18.** The WASM modules and the d17/d18 ELF files were handled by the same HCFG scanner pointed at the section stream or the raw file.
6. **d19 and d20 overlays.** The overlay is located with `rfind` on the tag (earlier occurrences can be code bytes). The solver checked that the declared lengths land exactly on end of file and that the embedded SHA-256 verifies, then scanned the decoded region for a CRC-valid record at offset 0:

   ```python
   off = raw.rfind(b"RS20")
   rlen = struct.unpack_from("<I", raw, off + 8)[0]
   slen = struct.unpack_from("<H", raw, off + 12)[0]
   digest = raw[off + 16 : off + 48]
   res = raw[off + 48 : off + 48 + rlen]
   scratch = raw[off + 48 + rlen : off + 48 + rlen + slen]
   assert off + 48 + rlen + slen == len(raw)
   assert hashlib.sha256(scratch).digest() == digest
   ```

   For PK19 the hash covers the *unpacked* bytes; for RS20 it covers the *scratch* bytes. Offset questions took the tag offset, size questions `file_length - offset`, and `hcfg_resource_size` was the u32 at +8 (not the HCFG payload length).
7. **Submission.** All 240 fields went in one call and came back CORRECT (1 call used).

## Time Trial and Mastery

**Time Trial** was the same 240 IDs against a fresh archive with different specimen variants for d01 and several other days (d01 was a stripped variant, so `recover_code` became `none`, the disassembly `[]` and `has_symtab` false; d02, d08 and d16 were unchanged). The Single Solve mapper was rerun unchanged on the new bytes with no per-ID overrides carried over. d12's storage class was re-checked by hand. The untimed attempt went CORRECT in one call. Two later ranked reruns (80 s window) were also CORRECT in one call each (7.9 s and 10.0 s end to end, download included).

**Mastery** gave five deliveries of 240 IDs. The mapper ran once per set with an `ANSWER_PREFIX=setN.` and the results were merged. The new wrinkle was that d02, d04 and d10 were PE files in sets 1, 3 and 5 (d02 MinGW C++, d04 Go, d10 Zig), where Single Solve and Time Trial had ELF. The mapper needed a COFF symbol reader (symbol count zero meaning no symbols) and resolution of `/<decimal>` PE section names through the COFF string table. d02's PE symbol answers kept the `.text$` and `.rdata$` prefixes. As a regression check, the Single Solve and Time Trial candidate JSON were regenerated after the patch and came out byte-identical. All 1,200 fields were accepted in one call. A ranked rerun (325 s window) was CORRECT in one call, 18.1 s total.

## Dead ends and gotchas

- **Overlay confusion on MinGW PE.** The bytes after the last section of the MinGW `d02.exe` are the COFF symbol and string tables, not a payload. The Go PE has a section named `.symtab` but no COFF symbols; check `NumberOfSymbols`, not the section name. (Both from the Mastery journal and the library gotchas.)
- **Zig `.relro_padding` is NOBITS and overlaps the `.hcfg` file range.** Skip NOBITS sections when mapping a file offset to a section (library note, Day 20).
- **Base64 carrier offsets.** For a Base64-carried record the offset question wanted the first Base64 character of the winning run, not the decoded record's offset of 0 (Day 17/19 journals; not re-exercised as a rejection here).
- **d03 ships two files.** The `.dll` is the specimen; the runtimeconfig JSON is delivery support.
- **The RS20 record is also visible to a plain literal scan** (at tag + 48), so a scan alone gives the right value, but the rubric text still asks for the bounds and hash checks. Conversely, PK19's payload is transformed, so a literal scan finds nothing and "no HCFG" would be a wrong conclusion.
- No submission was rejected in any mode; there was no validator guesswork to record.

## Takeaways

- **Cumulative recap bundles pay for validated mappers.** Day 15's graded mapper covered 150 of 240 IDs on sight; only the 90 new fields needed fresh logic, and diffing recomputed old fields against the previously accepted values was the cheap safety check. The same discipline (regenerate earlier modes after a patch, require byte-identical output) caught nothing here but is why the Mastery patch was safe to submit in one call.
- **Tagged overlay procedure:** `rfind` the tag, read the length fields, require that the layout ends exactly at EOF, verify the embedded SHA-256 against the right region, then re-run the HCFG scan on the decoded bytes. An exact EOF match is a cheap proof that the layout guess is right.
- **Static transform, no execution.** The PK19 byte transform (`ror8(b ^ k2, 3) ^ k1`) is given in the question and collapses to a single rotate and XOR; nothing needs emulation or an unpacker.
- **Manifest-driven, not name-driven.** Mastery's PE variants showed that a mapper tied to ELF reading breaks on the same day number in another container; classify by magic.

## Evidence

Leaf base: `challenges/day20/binary-reverse-engineering/binary-reverse-engineering-day-20-resources-and-overlays/`

- `single-solve/journal.md`, `single-solve/start_response.json`, `single-solve/artifacts/answer_map.py` (overlay readers `overlay_pk19` / `overlay_rs20`, `storage_class`), `single-solve/artifacts/submission-attempt-log.json` (one call, CORRECT; no saved `result.json`, hence "journal-recorded")
- `time-trial/journal.md`, `time-trial/start_response.json`, `time-trial/ranked-1/result.json`, `time-trial/ranked-2/result.json`
- `mastery/journal.md`, `mastery/start_response.json`, `mastery/artifacts/answer_map.py`, `mastery/ranked-1/result.json`
- Library: BRE-OVERLAY-B103 (overlay and packer triage), BRE-OVERLAY-TAGGED-001, BRE-HCFG-STORAGE-001, BRE-GOTCHAS-B106
- `progress.md` (grep `^| 20 .*resources-and-overlays`)
- Day 20 has no librarian note or lane log for this challenge (its `_run/` folder only holds the release list and watcher log).
