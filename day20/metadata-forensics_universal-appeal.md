# Universal Appeal

| | |
|---|---|
| Day / Lane | 20 / Metadata Forensics |
| Challenge ID | `metadata-forensics-day-20-universal-appeal` |
| Technique | Mach-O header parsing (thin vs fat/universal), answer-format conventions in a multi-file evidence bundle |
| Single Solve | Solved (journal-recorded) |
| Time Trial | Independent |
| Mastery | Independent |

## Summary

The Single Solve is one question: list the CPU types in a Mach-O file. The file was a 56-byte thin
ARM64 binary, so the parsing took a minute. Most of the cost was the answer format, which the
prompt did not pin down: two guesses were rejected before the validator accepted a JSON array of
zero-padded 8-digit hex strings. Time Trial and Mastery re-used the previous day's evidence-bundle
template with this one extra field, and both were solved by extending the same extractor.

## The challenge

Prompt: "A macOS deployment includes thin and universal binaries mixed with archives. Enumerate
every CPU type and identify what truly supports the advertised fleet." The wording suggests fat
binaries whose advertised architectures might not match the slices they contain. None of the
specimens we saw was fat.

- **Single Solve:** a ZIP (`metadata-day-20-evidence.zip`) holding one 56-byte `macos-001.bin`, and
  one question, `d20-macho-001-macho-cpu-type-set`, with answer kind `json` ("compact valid JSON
  array with exact values and order").
- **Time Trial:** one evidence ZIP and 25 questions. Twenty-three were the previous day's generic
  bundle (roster and note file hashes, sizes, BOM, plus one field per earlier format: JPEG, PNG,
  PDF, ZIP, PE, .NET, MSI, LNK, OOXML, email, audio, video, ELF). The last two were
  `d20-macho-001-macho-cpu-type-set` and a synthesis question that asked for the full-file SHA-256
  of several named files, ordered by that digest.
- **Mastery:** a ZIP of five nested evidence ZIPs (`evidence-01..05.zip`), the same 25 questions
  per set, 125 answers in total, with `setN.`-prefixed IDs.

## Solution

1. **Identify the file.** The first four bytes were `cf fa ed fe`. That is `MH_MAGIC_64` stored
   little-endian, so a thin 64-bit Mach-O and not a fat container (`ca fe ba be`, always
   big-endian).
2. **Read cputype.** In a thin header `cputype` is a 32-bit integer at offset 4, in the byte
   order the magic implies. It read `0x0100000c`, which is `CPU_TYPE_ARM64`
   (`CPU_TYPE_ARM | CPU_ARCH_ABI64`).
3. **Make the reader general.** For a fat file the extractor reads the big-endian `nfat_arch`
   at offset 4 and takes `cputype` from each 20-byte (`cafebabe`) or 32-byte (`cafebabf`)
   `fat_arch` entry. It also opens each slice at its own offset and reads that header's `cputype`,
   to catch a fat table that advertises an architecture the slice does not carry. That
   matches the prompt's "what truly supports the fleet". The thin specimens never used this branch.

   ```python
   if magic in (bytes.fromhex("cafebabe"), bytes.fromhex("cafebabf")):
       is64 = magic[3] == 0xbf
       sz = 32 if is64 else 20
       n = struct.unpack_from(">I", d, 4)[0]
       for i in range(n):
           o = 8 + i * sz
           table.append("0x%08x" % struct.unpack_from(">I", d, o)[0])
           off = struct.unpack_from(">Q" if is64 else ">I", d, o + 8)[0]
           # ...then read the slice's own header at `off` and compare
   ```
4. **Format the answer.** The accepted value was the JSON array as a string:
   `["0x0100000c"]`. Lowercase hex, `0x` prefix, zero-padded to 8 digits, no spaces.

## Time Trial and Mastery

Time Trial added no new parsing beyond `macho_cpu_types()`. The extractor from the previous day's
bundle was extended with that function, and the mapper gained one line for the `d20-` prefix.
The whole 25-field map went in one call and was CORRECT on the first submission.

Mastery needed the same extractor run over five nested sets, and went CORRECT first time on the
original release-day attempt (all five Mach-O files were thin ARM64). One later detail:
that first Mastery attempt was not the one that finally counted. A second `start` on 2026-10-21
left an open, ungraded attempt on a new artifact, so we re-solved on a fresh variant on
2026-10-23. That variant differed from the first in ways that broke the extractor:

- Set 5's inner ZIP had 16 members instead of 17: no `evidence-note-copy.txt`, and no
  `evidence-note-copy-*` questions. The extractor read that file unconditionally and crashed.
  The fix was to guard the read on membership.
- The magic-prefix and BOM questions named a different file (`roster-magic-prefix-hex-16` rather
  than the `evidence-note-...` variant), so the mapper's hardcoded file name no longer matched.
  The fix was to compute prefix and BOM facts for every member and match the question ID by suffix.

All 125 answers then went in one call and were CORRECT. We later ran a ranked Time Trial and
Mastery pass on this challenge. Both were CORRECT on one call, each solved in under 3 seconds
locally. They were bragging-rights runs and not part of the prize ranking.

## Dead ends and gotchas

- **The answer format was a guess.** `answer_format` only said "compact valid JSON array". Both
  an array of architecture names (`["arm64"]`) and an array of decimal ints (`[16777228]`) came
  back `incorrect`; the padded hex array was CORRECT on the third try. The convention was
  already visible in the previous day's hex fields (the PE entry-point RVA was accepted
  zero-padded, for example), so trying it first would have saved two attempts.
