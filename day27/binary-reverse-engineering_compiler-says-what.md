# Compiler Says What

| | |
|---|---|
| Day / Lane | 27 / Binary Reverse Engineering |
| Challenge ID | `binary-reverse-engineering-day-27-compiler-attribution` |
| Technique | cumulative recap bundle; two-member role pair; literal HCFG trailer (XOR + CRC32) read backward |
| Single Solve | Team solve (not ours) |
| Time Trial | Independent |
| Mastery | Independent |

## Summary

The title and prompt promised toolchain attribution ("name the toolchain"), but no question asked
for a compiler, a `.comment` string or a linker fingerprint. The day's new block was twelve fields
over a pair of ELF64 members named for `gcc` and `tcc`, plus a decoded config trailer. The other
312 fields were the recap of Days 1-26. A teammate scored the Single Solve within seconds of unlock,
so our submissions came back `already_solved` and nothing was graded for us. Time Trial and Mastery
were solved independently with the existing library reader, and each went CORRECT on the first
submission call.

## The challenge

Prompt: "Every compiler signs its work. Compare the runtime and linker fingerprints across builds and
name the toolchain - attribute this binary to its maker."

Delivery was one `evidence.zip` of binaries, with 324 answer ids per set. They split into 27 blocks
of 12 (`d01_...` through `d27_...`), one block per day of the event. The Day 26 block and earlier
were byte-identical to what Day 26 asked, including question text, member hashes and answer formats.
We checked this against Day 26's accepted plan, and every one of the 312 shared ids matched, so those
spellings were already known to be accepted. The new `d27_` block was:

- `d27_artifact_sha256_1/2`: SHA-256 of the two `d27--` members in ZIP central-directory order.
- `d27_analysis_{gcc,tcc}_format` (enum) and `d27_analysis_{gcc,tcc}_machine` (the ELF `e_machine` as
  an unsigned base-10 integer).
- `d27_analysis_config_{campaign,domain,mutex,url}`: decode both role members, require their config
  objects to be byte-for-byte equal, and answer the common value.
- `d27_analysis_gcc_config_{campaign,domain}`: the same record read from the `gcc.elf` member alone.

The config is an `HCFG` record: magic `HCFG`, version byte, one XOR key byte, little-endian payload
length, little-endian CRC32 of the decoded payload, then the XOR-encoded UTF-8 JSON. The new twist
in the wording was "scan backward from the end of the member for literal HCFG magic and accept the
first CRC32-valid decoded JSON record", which implies the file might carry decoy records earlier in
the bytes.

## Solution

1. **Triage.** The two members, `d27--sample-gcc.elf` and `d27--sample-tcc.elf`, were both ELF64
   x86-64 (`e_machine` 62), so format and machine were `elf` and `62` for both.
2. **Config trailer.** Each member held exactly one CRC-valid literal HCFG record near EOF (version
   1, key `0xc1`), and the two records were byte-identical. So "require both configs equal" had a
   trivial answer, and the per-member `gcc` fields were the same values. A CRC-valid record that
   also decodes to a JSON object is the test:

   ```python
   key = data[off + 5]
   n   = int.from_bytes(data[off + 6:off + 8], "little")
   want = int.from_bytes(data[off + 8:off + 12], "little")
   plain = bytes(b ^ key for b in data[off + 12:off + 12 + n])
   if zlib.crc32(plain) & 0xFFFFFFFF == want:   # then json.loads(plain)
       ...
   ```
3. **Answer assembly.** The library's `question_facts.py` resolved the ids from the question text. It
   needed two small extensions for this day: a role hint is taken from "basename ends `<role>.<ext>`"
   (so `gcc_format`, `tcc_machine`, `gcc_config_*` resolve), and "scan backward" orders the valid
   records by descending file offset. After these edits, 324 of 324 ids resolved, and the day's
   linter passed.
4. **Output.** All values went in as strings (`62` as `"62"`), and the `d27_` values were read from
   the fixture bytes, for example a campaign tag like `BR27-B72EA0BF`, a node hostname under
   `.day27.example.invalid`, a `Global\Huntress...` mutex and an `https://vault-.../hub/...` URL in the
   Time Trial map.

