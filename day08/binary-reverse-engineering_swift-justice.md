# Swift Justice

| | |
|---|---|
| Day / Lane | 8 / Binary Reverse Engineering |
| Challenge ID | `binary-reverse-engineering-day-08-swift-network-configuration` |
| Technique | Cumulative recap bundle; CRC-validated HCFG config record hidden in a Base64 run inside an ELF; ELF/PE format mix |
| Single Solve | Independent (graded CORRECT response saved in the leaf; solved on the third POST) |
| Time Trial | Independent (original attempt: CORRECT on the 12th submission call, journal entry only; ranked-2, -3, -4, -5 and -7 `result.json` files are the saved graded results) |
| Mastery | Independent (original attempt: CORRECT on the first call, journal entry only; ranked-1 to -4 and -6 `result.json` files are the saved graded results) |

## Summary

Day 8's binary challenge was not one binary but a recap: eight specimens (d01-d08, one per earlier
day plus a new Swift ELF), each asked about again with two extra fields. The new part was d08, whose
configuration record is not stored as a literal `HCFG` blob but inside a Base64 run, next to a
CRC-invalid literal decoy. All 80 fields were derived statically from the delivered bytes. Time Trial
exposed a real mapper gap (a native PE where the same slot had been an ELF), fixed in the middle of the
run.

## The challenge

Prompt: "Swift binaries speak in protocols and hide their networking in plain sight. Follow the
symbols to the URL configuration and see where this one phones home." The title and prompt describe
only d08; the 80 questions cover d01-d08, ten per day.

- **Single Solve:** one AES-encrypted `evidence.zip` (31,454,414 bytes, password `infected`) with nine
  members: d01 ELF, d02 ELF, d03 .NET DLL plus `runtimeconfig.json`, d04 ELF, d05 ELF, d06 JAR,
  d07 JAR, d08 ELF. 80 answer IDs, untimed.
- **Time Trial:** same 80 IDs, but the member bytes differ from Single Solve for most days, and the
  window was about 80 s (`started_at` to `deadline` in the saved start response).
- **Mastery:** five separate `evidence.zip` deliveries, IDs prefixed `set1.` to `set5.`, 400 answers.
  `start` listed them in `artifacts[]`; fetching only the first delivery is a trap.

Field kinds per day: SHA-256 of member 1 (and member 2 for d03), container/format and machine labels,
the four HCFG strings (`campaign`, `domain`, `mutex`, `url`), the record's CRC32 and XOR key, the record
version, ZIP compression methods for the JARs, d01 ELF header facts and symbol inventory, and two C++
predicates on d02 (`has_cpp_typeinfo`, `has_cpp_vtable`).

## Solution

1. **Recon.** The d01-d06 members were byte-identical to the previous binary challenge's Single Solve
   (Day 6), so after matching SHA-256 for each member the earlier accepted values for those fields were
   carried over. Everything new was derived from bytes: d01 entry point and `.symtab` presence, d02 symbol
   predicates, d03 URL/CRC, d04/d05 version and key, d06 key and compression methods, and all of d07 and d08.
2. **HCFG record format.** Every configuration is `HCFG`, version byte, XOR key byte, u16 length, u32
   CRC32, then a payload XORed with the key that decodes to JSON. Only a CRC-valid record counts, because
   decoys exist (d05 carries earlier marker-shaped ones, d08 carries one).
3. **The d08 trick.** The question text spelled out the carrier rule: test literal `HCFG` occurrences
   first, then every non-overlapping Latin-1 run matching `[A-Za-z0-9+/]{100,}={0,2}` after Base64 decoding,
   and accept the first CRC32-valid record. The ELF had exactly one literal `HCFG`, and it failed CRC. Out of
   5,484 qualifying Base64 runs, exactly one decoded to a valid record, starting at its first decoded byte,
   with XOR key `0x41`, payload length 368 and CRC32 `0x6a033bfd`.
4. **Scanner.** The extractor is a plain parser plus two passes over the file:

   ```python
   BASE64_RUN = re.compile(rb"[A-Za-z0-9+/]{100,}={0,2}")
   # in parse_hcfg(): XOR payload, then require the checksum before trusting anything
   payload = bytes(v ^ key for v in record[12:end])
   if zlib.crc32(payload) & 0xFFFFFFFF != expected_crc:
       return None
   ```

   Literal hits are listed first, then Base64-run hits, and the first record in that order is the answer.
5. **d02 predicates.** `readelf -sW` on the C++ specimen showed `_ZTI15DiscoveryEngine` and
   `_ZTV15DiscoveryEngine`, so both `has_cpp_typeinfo` and `has_cpp_vtable` were `true`.
6. **Submission.** Every value had to be a JSON string, including `"64"` for the ELF class, `"1"` for the HCFG
   version and the compression-method arrays as compact JSON text (`[0,8]`). The first call sent the ten d08
   fields and came back `accepted` with 70 remaining. The next 70-field call was rejected at validation
   (`invalid_answers`) because values were not all strings, which graded nothing. The third call sent the 70
   remaining fields as strings and returned CORRECT.

d08 answers, for reference: campaign `BR08-000`, domain `node-000.day08.example.invalid`, mutex
`Global\HuntressBR08_000`, URL `https://node-000.day08.example.invalid/api/BR08-000`, CRC32 `0x6a033bfd`,
format `ELF`, architecture and machine `x86-64`, class `64`.

