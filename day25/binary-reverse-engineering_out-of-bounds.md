# Out of Bounds

| | |
|---|---|
| Day / Lane | 25 / Binary Reverse Engineering |
| Challenge ID | `binary-reverse-engineering-day-25-synthetic-vulnerability-root-cause` |
| Technique | cumulative recap bundle; vulnerable/corrected JAR role pair, HCFG record in a companion classfile |
| Single Solve | Independent |
| Time Trial | Independent |
| Mastery | Independent |

## Summary

The prompt promised a bounds-error proof, a reachability trace and a comparison with a "corrected twin".
None of that was asked. The 300 questions were the usual cumulative recap (25 blocks of 12 static
facts), and the new `d25` block was a pair of small JARs, one named `vulnerable` and one `corrected`,
which only needed container and config-record facts read from them. The Day 24 resolver already handled
that shape, so it produced all 300 answers with no code change. All three modes went CORRECT.

## The challenge

Prompt (quoted from the start response): "Somewhere in here, the code reaches where it shouldn't. Prove
the bounds error is real, trace its reachability, and compare against the corrected twin that learned its
limits."

Single Solve delivered one `evidence.zip` (SHA-256 checked against the start response) and 300 answer
ids, 12 per block for `d01` to `d25`. The formats were the lane's usual mix: `pattern` (SHA-256 of
member N, a hex or machine label), `enum` (ELF class, container format) and `exact_text` (HCFG config
values). The new block was:

- `d25_artifact_sha256_1/_2`: SHA-256 of members 1 and 2 in ZIP central-directory order.
- `d25_analysis_corrected_format` and `d25_analysis_vulnerable_format`: pick the unique member whose
  basename ends `corrected.jar` / `vulnerable.jar`, classify it by magic and parsed container (enum
  `elf, pe, pe_cli, jar, wasm, neko_bytecode`).
- `d25_analysis_config_{campaign,domain,mutex,url}`: "decode both role members and require their config
  objects to be byte-for-byte equal before using the common value".
- `d25_analysis_corrected_config_{campaign,domain,mutex,url}`: the same four values from
  `corrected.jar` only.

No question asked for a diff, a changed function, an out-of-bounds index or a trace.

## Solution

1. **Recon.** `start` returned the ZIP and 300 ids. Counting ids by prefix showed 25 blocks of 12, and
   the `d25` block's question text was a copy of the `d24` block with the number changed. That matched the
   Day 24 "Patch Diffing" block, which had been solved the day before.
2. **Read the questions before reversing.** Because none of them asked for a vulnerability proof,
   decompiling the JARs was not needed. The two members only had to be classified, hashed and have their
   config record decoded.
3. **Run the existing resolver.** Our `question_facts.py` (library entry BRE-QFACTS-B107) resolved 300 of
   300 ids on the first run. Its Day 24 handling covers the three rules this block uses:
   - the member hint (`corrected`, `vulnerable`) selects a member by the name in the question;
   - "both role members" decodes the HCFG record of every non-`.json` member and requires the values to
     agree before answering;
   - the config scan walks every classfile's constant pool, not only the `Main-Class`.
4. **The record is not in the Main-Class.** On Day 24 the CRC32-valid HCFG record sat in the `...$.class`
   companion (Scala/Kotlin-object style) classfile, not the class named by `Main-Class`. The question's
   "Parse every JAR classfile constant pool" wording is what makes the any-classfile scan correct; a
   Main-Class-only scan returns no record. We did not re-inspect the Day 25 classfiles by hand: the Day 24
   finding was carried over, and the resolver's output was accepted by the validator.
5. **Wrap and submit.** The solver was a 49-line contract-v1 wrapper that runs `question_facts.py` and
   converts the map to strings; a second, independent solver (a 947-line extension of the Day 24 rubric
   solver) produced the same 300 answers. Where our alternate spellings were uncertain, the submission
   planner sent the base set (285 ids) first and then one call per risky group, each at its first variant.

