# Jack in the Box

| | |
|---|---|
| Day / Lane | 19 / Binary Reverse Engineering |
| Challenge ID | `binary-reverse-engineering-day-19-static-unpacking` |
| Technique | static unpacking of a trailing XOR/ROR overlay (`PK19`), plus a cumulative Day 1-19 recap bundle |
| Single Solve | Solved (journal-recorded) |
| Time Trial | Independent |
| Mastery | Independent |

## Summary

Day 19's one new idea was a packed overlay appended to an ELF: no literal `HCFG` record in the
file, just a `PK19` trailer that unpacks with a byte-wise XOR/rotate and carries its own SHA-256.
The rest of the 228 answer fields were a recap of Days 1-18, so most of the work was reusing a
validator-confirmed Day 15 mapper and extending it. All three modes went CORRECT on the first
submission, the full answer set in one call each.

## The challenge

Prompt: "There's a payload hiding inside this binary, and a little decoder stub wound up to pop it
out. Find the stub, watch it work, and catch what's inside - statically."

The Single Solve delivered one AES-encrypted ZIP (`infected`, 32.8 MB) holding 22 member files:
one `dNN--variant-000` binary per day from d01 to d19 (d03 is a .NET DLL plus its
`runtimeconfig.json`, d06/d07 are JARs, d15 has stripped and symbolized ELFs, d16 has compact and
direct WASM). There were 228 answer fields: 12 per day, 19 days. Formats were mixed (84 pattern,
78 exact_text, 29 enum, 29 integer, 8 json); every one had to be submitted as a string.

Only d19 was new. Its twelve fields were the usual artifact SHA-256, architecture/machine/format
labels, and then the overlay fields: overlay offset, overlay size, the HCFG version byte and the
unpacked configuration (`campaign`, `domain`, `mutex`, `url`, CRC32). The question text spelled out
the whole unpacking recipe, so the challenge was less "find the algorithm" than "implement it
exactly and verify it".

## Solution

1. **Extract.** The ZIP is AES-encrypted, so stdlib `zipfile` is not enough; we used `pyzipper`
   and verified the download against the SHA-256 in the start response before analysis.
2. **Find the overlay.** The d19 member is an ELF in which a plain scan for `HCFG` finds nothing.
   The last occurrence of `PK19` (`rfind`, not `find`) sits at offset 10808, giving an overlay of
   424 bytes (file length minus offset; not the packed length).
3. **Read the header.** Relative to the magic: byte +5 is `k1`, +6 is `k2`, +8 is a u32le packed
   length, +12 is the 32-byte SHA-256 of the plaintext, and the packed bytes start at +44.
4. **Unpack.** Each byte becomes `ror8(b ^ k2, 3) ^ k1`:

   ```python
   def ror8(b, n):
       return ((b >> n) | (b << (8 - n))) & 0xFF

   off = raw.rfind(b"PK19")
   k1, k2 = raw[off + 5], raw[off + 6]
   plen = struct.unpack_from("<I", raw, off + 8)[0]
   digest = raw[off + 12 : off + 44]
   packed = raw[off + 44 : off + 44 + plen]
   plain = bytes(ror8(b ^ k2, 3) ^ k1 for b in packed)
   assert hashlib.sha256(plain).digest() == digest
   ```

   The embedded SHA-256 matched, which is what lets you trust the plaintext without ever running
   the stub. The result was 380 bytes starting with a version-1 HCFG record.
5. **Decode the record.** HCFG layout: magic, version byte, a one-byte XOR key (different from
   `k1`/`k2`), u16 payload length, CRC32 of the decoded payload, then the payload. XOR the payload,
   check the CRC, parse the JSON. The CRC validated and the `campaign`, `domain`, `mutex` and `url`
   values came straight out of it (Single Solve: `BR19-000`, `node-000.day19.example.invalid`,
   `Global\HuntressBR19_000`).
6. **The recap blocks.** d01-d18 re-asked fields from earlier days (ELF headers, symbol tables,
   section names, Go/Rust predicates, JAR methods and access flags, Base64 offsets, HCFG
   versions and keys). We took the Day 15 mapper, which had already been validator-confirmed on 150
   fields, re-implemented it without `objdump`/`readelf` (pure-Python ELF, PE/COFF and classfile
   parsers; `capstone` only for the d12 cross-reference check) and added the new fields.
7. **Submit.** One call, all 228 fields, CORRECT.

## Time Trial and Mastery

