# Day 10 overview

**At a glance:** 5 challenges (one per lane, no web lane yet), 15 modes. By provenance label: 10
Independent, 5 Journal-recorded, 0 Local mark only, 0 Team, 0 n/a. Theme: a day of answer-format and
rubric traps. The technique in each lane was easy to find (Base32, a PE header field, a shuffled-fragment
asset, an XOR+CRC config record, a decoded descriptor), and nearly every wrong submission came from how
the answer had to be spelled or which field the validator trusted.

## Challenges

| Lane | Challenge | Technique | Single Solve | Time Trial | Mastery |
|---|---|---|---|---|---|
| Warmups | [Thirty-Two Below](warmups_thirty-two-below.md) | Base32 decode; numbered codec chain in Mastery | Journal-recorded | Independent | Independent |
| Metadata Forensics | [Where Do We Begin?](metadata-forensics_where-do-we-begin.md) | PE `AddressOfEntryPoint` RVA, fixed-width hex | Journal-recorded | Independent | Independent |
| Mobile Android | [Network Config Split](mobile-android_network-config-split.md) | `split` fragment codec in `assets/huntress_puzzle.json`; unreferenced Network Security Config as decoy | Journal-recorded | Independent | Independent |
| Binary Reverse Engineering | [Zig Zag](binary-reverse-engineering_zig-zag.md) | Static ELF/PE/JAR/.NET parsing, HCFG record (XOR + CRC32), cumulative recap fields | Journal-recorded | Independent | Independent |
| Script Analysis | [Bind Over Matter](script-analysis_bind-over-matter.md) | Decoding inert script descriptors (89 specimens, 753 fields); descriptor field vs. printed graph rubric | Journal-recorded | Independent | Independent |

How to read the labels (each page gives the evidence):

- **Journal-recorded (all five Single Solves).** The journal records our own submission as CORRECT. No
  `result.json` or response file was saved for any of the five, and `progress.md` has a "Solved" row for each.
- **Independent (all ten Time Trial and Mastery modes).** The original attempts (2026-10-23 per
  `progress.md`) are journal-recorded only. The label rests on the saved `ranked-N/result.json` files from the
  later ranked reruns, every one of which reads CORRECT in one submission call. Counting reruns: Warmups 1 TT
  + 4 Mastery, Metadata 1 + 1, Mobile 1 + 1, BRE 5 + 4, Script 2 + 3.
- **Script Analysis Single Solve caveat.** The journal's RESOLUTION section says the final CORRECT was
  "recovered via `op=status`", and the last accepted submission came after the recorded 30 of 30. There is
  no saved response for it. We call it Journal-recorded, but it is the weakest of the five.

## What the day had in common

