# Crash Course

| | |
|---|---|
| Day / Lane | 26 / Binary Reverse Engineering |
| Challenge ID | `binary-reverse-engineering-day-26-crash-triage` |
| Technique | cumulative-recap field bank; JVM classfile constant-pool and method-table reads; crash-trace JSON read |
| Single Solve | Independent |
| Time Trial | Independent |
| Mastery | Independent |

## Summary

"Crash Course" is a 312-question evidence bundle. 300 of the questions are recaps of the
previous 25 days' binary fields, and 12 are Day 26's own, built around a Groovy-compiled JAR and a
`crash/trace.json` file. The prompt talks about unwinding a stack and finding a faulting
instruction, but the data needed no debugging or disassembly. Three new fields were small reads
from the JAR and the trace, and our existing question-facts library covered everything else.
All three modes went CORRECT; the Single Solve took three submission calls and no fix round.

## The challenge

Prompt: "It crashed. That's the good news — the dump is a confession. Unwind the stack, find the
faulting instruction, and triage what went wrong from the wreckage."

`start` delivered one `evidence.zip` and 312 answer ids (26 blocks `d01`..`d26`, 12 questions per
block, each question scoped to the members named `dNN--*`). Answer kinds were mixed: `pattern`
(SHA-256, `0x`-prefixed hex), `enum`, `integer` and `exact_text`. The Day 26 block was:

- `d26_artifact_sha256_1` and `d26_analysis_format` (`jar`): the usual hash and container
  classification.
- Seven HCFG fields (`config_campaign`, `config_domain`, `config_mutex`, `config_url`,
  `config_crc32_hex`, `hcfg_version`, `xor_key_hex`): the recurring "HCFG" config record.
  The question spells out the record layout: Base64 constants of at least 80 characters in the
  primary classfile's constant pool, `HCFG` magic, version byte, one-byte XOR key, little-endian
  payload length and CRC32, XOR-decoded JSON payload.
- `crash_fault_method`: the exact first string of the `frames` array in `crash/trace.json`.
- `crash_frames_match_methods`: `true` exactly when every string in `frames` appears in the primary
  classfile's method-name set.
- `method_count`: the primary classfile's method-table count.

Time Trial and Mastery reused the same question shapes. Time Trial had an 80 s server window with
the same 312 ids; Mastery was ten bundles (`set1`..`set10`, ten `evidence.zip` files, 3,120 ids)
under a 660 s window.

## Solution

1. **Recon.** The JAR is the `d26--` member (`d26--sample.jar`, Main-Class `Day26Variant000`), a
   Groovy build. Besides the classfiles it holds `crash/trace.json` with
   `frames: [stageThree, stageTwo, stageOne]`.
2. **Recap fields.** The Day 26 block is the only new part of the bank. The other 300 ids resolved
   from the library's question-facts reader (it already carried every older field), so nothing
   for Days 1-25 was rewritten. 310 of the 312 ids resolved unchanged before the Day 26 handlers
   existed.
3. **HCFG.** In this JAR the CRC-valid HCFG record sits in the **Main-Class** classfile (version 1,
   header key byte `0xb1` in the Single Solve). The Groovy closure class carries no record, which
   makes "primary classfile" unambiguous: it is the one named by `Main-Class`. The decode is the
   one the question prescribes: XOR the payload with the key, check CRC32, read the JSON, take the
   top-level property.
4. **Crash fields.** `crash_fault_method` is a plain read: `stageThree`. For
   `crash_frames_match_methods`, the solver parsed the Main-Class method table directly and
   checked that all three frame names were in it: `true`. `method_count` is the Main-Class method
   table count: 10 (the closure class has 5; the question asks for the primary one).
5. **Submission.** The solver wrote a 312-answer map and submitted it. For the three new
   fields the plan carried alternates (method count 10 | 5 | 15 for the whole JAR; frames match
   `true` | `false`); the first variant of each was accepted, so the alternates were never used.
   Single Solve: 3 calls, CORRECT.

A short version of the method-table parse (from the solver) shows why it was done by hand rather
than with a Java tool: the constant pool has to be walked anyway to find the HCFG constants.

