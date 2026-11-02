# ELF Aware

| | |
|---|---|
| Day / Lane | 1 / Binary Reverse Engineering |
| Challenge ID | `binary-reverse-engineering-day-01-c-elf-orientation` |
| Technique | ELF header / section-table / symbol-table reading; locating a `mov eax, imm32; ret` stub in `.text` |
| Single Solve | Solved, graded result not preserved (local mark only) |
| Time Trial | Independent (ranked rerun on 2026-10-26 graded CORRECT; earlier 2026-10-18 solve is a local mark) |
| Mastery | Solved, graded result not preserved (local mark only); no mode directory in the leaf |

## Summary

An orientation challenge: a handful of small, plain C ELF64 binaries, and questions that are just
header and symbol facts (class, endianness, machine, type, symbols, entry point) plus one value
hidden in a trivial function. Nothing needs execution. The only real difficulty was answer
spelling: the validator wanted `little` for endianness, not `little-endian`.

## The challenge

The prompt: "Before you can reverse anything, you have to learn to read. These plain C binaries are
your primer: headers, section tables, symbol tables. Start with `readelf -h -S` and work your way
down."

Single Solve delivers `evidence.zip` (the only file this leaf kept for that mode; no journal or
start response survives). The Time Trial rerun delivered a ZIP of `slot-N.elf` members and asked 36
questions, nine fields for each of four slots, all `exact_text` ("exact artifact-derived value
only ... use the literal `none` only when the requested field is absent"):

`elf_class`, `endianness`, `machine`, `elf_type`, `marker`, `has_symtab`, `exported_symbol`,
`recover_code`, `entry_point`.

The Time Trial window was 50 seconds, and a plain `retry` on this legacy-route day was the ranked
attempt, so the solve was scripted end to end (issue, download, solve, submit).

## Solution

The specimens come in four build profiles, identified by a marker string in a `.huntress` section
(`BR01-SYMBOL-ORIENTATION`, `-STRIPPED-PROFILE`, `-SECTION-BEACON`, `-IMMEDIATE-RECOVERY`). The Time
Trial drew two of them (`SYMBOL-ORIENTATION` and `IMMEDIATE-RECOVERY`), each twice. The solver is
a small script on top of the workspace's ELF reader:

1. Header fields come straight from the ELF header: class `ELF64`, little-endian, machine
   `x86-64`, type `ET_EXEC`, entry point `0x401030` (identical across the four slots in that
   attempt).
2. `marker` is the text of the `.huntress` section.
3. `has_symtab` is true when the section-name table contains `.symtab`.
4. `exported_symbol` is the one non-baseline function name, either `recover_code` or
   `inspect_section`; for a stripped file there is no `.symtab`, so the answer is `none`.
5. `recover_code` is the immediate in the lone `mov eax, imm32; ret` stub in `.text`. The solver
   scans `.text` for `\xb8<imm32>\xc3` and accepts the result only if there is exactly one hit,
   which works even when the symbol name has been stripped:

```python
STUB_RE = re.compile(rb"\xb8(....)\xc3", re.S)
...
if name == b".text":
    hits = STUB_RE.findall(raw[s[4]:s[4] + s[5]])
    if len(hits) == 1:
        return struct.unpack("<I", hits[0])[0]
```

In the accepted Time Trial map the stub values were `1337` (symbol-orientation profile) and `7331`
(immediate-recovery profile).

## Time Trial and Mastery

The leaf's Time Trial evidence is the ranked rerun of 2026-10-26: 36 answers in 5 spelling groups,
12 submission calls, CORRECT, 6.67 s total inside the 50 s window (0.99 s to issue, 0.28 s to
download, 0.03 s to solve, 5.36 s to submit and walk the spelling variants).

The leaf has no Mastery directory and no saved Mastery response. The project notes record that the
Day 1 Mastery for this challenge had no rotation left (`rotation_exhausted`) when a ranked rerun
was tried, so it could not be redone. The only record of that mode is the `progress.md` row marked
as a player-directed local mark; I could not verify the Mastery solve from files in the leaf.

## Dead ends and gotchas

- **`endianness` spelling.** The first full submission (all primary spellings) came back
  `incorrect`. Re-submitting by field group, the other four groups (`elf_class` `ELF64`, `machine`
  `x86-64`, `elf_type` `ET_EXEC`, `exported_symbol`) were accepted on their first variant, and
  `endianness` was rejected for `little-endian` and `little endian` and accepted for the third
  variant, bare `little`. Later days accept `little-endian`; Day 1 does not. The validator's
  `incorrect` is batch-level, so the only way to find this was submitting by group.
- **The plan file shows the rejected primary.** `accepted_plan.json` in the leaf still lists
  `little-endian` for `endianness`; the winning value is `chosen.endianness = 2` (`little`) in
  `result.json`. Read the chosen variant index, not the plan's first column.
- **Stripped specimens.** In the stripped profile the `recover_code` name is gone, so a
  symbol-based lookup returns nothing; the `.text` stub scan still finds the value, and
  `exported_symbol` is then `none`. The Time Trial draw here contained only unstripped profiles,
  so that path was not exercised live in this attempt.
- No start response or journal was kept for the original Single Solve, so its exact question set
  and accepted spellings were never recorded; the Time Trial spellings above are the only
  validator-confirmed ones for this challenge.

## Takeaways

- For an ELF "primer", use an off-the-shelf parser for every header and section fact
  (`readelf -h -S`, or a parsing library) instead of hand-rolling offsets, and reserve custom code
  for the one non-standard item (the `mov eax, imm32; ret` stub).
- Match tiny stub functions by byte pattern in `.text` rather than by symbol name, so the same
  solver survives stripped builds.
- When a field is free text (`exact_text`), keep an ordered list of standard spellings and
  submit by field group, so a single wrong spelling does not hide the other fields' correctness.
  Searching spellings inside a 50 s window is cheap (two extra calls here) if the groups are
  prepared before the clock starts.

## Evidence

Leaf base path: `challenges/day01/binary-reverse-engineering/binary-reverse-engineering-day-01-c-elf-orientation/`

- `time-trial/ranked-1/result.json`, `accepted_plan.json`, `pass.log`, `start_response.json`
  (CORRECT, chosen variants, submission rounds, prompt and question format)
- `single-solve/evidence.zip` (raw delivery only; no journal or result)
- Solver: `tools/ranked/solvers/binary-reverse-engineering-day-01-c-elf-orientation/solve.py`
- `progress.md` rows for Day 1 (all three modes: "Player-directed local mark", 2026-10-18)