- **Cumulative-day structure.** Warmups Mastery chained the codecs of Days 1-10 (record N uses Day N's codec),
  and the BRE challenge was a ten-group recap in which only the d10 block was new. The mapper or chain
  decoder built earlier did most of the work; the day's own new piece was small.
- **Short windows, so scripted solves.** Day 10 ran on the "legacy" route, where a plain `retry` is the ranked
  attempt (the `result.json` files say `route: legacy`). Windows were 20 s (Warmups TT), 50 s (Metadata,
  Mobile and Script TT), 65 s (Warmups Mastery), 80 s (Metadata and Script Mastery, BRE TT), 110 s (Mobile
  Mastery) and 200 s (BRE Mastery). Each lane ended up as one fetch, solve, submit pass.
- **Prompts were decoys or only loosely tied to the answer.** The Mobile prompt asks for a manifest vs.
  Network Security Config diff (irrelevant to the answer), and the Script prompt talks about a WMI MOF
  subscription (not one of the 753 questions). The Warmups title ("32") was the real clue.
- **Time Trial and Mastery changed fixtures, not mechanism.** Metadata answer IDs start `d09-pe-...` on
  Day 10 and must be copied from the live response. Mastery bundles could arrive as several artifacts
  (Mobile 3 APKs, BRE 3 deliveries) or nested ZIPs (Metadata, Script), so a single download link was not enough.

### Validator and format traps seen across lanes

| Lane | Trap |
|---|---|
| Metadata | `0x65bd0` rejected, `0x00065bd0` accepted: the entry RVA wants the full 32-bit width though the prompt only says "0x prefix". |
| BRE | Booleans must be lowercase `true`/`false` strings; `d03_analysis_format` needed the compound token `PE32_CLI`, not bare `PE32`. Mastery set 3 delivered d02/d04/d10 as PE while sets 1 and 2 were ELF, so classification has to use magic bytes. |
| Script | Descriptor `attack`/`adapter`/`role` fields beat both the printed graph rule and the operation-plan text. A staged chain ending in `dispatch_process` took the language sub-technique (`T1059.006`, `.004`, `.007`), though the question text said `T1105` was "also accepted". Batch `*-control` answers followed the wrapper's structure, not its wording. |
| Mobile | `order[i]` is the output slot of `parts[i]`, not the i-th piece to emit. Mastery delivers several APKs. |
| Warmups | The answer is the last non-empty decoded line; the Mastery members were 10-record chains, not single blobs. |
| All batch submissions | `incorrect` on a multi-answer call carries no per-field feedback, so cause isolation costs submissions (Script Single Solve: 30 logged). Every answer value must be a string. |

CLAUDE.md carries the lasting versions of these lessons: the Day 10 Bind Over Matter bullets (descriptor
`attack` wins; `adapter` is `call` not `set`; the staged-chain terminal `dispatch_process` took the language
sub-technique), the padded-hex note for the PE entry RVA, and the "Mastery d02/d04/d10 sets mix PE and ELF"
note under Binary.

## Mistakes in our own tooling or process

Collected here from the challenge pages, with the evidence for each.

- **Bind Over Matter (Script).**
  - Single Solve spent attempts on a 665-field submission and several full maps whose `incorrect` result
    carried no accepted IDs, before switching to small groups. A partition plan (family, then suffix, then ID)
    from the start would have been cheaper.
  - The final successful Single Solve submission(s) came after the recorded 30 of 30 and are not counted in
    `challenges/day10/submission-attempts.md`; no response file survives. `progress.md` and the journal say the
    cap was a player-set limit the server did not enforce. We could not determine from the files how the
    extension past 30 was authorised or how many submissions it took.
  - The Single Solve journal's RESOLUTION text says Bash and Node.js chains were accepted "as originally
    mapped", which conflicts with its own attempt log (`T1105` rejected). The Time Trial journal's 0 of 753 diff
    against the final map is the reliable statement.
  - The corrected Time Trial mapper first differed from the accepted Single Solve map in 4 of 753 fields (the
    staged-chain regression). Diffing against that map before the live timer caught it with no submissions
    spent. The first Time Trial start also expired before analysis began and was re-issued with `retry`.
- **Zig Zag (BRE).**
  - Single Solve submission 1 failed on our own formatting: Python `True` instead of `true` for four enums, and
    bare `PE32` for the compound `PE32_CLI`. The pre-submission answer linter was added afterwards.
  - Time Trial and Mastery each lost a first attempt to a window that expired during mapper work; the Mastery
    submission on the retried attempt first hit an HTTP 503 (not a verdict).
  - The Time Trial and Mastery journals mis-stated which delivery mixed PE files (a later review corrected them:
    the accepted Time Trial bundle was all ELF, only Mastery set 3 had PE). The Time Trial leaf's `artifacts/`
    still holds `.exe` extractions that we could not reconcile with that review.
  - Two later ranked reruns (Time Trial `ranked-5`, Mastery `ranked-4`) have only a `pass.log`, which reads
    "solver failed" because `pyzipper` was not available in that environment. Per `tools/ranked/PROCESS.md` the
    pass issues the attempt before the solver runs, so those attempts were issued and spent. The next reruns
    (`ranked-6`, `ranked-5`) went CORRECT.
- **Thirty-Two Below (Warmups).** The Mastery journal first described the 48 members as single Base32 blobs and
  corrected itself later: they are 10-record chains. The solver was unaffected.
- **Metadata, Warmups, Mobile, BRE (bookkeeping).** `challenges/day10/submission-attempts.md` reads 0
  submissions for their Time Trial and Mastery rows (only the Script TT and Mastery rows read 1) although the
  journals record a submission each; the table was not updated after those solves. The same file lists a cap of
  10 per ID+mode for these rows while several journals say "attempt 1 of 20"; we could not determine which cap
  applied.
- **Where Do We Begin? (Metadata).** The Single Solve journal contains literal NUL bytes (a raw PE signature
  pasted in), so `grep` treats it as binary.
- **Network Config Split (Mobile).** Not a mistake, but a gap: the journal did not test the inverse reading of
  `order`, so the direction rule rests on the accepted answer plus a library note about the sibling codec.

## Tooling and process changes around this day

The repository history starts on 2026-10-22, after Day 10 was first played (Single Solve rows are dated
2026-10-20), so the original solving has no commits of its own. What the history does show:

- `800c882a` (2026-10-22): first commit of the Day 10 work, i.e. the Single Solve journals, mappers, the 30
  Script submission request/response files, and `submission-attempts.md`.
- `1d82b9d5` (2026-10-23): Time Trial and Mastery backlog for Days 5, 7, 8, 10, 11. Its message records
  the `tools/gm.py artifact` `--capability`/`--name` options (so each delivery of a multi-artifact Mastery can be
  fetched) and the Day 10 staged-chain ATT&CK lesson added to CLAUDE.md. That message also says the modes were
  "confirmed via read-only `gm.py status`"; that is not the evidence used for the labels above.
- `1df4ec2c` (2026-10-25): the Days 1-10 ranked sweep. It added the per-challenge solvers under
  `tools/ranked/solvers/` (Day 10 used the shared engines `bank_d9_10.py`, `bre_engine.py` and the metadata
  engine), the `ranked-1` result files, and `tools/ranked_pass.py`. These are what back the Independent labels.
- `38668151` and `f0e20aa1` (2026-11-01): further ranked reruns (`ranked-2` to `ranked-6`) in a speed-scoreboard
  gap sweep, including the two failed-solver logs noted above.

## Effort

Submission counts, from `submission-attempts.md` and the journals (the Time Trial and Mastery counts in that
file are partly stale, see above):

| Lane | Single Solve | Time Trial (original) | Mastery (original) | Ranked reruns that went CORRECT |
|---|---|---|---|---|
| Warmups | 1 | 1 | 1 | 1 TT, 4 Mastery |
| Metadata | 2 | 1 | 1 | 1 TT, 1 Mastery |
| Mobile | 1 | 1 | 1 | 1 TT, 1 Mastery |
| BRE | 2 | 1 (retried attempt) | 1 on the second attempt | 5 TT, 4 Mastery |
| Script | 30 logged, plus unlogged | 1 | 1 | 2 TT, 3 Mastery |

Ranked-rerun total times per pass (`t_total` in `result.json`): Warmups 1.6-5.5 s, Metadata 1.6 s, Mobile 1.3-2.2 s,
Script 2.7-4.4 s, BRE 5.7-15.7 s (dominated by the large archive download and the solve). The server-side rank
is not recorded in the leaves.