```python
# classfile constant pool: tag 1 = Utf8, 5/6 (long/double) take two slots
cnt = struct.unpack(">H", b[8:10])[0]
...
elif t in (5, 6):
    pos += 9
    i += 1          # long/double occupy two constant-pool entries
```

After the pool, the access/this/super/interfaces header is skipped, then fields and methods are
read as `(access, name_index, descriptor_index, attributes)` records; the methods table gives both
the count and the name set.

## Time Trial and Mastery

- **Time Trial.** The same solver went CORRECT on the first submission of the first ranked
  attempt (80 s window, 11.15 s end to end locally, including download). The HCFG key changed per
  fixture (the Single Solve key was `0xb1`; Time Trial attempts drew others, e.g. `0x2b` and
  `0x74`), but the crash fields stayed `stageThree` / `true` / 10 methods.
- **Mastery.** Ten sets under one timer, 3,120 answers submitted in a single call (CORRECT on
  the first call; 28.05 s locally, 660 s window). The ids are prefixed `setN.`. The HCFG key
  differs per set (`0xcd`, `0x98`, `0x36`, ...), but the crash fields were identical across all
  ten sets: `stageThree`, frames match `true`, method count 10.
- Later ranked reruns of both modes (re-checks on later dates) were also CORRECT. Time Trial
  needed three calls on those reruns because the alternate-reading groups were submitted as
  separate single-id calls first; the base map was unchanged.

## Dead ends and gotchas

- **The prompt oversells the work.** "Find the faulting instruction" suggests a dump to unwind
  and disassemble. There is no machine code to read: the "crash" is a JSON file listing three
  method names, and the questions are three string/count facts. The title and prompt were
  thematic only.
- **`false` for frames-match was never exercised.** Every delivery we saw (Single Solve, Time
  Trial, all ten Mastery sets) had all frames in the Main-Class method set. The predicate's
  `false` branch and the "whole JAR" or closure-class `method_count` readings exist only as
  planned alternates; none were ever needed, so we cannot say the validator would have accepted
  them.
- **"Primary classfile" is Main-Class.** Method count 10 (Main-Class) vs 5 (closure) vs 15
  (everything) is the one ambiguity in the block. Reading the question as "the classfile named
  by `Main-Class`" matched the validator.
- **Ids come from `start`.** Mastery ids carry a `setN.` prefix and each set has its own key and
  hash, so the map has to be built per delivery rather than reused from the Single Solve.

## Takeaways

- Recap days are cheap if the earlier fields live in one tested reader: here only three
  handlers were new (`crash_fault_method`, `crash_frames_match_methods`, `method_count` for the
  Main-Class classfile) and the library's regression run reproduced the accepted maps exactly
  (7,176 of 7,176 values across five leaves).
- For JVM questions, read classfiles directly: the constant pool gives the Base64 HCFG candidates
  and the method table in one pass, and "primary" resolves from the manifest's `Main-Class`.
- Name to search for: **HCFG record in a classfile constant pool** (Base64 `HCFG` header, XOR
  key, CRC32). The same record shape recurs through the binary lane's recap fields.
- Faster next time: submit the base map once and the alternate groups only after a rejection.

## Evidence

Leaf base path: `challenges/day26/binary-reverse-engineering/binary-reverse-engineering-day-26-crash-triage/`

- `single-solve/journal.md`, `single-solve/start_response.json`, `single-solve/accepted_plan.json`
- `single-solve/artifacts/cand-rubric/solve.py` (question-facts wrapper plus the alternate groups),
  `single-solve/artifacts/cand-adapter/solve.py`
- `time-trial/ranked-1/result.json`, `time-trial/regular/result.json`
- `mastery/ranked-1/result.json`, `mastery/regular/result.json`, `mastery/ranked-1/start_response.json`
- `challenges/day26/_run/librarian/binary-reverse-engineering-day-26-crash-triage.md`,
  `challenges/day26/_run/lane-binary-reverse-engineering-day-26-crash-triage.log`
- `library/binary-reverse-engineering/question-facts.md` (BRE-QFACTS-B107)
- `progress.md` (grep `^| 26 .*crash-triage`)
