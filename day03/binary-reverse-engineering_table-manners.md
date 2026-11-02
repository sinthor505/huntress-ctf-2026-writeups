# Table Manners

| | |
|---|---|
| Day / Lane | 3 / Binary Reverse Engineering |
| Challenge ID | `binary-reverse-engineering-day-03-managed-configuration` |
| Technique | .NET PE/CLR metadata triage; XOR-encoded `HCFG` config record carried as a static-array blob; ELF header/section/symbol facts |
| Single Solve | Independent (`CORRECT`, 24 fields, second submission) |
| Time Trial | Independent (`CORRECT`; plus a ranked rerun, `CORRECT` in 1 call, 1.38 s end to end) |
| Mastery | Independent (`CORRECT`, 2 delivery sets, 48 fields) |

## Summary

The prompt is about a .NET DLL that hides its configuration, but the question list is a recap of the first three
days of the binary lane: ELF header facts for Day 1, an XOR-encoded config record in a Day 2 native specimen, and
the managed DLL's PE/CLR facts plus its config for Day 3. All three modes were solved offline from the delivered
bytes with one static extractor; the only real friction was answer formatting.

## The challenge

Prompt: "A .NET binary is a database wearing a trench coat. Read the CLI metadata tables - methods, strings,
signatures - and find where the configuration is stashed." It suggests `ilspycmd` and `strings`.

Each mode delivered an AES-encrypted `evidence.zip` (password `infected`, the same as other lanes). Single Solve's
archive held four members: `d01--variant-000.elf`, `d02--variant-000.elf`, `d03--variant-000.dll` and
`d03--variant-000.runtimeconfig.json`. Single Solve asked for 24 string answers:

- **Day 1 ELF (8):** SHA-256, ELF class, machine, section names in header order, `.symtab` names, disassembly of
  `recover_code`, `e_type`, endianness.
- **Day 2 specimen (8):** SHA-256, container format, and the config fields campaign, domain, mutex, URL, CRC32 and
  XOR key.
- **Day 3 DLL (8):** SHA-256 of the DLL and of the runtimeconfig, compound format, machine, metadata version, and
  config campaign, domain and mutex.

## Solution

1. **Decrypt and hash.** The archive opens with AES (WinZip AE-2) support, so `pyzipper` rather than stdlib
   `zipfile`. Member SHA-256 values answer the `*_artifact_sha256_*` fields.
2. **Day 1 ELF.** `readelf` facts: ELF64, little-endian, `ET_EXEC`, x86-64, 28 sections with a `.huntress` section
   between `.rodata` and `.eh_frame_hdr`. `recover_code` is a 6-byte global FUNC, `b8 a4 1f 00 00 c3`, i.e.
   `mov $0x1fa4,%eax; ret`.
3. **Day 2 config.** The specimen carries an `HCFG` record: 4-byte magic, version byte, XOR key byte, 16-bit LE
   payload length, 32-bit LE CRC32, then the payload. XORing the payload with the key yields JSON, and the CRC32 of
   the *plaintext* equals the stored value, which is a free correctness check. Here the key was `0x31`, length 367,
   CRC `0x6149aa00`; campaign `BR02-000`, mutex `Global\HuntressBR02_000`.
4. **Day 3 DLL.** The PE is PE32 with COFF Machine `0x014c` (I386). The CLI metadata root (`BSJB`) carries the
   version string `v4.0.30319`, which is a different fact from the target framework (`.NET 8.0`) that `strings`
   also reveals. The same `HCFG` record (key `0x41`, length 367, CRC `0xcb6cf23f`) sits at file offset 6296,
   decoding to campaign `BR03-000` and mutex `Global\HuntressBR03_000`. The container answer is the compound
   `PE32_CLR`.
5. **Submit.** The extractor (`solve_evidence.py --answer-map`) prints one JSON map. Its decode core is small:

```python
version, key = data[start + 4], data[start + 5]
length       = struct.unpack_from("<H", data, start + 6)[0]
expected_crc = struct.unpack_from("<I", data, start + 8)[0]
plain = bytes(v ^ key for v in data[start + 12:start + 12 + length])
assert zlib.crc32(plain) & 0xffffffff == expected_crc
config = json.loads(plain)
```

Single Solve answers (all `CORRECT` together): Day 1 disassembly `["mov $0x1fa4,%eax","ret"]`; Day 2 campaign
`BR02-000`, domain `node-000.day02.example.invalid`, URL `https://node-000.day02.example.invalid/api/BR02-000`,
XOR key `0x31`; Day 3 format `PE32_CLR`, machine `I386`, metadata version `v4.0.30319`, campaign `BR03-000`,
domain `node-000.day03.example.invalid`.

