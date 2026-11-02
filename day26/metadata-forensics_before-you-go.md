# Before You Go

| | |
|---|---|
| Day / Lane | 26 / Metadata Forensics |
| Challenge ID | `metadata-forensics-day-26-before-you-go` |
| Technique | Windows Prefetch header parsing, including Win10 `MAM` (XPRESS-Huffman) compressed `.pf` files |
| Single Solve | Independent |
| Time Trial | Independent |
| Mastery | Independent |

## Summary

Day 26's new metadata field was `prefetch.executable_name`. The Single Solve asked only that,
for one `trace-001.pf`, and the file was a Windows 10 compressed Prefetch (`MAM\x04`), so the
header name could not be read until the payload was decompressed with MS-XCA "LZ77+Huffman". We
wrote a pure-Python decoder, checked it byte-for-byte against Windows' own `ntdll`, and read the
UTF-16 name at offset 16. The stored upper-case spelling was accepted on the first call in every
mode. Time Trial and Mastery re-asked the same field as one of 31 per-set recap questions, and
there the `.pf` files were plain `SCCA`.

## The challenge

Prompt: "Prefetch evidence records what Windows prepared to execute even after filenames changed.
Recover executable identities and connect execution traces to the correct hosts."

- **Single Solve:** one ZIP (`metadata-day-26-evidence.zip`), one question,
  `d26-prefetch-001-prefetch-executable-name` (`exact_text`: "exact artifact-derived value only";
  the literal `none` only if the field is absent). No deadline.
- **Time Trial:** one `evidence.zip`, 31 questions, a 50 s ranked window (the untimed regular
  attempt had no deadline).
- **Mastery:** one bundle of 10 evidence sets, 31 questions each (310 answer ids, prefixed
  `setN.`), a 325 s ranked window.

The 31 Time Trial questions are a recap of the whole month's metadata curriculum: filename, size,
MD5/SHA-1/SHA-256 and magic-prefix questions about the roster and evidence-note files, then one
field per earlier day's specimen (JPEG SOF dimensions, PNG IHDR, PDF header version, ZIP EOCD,
PE/.NET/MSI/LNK/OOXML fields, email Message-ID, WAV/MP4 media fields, ELF/Mach-O, APK, JAR,
SQLite, EVTX record count, registry hive root name), then the new Day 26 prefetch name, and a
final `synthesis-sha256-order` JSON question. Only the prefetch field was new that day; the
earlier ones were already handled by the shared bundle reader.

## Solution

1. **Recon.** The bundle held `trace-001.pf` among its members. The question names the field
   (`prefetch.executable_name`), which maps to the 60-byte UTF-16-LE name in the Prefetch header.
2. **The twist.** `trace-001.pf` did not start with a version word and `SCCA`. It started with
   `MAM\x04`: a Windows 10/11 compressed Prefetch. Bytes 4-7 are the uncompressed size (15,412
   here; the file was 3,896 bytes), and the payload from offset 8 is MS-XCA "LZ77+Huffman"
   (`COMPRESSION_FORMAT_XPRESS_HUFF`). Our existing Prefetch reader only identified the
   container, so this was the gap.
3. **Decoder.** We added a standard-library XPRESS-Huffman decoder to the shared metadata reader.
   The format: output in 64 KiB chunks, each led by a 256-byte table of 512 nibble code lengths;
   a canonical Huffman code over a stream of little-endian 16-bit words consumed MSB first;
   symbols of 256 and above are (length, offset-bits) matches with extra length bytes inline.
4. **Verification before submitting.** Because the decoder had only one real specimen, we
   compared its output with Windows' `RtlDecompressBufferEx` through `ntdll` on the same file.
   The bytes were identical and the decompressed header read `SCCA`. The file-metrics list inside
   the decompressed file also showed `\WINDOWS\SYSTEM32\CMD.EXE`, agreeing with the header name.
5. **Reading the name.** After decompression, the executable name is the NUL-terminated UTF-16-LE
   field at offset 16:

   ```python
   def read_prefetch(d):
       compressed = d[:4] == b"MAM\x04"
       if compressed:
           d = xpress_huffman_decompress(d[8:], u32(d, 4))
       return {"signature": d[4:8].decode("latin-1", "replace"),
               "executable_name": d[16:76].decode("utf-16-le", "replace").split("\0")[0],
               "compressed": compressed}
   ```