- **A scripted format loop kept going after CORRECT.** A fourth submission (unpadded hex) was
  sent after the third had scored, and returned `already_solved`. It graded nothing and cost a
  ledger entry. This was a mistake in our own script; see the day overview.
- **Fat-table order was never tested here.** With one slice per specimen the array order could not
  be checked on Day 20. On Day 23 a later challenge rejected fat-table order and accepted the same
  values sorted ascending. If a fat Mach-O turns up, answer the sorted set.
- **`CAFEBABE` is ambiguous.** Java class files share that magic with fat Mach-O. The next
  dword tells them apart: a value below 45 is a fat arch count, a larger one is a Java major
  version. The extractor in the leaf does not handle this; the later toolkit reader does.
- The unscored Mastery attempt from the release-day rehearsal received 9 submissions to a variant that
  was never graded. That cost was a mistake in our own tooling, not a challenge trap.

## Takeaways

- **Mach-O cputype set.** Thin header: `cputype` at offset 4 in the byte order the magic gives.
  Fat header: big-endian table of 20- or 32-byte entries. Cross-check against each slice's own
  header, since the prompt hints that the table can lie. Answer the sorted array of `0x%08x`
  strings.
- **Search prior formats before guessing.** When a new question's format is vague, reuse the
  formatting convention that the same validator already accepted for neighbouring fields.
- **Bundle extractors should be manifest-driven.** Members come and go between variants and
  between sets of the same Mastery (an optional file, a different file named in a magic-prefix
  question). Guard reads on membership and resolve the target file from the question ID.

## Evidence

Leaf base: `challenges/day20/metadata-forensics/metadata-forensics-day-20-universal-appeal/`

- `single-solve/journal.md`, `single-solve/start_response.json`,
  `single-solve/artifacts/submission-attempt-log.json` (the three attempts and the wasted fourth;
  there is no saved result file for this mode)
- `time-trial/journal.md`, `time-trial/start_response.json`, `time-trial/artifacts/extract_facts.py`
  (`macho_cpu_types()`), `time-trial/artifacts/answer_map.py`, `time-trial/ranked-1/result.json`
- `mastery/journal.md` (including the 2026-10-23 re-solve), `mastery/start_response.json`,
  `mastery/artifacts/extract_facts.py`, `mastery/artifacts/answer_map.py`,
  `mastery/ranked-1/result.json`
- Library: Mach-O CPU type set (META-MACHO-CPUTYPE-001); evidence-bundle toolkit
  (META-BUNDLE-TOOLKIT-B101); metadata gotchas for Days 11-20
- `progress.md` (grep `^| 20 .*universal-appeal`)