The Single Solve map came out as 324 answers with no alternate-spelling groups. Because our four
rounds of submissions (36 calls) all returned `already_solved`, that map was never validated by
the Single Solve itself.

## Time Trial and Mastery

- **Time Trial.** A plain `start` returned the team's CORRECT readout with no artifact, so a fresh
  `retry` was needed to get bytes to solve. The ranked attempt (80 s window) was CORRECT on call 1,
  12.79 s end to end locally (7.5 s of that was the solve). An untimed regular attempt was also CORRECT
  on call 1.
- **Mastery.** The bundle was 11 sets, each a full 324-id block (3,564 ids, set-prefixed as
  `setN.d01_...`), with 11 `evidence.zip` downloads. The unchanged reader solved all sets: 29.17 s
  locally against a 725 s ranked window, one submission call, and the regular attempt also went CORRECT
  on call 1. A regression of the final reader against the accepted plans of those four attempts gave
  7,776 of 7,776 ids exact (324 x 2 Time Trial, 3,564 x 2 Mastery).
- Variant groups for the backward-scan fields were prepared but never needed. Every sampled member
  carried one literal HCFG record, so the backward-first order and the "members disagree" alternates
  stayed unexercised.

## Dead ends and gotchas

- **The title was a red herring.** Nothing in the 324 ids asked about a compiler. There was no
  `.comment` section, `GCC:` banner or linker-version question; the gcc/tcc names were only role labels
  for the two members. We could not determine from the files whether the members were actually built by
  those compilers, because no question required it.
- **`already_solved` on the Single Solve is not a rejection.** It meant a teammate's attempt had
  already scored; the four rounds graded nothing, so no value was ever contradicted. We stopped
  treating it as evidence about the answers (see the day overview for how the automation handled
  that).
- **Team-status payloads carry no artifact.** Starting the Time Trial or Mastery returned the team's
  CORRECT status object with no `artifacts[]`, so a solver run against it failed for lack of files
  rather than for a wrong answer. A fresh `retry` fixed that.
- **Untested ordering rules.** The "first CRC-valid record scanning backward" rule matters only when
  several valid records exist. On the fixtures we saw it never did, so we cannot say which tie-break the
  validator uses.

## Takeaways

- Recap-bundle days are mostly regression: the 312 older ids needed no work because their reader and
  spellings were already validated, and only the new block had to be understood. Diffing the question
  text and expected values against the previous day's accepted plan up front is what made that safe.
- For HCFG-style trailers, validate by CRC32 plus a JSON-object parse rather than by finding the magic
  string, and scan with a rule that keeps CRC-invalid hits visible as decoys. Library entries to search:
  BRE-QFACTS-B107 (question-driven field reader), BRE-ROLEPAIR-B108 (role-member pairs, "both members
  equal" case) and BRE-INSPECT-B101 (literal HCFG scan).
- When a title promises a high-level judgement ("attribute the toolchain"), read the actual question ids
  before planning any deeper analysis.

## Evidence

Leaf: `challenges/day27/binary-reverse-engineering/binary-reverse-engineering-day-27-compiler-attribution/`

- `single-solve/journal.md`, `single-solve/team_solved.json`, `single-solve/start_response.json`
  (the saved start is a team-status payload)
- `single-solve/artifacts/cand-adapter/solve.py` (wrapper around the library reader plus an independent
  HCFG scan)
- `time-trial/ranked-1/result.json`, `time-trial/ranked-1/accepted_plan.json`,
  `time-trial/regular/result.json`
- `mastery/ranked-1/result.json`, `mastery/ranked-1/accepted_plan.json`, `mastery/regular/result.json`
- `challenges/day27/_run/librarian/binary-reverse-engineering-day-27-compiler-attribution.md`
- `challenges/day27/_run/lane-binary-reverse-engineering-day-27-compiler-attribution.log`
- `progress.md` (grep `^| 27 .*compiler-attribution`)