Single Solve went CORRECT after six calls: base set, then groups `3`, `machine`, `1`, `2` and `format`,
every group accepted at variant 0. The six calls are the planner's group-by-group isolation, not six
attempts at fixing wrong answers. There was no fix round.

## Time Trial and Mastery

The Time Trial was the same 300-id schema with a fresh bundle. Mastery delivered **nine** evidence ZIPs
(nine sets, 2,700 ids; Day 24 had eight). Both regular (untimed) attempts and the first ranked attempts
were CORRECT in one call with the full map (`full-prefer`): ranked Time Trial in an 80 s window (9.6 s end
to end locally), ranked Mastery in a 595 s window (21.2 s end to end). Mastery needed the resolver run
once per set; sets are paired to their ZIPs by SHA-256. The solvers ran per set in parallel, 2 to 7 s
each.

Two later ranked Time Trial reruns and one ranked Mastery rerun on later dates were also CORRECT. The
Time Trial reruns used the grouped submission (7 calls and 6 calls); Mastery was again one call. These
reruns changed nothing in the solve and were not part of the prize ranking.

## Dead ends and gotchas

- **The prompt was bait.** "Prove the bounds error is real, trace its reachability" read like a
  decompile-and-prove challenge. It was static recap fields only, the same lesson as Day 24 where the
  title was "Patch Diffing". Read the question text before opening a disassembler.
- **Alternate spellings were carried but never needed.** The plan held variants for the enum (`jar` vs
  `JAR`), the member-order SHA-256 groups from earlier days, and the `i386`-style machine labels for
  earlier days' PE fields. The validator accepted variant 0 everywhere, so none of those alternates was
  validator-confirmed on Day 25. Do not treat the planner's variant lists as established rules.
- **PE machine labels.** The accepted Mastery maps carried `AMD64` where the resolver emits `x86-64`
  (10 to 14 fields per Mastery run, 3 in one Time Trial). These are the same fact under the pattern
  syntax and were accepted either way; they are regression "equivalents", not errors.
- **The "285 accepted" counter in the journal** is the running total one group before the final CORRECT,
  not the number of fields needing a second guess.

## Takeaways

- **Read the question, not the prompt.** In this lane the flavor text (the "bounds error", the "corrected
  twin") can be unrelated to what is graded. Check the question text of the new block first.
- **Role-member pair recipe** (library entry BRE-ROLEPAIR-B108): select members by the role named in the
  question, require both configs to agree, and scan every classfile constant pool for the CRC32-valid
  HCFG record rather than trusting `Main-Class`.
- **A cumulative resolver pays off.** Because the bundle is the previous day's plus one block, the same
  tool resolved 300/300 with no change; a regression run of it against the saved accepted plans confirmed
  Single Solve 300/300 exact and Time Trial and Mastery with only the AMD64 label differences.
- What we would do the same: submit the base set first, then risky groups individually, so a wrong group
  can be isolated without discarding the rest.

## Evidence

Leaf: `challenges/day25/binary-reverse-engineering/binary-reverse-engineering-day-25-synthetic-vulnerability-root-cause/`

- `single-solve/journal.md`, `single-solve/start_response.json`, `single-solve/accepted_plan.json`
- `single-solve/artifacts/cand-adapter/solve.py` (wrapper over `question_facts.py`),
  `single-solve/artifacts/cand-rubric/solve.py` (second solver, identical output)
- `time-trial/regular/result.json`, `time-trial/ranked-1/result.json` (also `ranked-2`, `ranked-3`)
- `mastery/regular/result.json`, `mastery/ranked-1/result.json` (also `ranked-2`)

Day-level: `challenges/day25/_run/lane-binary-reverse-engineering-day-25-synthetic-vulnerability-ro.log`,
`challenges/day25/_run/librarian/binary-reverse-engineering-day-25-synthetic-vulnerability-ro.md`,
`library/binary-reverse-engineering/` entries BRE-QFACTS-B107 and BRE-ROLEPAIR-B108, and
`progress.md` (grep `^| 25 .*root-cause`).