- **Time Trial** was the same 228 field IDs against a fresh archive (32.8 MB). The d01 member was a
  stripped ELF this time (`.symtab` absent): symbols `[]`, disassembly `[]`, `recover_code`
  `none`. The unchanged Single Solve mapper, diffed against its earlier output, differed only in
  artifact-derived values, and the one-call submission was CORRECT.
- **Mastery** delivered five archives (`artifacts[]` in the start response; the stock `gm.py
  artifact` fetches only the first, so we fetched each from its capability link) with 1,140
  fields: five sets of 228, IDs prefixed `set1.` to `set5.`.
  What changed: in sets 1, 3 and 5, d02, d04 and d10 were **PE32+ executables** instead of ELF.
  The mapper needed pure-Python PE support: section names of the form `/<decimal>` resolved
  through the COFF string table, primary COFF symbols read while skipping auxiliary records, and
  section-definition symbols kept so that the first match for a symbol-score question keeps its
  `.text$...`/`.rdata$...` prefix. The HCFG storage rule was generalized to an ordered check:
  `base64`, then `raw_hcfg_section` (record at the start of `.hcfg`), then `raw_const_array`
  (an instruction references the record's address), then `raw`. d12 came out as `raw_const_array`
  in every set. The d19 overlay offset (10808), size (424) and unpack were identical across the
  five sets. The whole map went in one call, CORRECT.
- We also ran a separate ranked Time Trial and Mastery attempt afterwards (80 s and 325 s windows;
  8.8 s and 18.4 s end to end, one submission call each). Those are speed-scoreboard reruns, not
  the original solves.

## Dead ends and gotchas

- **No literal config.** A plain `HCFG` scan of d19 returns nothing; the record only exists after
  unpacking. Without the prompt's recipe, the tell would be `rfind(b"PK19")` landing near the end
  of the file.
- **Use the last `PK19`.** Earlier byte matches can occur in code or data; the overlay is the final
  occurrence.
- **Overlay size** is file length minus the magic's offset, not the packed length at +8 (which
  excludes the 44-byte header).
- **Stripped variants.** The Time Trial d01 member had no `.symtab`, so the symbol list and
  disassembly fields become empty lists and `recover_code` becomes `none`; a mapper that assumes
  `recover_code` always exists would fail there.
- **Accepted label spellings** (validator behavior, per the journals): `x86-64` for architecture and
  machine, including PE AMD64 in d10; lowercase `elf`/`pe`/`wasm`/`jar` for the enum
  "classify final bytes" questions; `ELF64`/`PE32+` for the "ELF or PE" pattern questions;
  objdump-style `["mov eax,0x1fa4","ret"]` for the d01 disassembly; the HCFG storage enum values
  named above. Because the first submission of each mode was CORRECT, we never learned which
  alternates would have been rejected.
- Host quirk: `python -I` hides user-site packages such as `pyzipper`, and ignores `PYTHONUTF8`.
  (A tooling note, not a challenge trap.)

## Takeaways

- **Cumulative recap bundles reward reuse.** A day that adds twelve fields to eighteen days of old
  ones is mostly a regression test of your last validator-confirmed mapper. Start from it, diff
  per-ID output against the previous mode, and avoid per-ID overrides so the same file transfers to
  Time Trial and Mastery.
- **Static unpacking is just reimplementation plus an integrity check.** When the packer ships a
  hash of its plaintext, verify it before parsing; then the downstream CRC is a second confirmation.
  The overlay recipe (final magic, two key bytes, length, SHA-256, packed bytes) is worth keeping
  as a reusable snippet: library entry BRE-OVERLAY-PK19-001 and its generalization.
- **Mastery varies container format, not just content.** Expect member kinds (ELF vs PE) to change
  per set, so classify by magic bytes and keep parsers for both.

## Evidence

Leaf base: `challenges/day19/binary-reverse-engineering/binary-reverse-engineering-day-19-static-unpacking/`

- `single-solve/journal.md`, `single-solve/start_response.json`,
  `single-solve/artifacts/answer_map.py` (`unpack_overlay`),
  `single-solve/artifacts/submission-attempt-log.json` (1 call, CORRECT; no `result.json` saved)
- `time-trial/journal.md`, `time-trial/artifacts/submission-attempt-log.json`,
  `time-trial/ranked-1/result.json`
- `mastery/journal.md`, `mastery/start_response.json`, `mastery/artifacts/answer_map.py`,
  `mastery/artifacts/notes.txt`, `mastery/ranked-1/result.json`
- Library: `library/binary-reverse-engineering/pk19-overlay-unpack.md` (BRE-OVERLAY-PK19-001),
  `recap-bundle-d19-validation.md` (BRE-RECAP-D19-001)
- `progress.md` (grep `^| 19 .*static-unpacking`)