6. **Submission.** The answer was the name exactly as stored in the header, upper case with its
   extension: `CMD.EXE`. It was CORRECT on the first call. The solver had also queued the lower
   case spelling (`cmd.exe`) and the bare stem (`CMD`) as alternates; neither was needed.

## Time Trial and Mastery

- The compressed container did not recur. The two Time Trial bundles and the 20 Mastery sets we
  saw (two Mastery deliveries of 10 sets) all carried `trace-001.pf` as a plain version-30
  `SCCA` file of about 15.3 KB. So the same challenge mixed both containers; the reader has to
  dispatch on the magic bytes, not on the extension or the day.
- The Time Trial and Mastery answers came from the general bundle reader that already covered
  the earlier days' fields, plus the new prefetch field. The ranked Time Trial was CORRECT in two
  submission calls for the regular attempt and one for the ranked attempt; the ranked Mastery
  (310 answers) was CORRECT in one call. Locally these took 3.9 s (Time Trial, 50 s window) and
  4.9 s (Mastery, 325 s window).
- Mastery needed no prefetch-specific handling beyond the shared reader. We did not determine
  the cause of the regular Time Trial needing two calls beyond what the lane log shows: the first
  call submitted only the prefetch-name group and returned `accepted` with 30 remaining, then the
  other 30 fields were accepted together as CORRECT.

## Dead ends and gotchas

- **Compressed Prefetch is not obvious from the extension.** A `.pf` that fails the `SCCA`
  signature check is not necessarily corrupt; check for `MAM` first. Reading offset 16 of the raw
  file yields garbage.
- **A one-field solver is a trap for the larger modes.** The first merged Time Trial candidate
  came from a solver that only knew the prefetch field and wrote that one value into all 31 ids;
  it was rejected by our own format lint before anything was submitted. The fix was to answer the
  full bundle with the shared reader and add the alternate spellings as extra variant groups. This
  was a tooling issue rather than a challenge trap (see the day overview).
- **Spelling.** The format hint says "exact artifact-derived value", and the stored header form
  (upper case, `.EXE`) was accepted in every mode. The name in the header is not necessarily the
  name of the `.pf` file.

## Takeaways

- Technique to search for: **Windows Prefetch `MAM` compression / MS-XCA XPRESS-Huffman (LZ77+Huffman)**.
  Win10+ prefetch files are usually stored compressed; many forensic tools decompress them via
  `ntdll`, which is unavailable off Windows, so a pure-Python decoder is useful.
- When writing a decoder from a spec with a single specimen, validate it against the platform's
  own implementation if you have one (here `RtlDecompressBufferEx`) before trusting it on the
  graded answer.
- Always dispatch on magic bytes. The same challenge gave one compressed file in Single Solve and
  plain files in every other mode.
- Only the executable-name field was ever asked. Run count, last-run times, volumes and file
  metrics were not required, and our reader does not parse them.
- Limits: only one real `MAM` specimen was seen. Multi-chunk payloads (over 64 KiB of output),
  Win8 `MAM\x03` files and older Prefetch versions (17/23/26) are untested.

## Evidence

Base path: `challenges/day26/metadata-forensics/metadata-forensics-day-26-before-you-go/`

- `single-solve/journal.md`, `single-solve/start_response.json`, `single-solve/accepted_plan.json`
- `single-solve/artifacts/xcheck_ntdll.py` (ntdll cross-check of the decoder)
- `time-trial/regular/result.json`, `time-trial/ranked-1/result.json`
- `mastery/regular/result.json`, `mastery/ranked-1/result.json`
- Day-level (under `challenges/day26/_run/`): `librarian/metadata-forensics-day-26-before-you-go.md`,
  `lane-metadata-forensics-day-26-before-you-go.log`
- Library: META-PREFETCH-MAM-001 (compressed Prefetch), `metaread.read_prefetch` and
  `question_facts.py` field `prefetch.executable_name`
- `progress.md` (grep `^| 26 .*before-you-go`)
