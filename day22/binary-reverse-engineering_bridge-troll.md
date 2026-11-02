# Bridge Troll

| | |
|---|---|
| Day / Lane | 22 / Binary Reverse Engineering |
| Challenge ID | `binary-reverse-engineering-day-22-managed-native-bridge` |
| Technique | cumulative recap bundle (264 static fields); new block is a managed .NET assembly with an HCFG config record |
| Single Solve | Solved (journal-recorded); no `result.json` was preserved, see "Provenance note" |
| Time Trial | Independent |
| Mastery | Independent |

## Summary

"Bridge Troll" is the Day 22 binary challenge. Despite the P/Invoke-and-FFI story, it asks for 264 static
facts about an AES-protected ZIP of specimens, the same recap format as the previous days with one more
12-field block added for Day 22. The new specimen is a managed .NET DLL, and all 12 new values read
cleanly from its bytes. The only field that failed was one spelling: the machine label had to be `i386`
in lowercase, and `I386`, `0x14c` and `332` were all rejected. Single Solve needed an extension of the
attempt budget to find that one field. Time Trial and Mastery then went CORRECT in one call each once the
solver emitted `i386`.

## The challenge

Prompt: "Managed code on one side, native code on the other, and a bridge of P/Invoke and FFI in between.
Pay the troll: trace every crossing and map what goes over." Nothing in the prompt matches the questions.
`start` delivered one `evidence.zip` (AES-encrypted, password `infected`) and 264 answer IDs:

- Single Solve and Time Trial: 22 blocks of 12 IDs (`d01_...` through `d22_...`), one block per release day
  so far. The d01-d21 members were carried over from earlier days. Each question starts "Use all dNN--
  members in ZIP central-directory order" or names a primary specimen.
- Mastery: six deliveries (six ZIPs), 1,584 IDs, each prefixed `setN.`.

The Day 22 block has 12 fields:

- `d22_artifact_sha256_1..3`: SHA-256 of the three `d22--` members in central-directory order.
- `d22_analysis_format`: an enum (`elf`, `pe`, `pe_cli`, `jar`, `wasm`, `neko_bytecode`).
- `d22_analysis_machine`: "the processor architecture identified by its executable-header machine field".
  Any normalized label, standard tool label or decimal/hex code was advertised as accepted.
- `d22_analysis_metadata_version`: the CLR metadata-root version string at `BSJB + 16`, NULs stripped.
- `d22_analysis_config_{campaign,domain,mutex,url,crc32_hex}` and `d22_analysis_hcfg_version`: fields of the
  first CRC-valid `HCFG` record in the primary assembly. The record layout is given in the question: magic,
  version byte, XOR key byte, little-endian length, CRC32, then the XOR-encoded JSON payload.

## Solution

1. **Inventory the Day 22 members.** The three `d22--` members were the managed primary `sample.dll`, its
   `sample.runtimeconfig.json`, and `FSharp.Core.dll` (a 5.7 MB dependency with no HCFG record and a
   non-I386 machine field). The questions say the `.runtimeconfig.json` and dependency DLLs are "delivery
   support only", so the primary assembly had to be picked by rule.
2. **Pick the primary.** The unique `d22--` name pattern (starts `d22--`, ends `.dll`) also matches the
   FSharp.Core dependency. The solver chose the managed DLL that carries a CRC-valid HCFG record and
   broke ties by smallest image:

   ```python
   withcfg = [t for t in dlls if t[2]]            # managed DLLs with a CRC-valid HCFG record
   pool = withcfg or dlls
   row22, pe22, recs22 = min(pool, key=lambda t: len(t[0]["data"]))
   ```

3. **Read the fields.** The primary had a populated CLR directory, so `format` is `pe_cli`. The PE
   machine field is `0x14c`, the CLR metadata version string is `v4.0.30319`. The HCFG scan found two
   byte-identical CRC-valid records (offsets 15891 and 18432), so "first record" and "last record" were
   the same: campaign `BR22-000`, domain `node-000.day22.example.invalid`, mutex
   `Global\HuntressBR22_000`, URL `https://node-000.day22.example.invalid/api/BR22-000`, CRC32 `0x1704bcdf`,
   version `1`. We cross-checked all 12 values afterwards with an independent HCFG reader and `pefile`.