The extractor locates the record with a plain `data.find(b"HCFG")`. It was only afterward, while promoting the
work to the library, that the "where is it stashed" question was answered properly: on the DLL the record is the
backing bytes of a compiler-generated `<PrivateImplementationDetails>` static array (a `FieldRVA` entry, 379 bytes
= 12-byte header + 367-byte payload), loaded at run time through `RuntimeHelpers.InitializeArray`. It is not in
`#Strings`, `#US` or `#Blob`, so searching the string heaps finds nothing. The literal scan and the table-driven
route agree.

## Time Trial and Mastery

- **Time Trial** had a 50 s server window and the same 24 questions, with fresh fixtures (a different Day 3 DLL
  variant). The unchanged extractor produced all 24 values and the first submission was accepted. A later ranked
  rerun used a per-challenge solver and went `CORRECT` on its first call (about 1.4 s from issue to result).
- **Mastery** gave an 80 s window and two deliveries of the same archive shape, answered as 48 fields with `set1.`
  and `set2.` prefixes: run the extractor once per archive and prefix the keys. The deliveries differed from
  Single Solve and from each other: set 1 had a different Day 3 DLL (campaign `BR03-001`) and an ELF Day 2
  specimen; set 2 had a PE32+ Day 2 specimen (answer `PE32+`) and the original Day 3 DLL. The Mastery start
  reported `unique_attempts_remaining: 0`, so there was no second try on this ID.
- The `d03` runtimeconfig hash was identical across modes and sets, while the DLL hash varied.

## Dead ends and gotchas

- **Native types are rejected.** The first Single Solve submission returned `invalid_answers` because JSON-array
  answers (section names, symbols, disassembly) went in as arrays. Every value must be a string, with arrays as
  compact JSON text. The corrected 24-field submission was `CORRECT`.
- **`d01_analysis_symbols` was ambiguous.** The prompt did not say which symbols. The journal carried both the
  full `.symtab` name list (including FILE entries and duplicates, in table order) and a narrower custom-symbol
  reading; the full list was submitted and accepted.
- **Disassembly spelling.** The accepted answer was AT&T syntax from `objdump` (`mov $0x1fa4,%eax`), not Intel.
  Whether the Intel form would also have passed was not tested.
- **Machine and format spellings.** `d03_analysis_machine` took `I386`; a deep review noted it belongs to a
  syntax family that also accepted `i386`, `x86`, `Intel 80386` and `0x014c`. `d03_analysis_format` needs the
  managed half (`PE32_CLR`); a bare `PE32` is the wrong shape (a related rejection happened on a later day).
- **Backslash in the mutex.** The JSON form is `Global\\HuntressBR03_000`; the actual value, and the answer, has
  one backslash.
- **Metadata version vs framework.** `v4.0.30319` (CLI metadata root) is not the `.NET 8.0` target framework.
- **Saved archives can be corrupted.** The local copies of these archives later turned out to be damaged by
  line-ending translation; the offline regression rebuilt members by SHA-256 from intact later archives.

## Takeaways

- A "metadata" prompt can be a cover for a cumulative recap: read the question IDs (`d01_`, `d02_`, `d03_`) before
  deciding what to analyze.
- The recurring config container in this lane is `HCFG` (magic, version, XOR key, length, CRC32, XOR-ed JSON);
  the CRC over the plaintext tells you immediately whether the key and offsets are right.
- For .NET, search the `FieldRVA` / `<PrivateImplementationDetails>` static-array blobs, not the string heaps, and
  keep metadata version and target framework apart.
- Serialize every answer as a string before the first submission, and run the same extractor per archive for
  multi-set Mastery.

## Evidence

Leaf: `challenges/day03/binary-reverse-engineering/binary-reverse-engineering-day-03-managed-configuration/`

- `single-solve/journal.md`, `single-solve/artifacts/solve_evidence.py` (the extractor), `single-solve/artifacts/README.md`,
  `single-solve/candidate-answers.json`, `single-solve/submission-response.json`
- `time-trial/journal.md`, `time-trial/submission-response.json`, `time-trial/ranked-1/result.json` (`CORRECT`, 1 call),
  `time-trial/ranked-1/accepted_plan.json`
- `mastery/journal.md`, `mastery/set1-answers.json`, `mastery/set2-answers.json`, `mastery/submission-response.json`
- Library technique entries: .NET metadata/config location, HCFG records, BRE field blocks for Days 1-10.