## Time Trial and Mastery

**Time Trial.** About 80 s, with fresh bytes for d01 and d03 to d07; d08 matched Single Solve. The
mapper was therefore rebuilt to be byte-driven rather than reusing earlier answers: unzip with `pyzipper`,
run the library's ELF/PE/JAR inspector over the members, and map its output onto the 80 IDs.

The real problem was in one delivery whose **d02 specimen was a native MinGW PE (`PE32+`) instead of an
ELF**. The inspector parsed it correctly, but the mapper read the C++ typeinfo and vtable predicates only
from the ELF `symtab` key. On a PE the names live in the COFF symbol list under a different key, so both
predicates came out `false`. The full batch was `incorrect`; submitting by day prefix (`d01_` to `d08_`)
showed seven days `accepted` and only `d02_` rejected, which pinned the bug without single-field probes. The
fix was to fall back to the COFF symbols when `symtab` is absent. By then the window had expired, so a
fresh `retry` was needed; the corrected mapper then got all 80 fields CORRECT. Two attempts expired
along the way (one before the first submit landed, one during bisection), which is why the call count is 12.

**Mastery.** Five deliveries, each 80 fields, answered by one function applied per delivery and merged with
`setN.` prefixes. The format mix was confirmed by container detection, not file extension:

| set | d02 and d04 | d01, d05, d08 | d03 | d06, d07 |
|---|---|---|---|---|
| 1, 3, 5 | PE32+ | ELF64 | .NET PE32 | JAR |
| 2, 4 | ELF64 | ELF64 | .NET PE32 | JAR |

d02 and d04 switched container together; the other specimens never changed type. With the fix from Time
Trial already in the mapper, the single 400-field call was CORRECT. For a PE d04 the architecture label
that was accepted was `AMD64`, not `x86-64`.

**Later ranked reruns.** The leaf also holds ranked-attempt folders from a later re-run pass, using the
shared solver instead of the leaf mapper. Time Trial reruns 2, 3, 4, 5 and 7 and Mastery reruns 1 to 4 and 6
each record CORRECT in one submission call, with 5.8 to 10.4 s and 8.5 to 22.1 s total pass time. Time
Trial `ranked-1` holds only a start response with no result, so no graded outcome exists for it. Time
Trial `ranked-6` and Mastery `ranked-5` have no result either; their logs show the solver failing before
submitting because `pyzipper` was not importable in that run (see the day overview for tooling
mistakes).

## Dead ends and gotchas

- **Literal marker decoy.** A plain `grep HCFG` finds a record that fails CRC. Only the Base64-run
  scan, filtered by CRC32, yields the real one. The question text states this order; the decoy exists to
  punish scanning only the first hit.
- **ELF-only symbol code on a PE member.** The Time Trial failure above. The same mistake was independently
  made on Day 9 Mastery, and the earlier fix lived only in that leaf's mapper.
- **Strings only.** The 70-field batch with non-string values returned `invalid_answers`. Nothing was
  graded, but the call still counted.
- **Per-delivery labels.** d04 `architecture` is `x86-64` for an ELF and `AMD64` for a PE; d08 `format` was
  accepted as `ELF`; d01 endianness as `little endian`; d03 `PE32_CLR` and `x86`.
- **Attempts are not 1:1 with fixtures.** Time Trial and Mastery draw members from a variant pool, so
  an earlier accepted value is only safe for a member whose SHA-256 you have just compared.

## Takeaways

- For "recap" bundles, treat each day's block as independent and key everything off the delivered bytes
  and the container magic, not the file name or a previous mode's accepted map.
- HCFG-style records: parse by layout and keep a record only if the CRC32 matches; scan literal markers
  and Base64 runs, in the order the question states.
- On a bundle that can swap ELF and PE per slot, any symbol predicate must read both symbol-table forms.
- When a full batch comes back `incorrect`, bisect by block (`dNN_` prefix, then `setN.`) before single IDs;
  here that located the bug in 8 cheap calls.
- Look for these as techniques: HCFG record parsing, Base64-run carving, COFF vs ELF symbol tables, C++
  typeinfo/vtable mangled names (`_ZTI`, `_ZTV`).

## Evidence

Base path: `challenges/day08/binary-reverse-engineering/binary-reverse-engineering-day-08-swift-network-configuration/`

- `single-solve/journal.md`, `single-solve/start-response.json`, `single-solve/submission-response-03.json`
  (CORRECT), `single-solve/artifacts/extract_d08_facts.py`, `single-solve/artifacts/answer_map.py`,
  `single-solve/submission-response-02-validation-error.json`
- `time-trial/journal.md`, `time-trial/artifacts/answer_map.py`, `time-trial/ranked-{2,3,4,5,7}/result.json`
- `mastery/journal.md`, `mastery/artifacts/answer_map.py`, `mastery/ranked-{1,2,3,4,6}/result.json`
- Per-challenge solver for the rerun pass: `tools/ranked/solvers/binary-reverse-engineering-day-08-swift-network-configuration/solve.py`
- `progress.md` rows for day 8, challenge `binary-reverse-engineering-day-08-swift-network-configuration`
  (Single Solve, Time Trial and Mastery all `Solved`).