4. **Inherited d01-d21 block.** The earlier days' mapping was copied from the previous day's
   validator-confirmed solver (the library's cumulative-recap procedure) rather than rewritten.
5. **Submit.** The first full submission of 264 fields was `incorrect`. Bisection with the 20-call cap
   narrowed 262 of 264 fields to accepted and ran out of calls with two ids unresolved. The player then
   authorized 30 more attempts (35 of an approved 50 were used). Splitting the remainder isolated one
   wrong field, `d22_analysis_machine`. We had sent `I386`. `0x14c` and `332` were rejected, then
   `i386` returned **CORRECT** for all 264.

## Time Trial and Mastery

- **Time Trial** was the same 264-field shape with a fresh `evidence.zip` (different SHA-256 per attempt).
  The regular attempt on release day went CORRECT after 6 calls: the merged plan submits the
  non-ambiguous 257 fields first, then each ambiguous group alone, and the final group returned CORRECT.
  All five groups were accepted at variant 0. The ranked attempt (80 s window) went CORRECT in one call, 11.4 s
  total including download and solve. Later ranked reruns also went CORRECT (those are speed-board attempts, not
  the prize clock).
- **Mastery** delivered six ZIPs (1,584 IDs with `setN.` prefixes). The same solver ran once per delivery
  and one call with every set's answers went CORRECT in both the regular attempt and the ranked attempt
  (400 s window, 20.1 s total).
- The only change needed was the Single Solve fix (emit `i386`); the logs show no other adjustment.

## Dead ends and gotchas

- **The machine label is validator-specific, not value-specific.** For the same I386 PE, the d03 machine field
  accepted `I386`, `i386`, `x86`, `Intel 80386` and `0x014c`, but the d22 "executable-header machine"
  field rejected `I386`, `0x14c` and `332` and took only `i386` (the field text promises that decimal
  and hex codes are accepted, and they were not). The `I386` rejection happened inside the very submission where
  d03's `I386` was accepted, so the two fields do not share a rubric. The library's rule is to emit
  lowercase `i386` for this family of fields from Day 22 on.
- **The right alternate was already in the solver.** The rubric solver had declared `i386` as one
  of its own group variants for the machine field, but the release automation's merge step only built
  alternate groups for ids where the two solvers disagreed. Both solvers agreed on `I386`, so the
  alternate was never tried. That is an automation bug (see the day overview), not a challenge trap, but it is why
  a one-field spelling issue cost a second budget extension instead of one extra call.
- **A narrow bisection log is hard to use.** The first run logged only counts per chunk, not the ids,
  so two unresolved ids could not be identified from the log. The offline cross-check showed that all 12
  d22 values were correct and that the problem was not a new Day 22 rubric miss, but that evidence does not point at
  the machine field; only the live isolation did.
- **Primary assembly ambiguity.** `FSharp.Core.dll` also matches "starts `d22--`, ends `.dll`". Picking by
  HCFG presence rather than name was a choice we could not check against the validator before the machine
  field was fixed, but it did not cause any rejection.
- **Names vs. story.** Nothing about P/Invoke or FFI appears in any question. The title is not a guide to the work.

## Takeaways

- **Cumulative recap bundle**: carry the previous day's validator-confirmed mapping forward and add
  only the new block. Verify the new block's values from the bytes with a second reader before spending
  attempts.
- When a pattern field says many spellings are accepted, still treat the spelling as an enum-like unknown. Submit
  one variant per field in isolation early. A rejected 264-field batch tells you nothing about which field failed.
- For labels of an I386 PE, try lowercase `i386` first when the question says "executable-header machine"; the
  library records this as the accepted spelling for Days 22-24 and keeps `I386` for the d03-style wording.
- Declared alternates should be submitted even when candidates agree. The merge logic was fixed after this challenge.

## Evidence

Leaf base path: `challenges/day22/binary-reverse-engineering/binary-reverse-engineering-day-22-managed-native-bridge/`

- `single-solve/journal.md` (cap_reached, cross-check, resolution), `single-solve/artifacts/fix-submit*.log`
- `single-solve/artifacts/cand-rubric/solve.py` (the `d22` block and the `i386` line)
- `time-trial/regular/result.json`, `time-trial/ranked-1/result.json` (also `ranked-2`, `ranked-3`)
- `mastery/regular/result.json`, `mastery/ranked-1/result.json` (also `ranked-2`)
- `time-trial/*/start_response.json`, `mastery/*/start_response.json` (question counts, deliveries)
- `challenges/day22/_run/lane-binary-reverse-engineering-day-22-managed-native-bridge.log`
- `library/binary-reverse-engineering/lane-evolution-and-playbook.md` (machine-label family ruling) and
  `library/binary-reverse-engineering/question-facts.md` (accepted spellings)
- `progress.md` (grep `^| 22 .*managed-native-bridge`)

**Provenance note.** The Single Solve leaf holds no `result.json` or `accepted_plan.json`. The journal
and the `progress.md` row record our own CORRECT on release day (after the budget extension), and the lane
log shows the Time Trial run starting with "Single Solve already CORRECT". The leaf also has a
`team_solved.json` written on 2026-11-01. It was written by a later resume run whose `start` simply read the
existing CORRECT status, and it does not describe who solved it on release day. That is why the Single Solve
is labeled journal-recorded rather than independent. There is no preserved graded result file to
back it.
