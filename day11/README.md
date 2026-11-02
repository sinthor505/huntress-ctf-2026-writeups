# Day 11 overview

**At a glance:** 5 challenges (one per lane, no web lane yet), 15 modes. By provenance label: 8
Independent, 7 Journal-recorded, 0 Local mark only, 0 Team, 0 n/a. Theme: a day of recap fields and
decoy prompts. Four of the five lanes reused a mechanism from Day 10 or earlier (a PE resource field, a
declared APK codec, the cumulative binary bundle, the script-descriptor bank), so the cost was in format
spellings, rubric choices and short windows, not in finding the technique.

## Challenges

| Lane | Challenge | Technique | Single Solve | Time Trial | Mastery |
|---|---|---|---|---|---|
| Warmups | [Safe Travels](warmups_safe-travels.md) | URL-safe Base64 with stripped padding; numbered-record chain in Mastery | Journal-recorded | Independent | Independent |
| Metadata Forensics | [What's My Name Again?](metadata-forensics_whats-my-name-again.md) | PE `RT_VERSION` `OriginalFilename` | Journal-recorded | Independent | Independent |
| Mobile Android | [Local Secrets](mobile-android_local-secrets.md) | Declared Base64 + XOR asset (`huntress.mobile.puzzle.v1`); credential-looking strings as decoys | Journal-recorded | Independent | Independent |
| Binary Reverse Engineering | [D-Day](binary-reverse-engineering_d-day.md) | Static parsing of a d01-d11 recap bundle; HCFG records (XOR + CRC32), new Base64-run carrier | Journal-recorded | Journal-recorded | Journal-recorded |
| Script Analysis | [By Any Other Name](script-analysis_by-any-other-name.md) | Decoding inert script descriptors (95 specimens, 811 fields); ATT&CK and Batch-control rubric | Journal-recorded | Independent | Independent |

How to read the labels (each page gives the evidence):

- **Journal-recorded (all five Single Solves).** Each journal records our own submission as CORRECT (Script
  Analysis on the second 20-attempt authorization). No `result.json` or response file was saved for any of
  them, and `progress.md` has a "Solved" row for each (dated 2026-10-20).
- **Independent (Warmups, Metadata, Mobile and Script Time Trial and Mastery).** The original attempts
  (2026-10-20 for Script, 2026-10-23 for the others, per `progress.md`) are journal-recorded only. The label
  rests on the saved `ranked-N/result.json` files from later ranked reruns, all CORRECT. Counting reruns that
  graded: Warmups 1 TT + 4 Mastery, Metadata 1 + 1, Mobile 1 + 1, Script 2 + 3. Two further Metadata TT reruns
  returned `practice_only` (pool spent) and are not counted.
- **Journal-recorded (Binary Time Trial and Mastery).** No ranked result exists, because the pools were spent
  (below). The journals record CORRECT on attempt 4 and attempt 3, and `challenges/submission-ledger.json`
  independently logs a CORRECT response on the last call of each (2026-10-23). We kept the label because the
  ledger is a call log, not a saved result file.

## What the day had in common

- **Cumulative-day structure.** The Binary challenge was again a recap (110 ids, 10 per day-block d01-d11);
  only the d11 block was new (an `HCFG` record in a long Base64 run, plus a byte-offset question). Script
  Analysis re-asked Day 10's bank (753 ids) plus 58 new ones (811). Metadata's answer ids still carry a
  `d09-pe-` prefix, so the generator's numbering lags the release day.
- **Short windows, so scripted solves.** The ranked passes show `route: legacy` for every mode (a plain
  `retry` is the ranked attempt). Time Trial windows: Warmups 20 s, Metadata, Mobile and Script 50 s, Binary
  80 s. Mastery: Warmups 70 s in the ranked runs (the journal saw about 62-70 s), Metadata and Script 80 s,
  Mobile 110 s, Binary 200 s. Single Solve was untimed.
- **Prompts were decoys or only loosely tied to the questions.** Binary talks about D-language templates
  (no D-specific question), Metadata promises an "impersonator" (every question was one `OriginalFilename`),
  Script talks about native APIs and "receipts" (neither appears in the questions), Mobile frames active vs.
  decoy credentials (the answer is in the asset, not in any secret). Warmups was the exception: the riddle
  ("URL-friendly vacation") named the codec.
- **Time Trial and Mastery changed fixtures, not mechanism.** Metadata specimens varied by architecture
  (`386`, `amd64`, `arm64`), so a filename from one mode does not carry to the next. Mastery arrived as a
  53-member ZIP (Warmups), nested evidence ZIPs (Metadata, Script: two full copies of 811 ids), three APKs
  (Mobile) or three evidence ZIPs (Binary: 330 ids).

### Validator and format traps seen across lanes

| Lane | Trap |
|---|---|
| Script | Python staged chains (`acquire_remote` ... `dispatch_process`) took `T1059.006`, not `T1105`; Bash and Node.js chains of the same shape took `T1105` on Day 11. The descriptor's own `attack` beat the printed graph rule on 22 v3/host fields. |
| Script | One-item `FOR /L ... DO CALL` Batch wrappers: the question says `for_once`, but Mastery and a later Time Trial fixture wanted `call_subroutine`; Single Solve and the first Time Trial accepted `for_once`. The answer varied by fixture. |
| Binary | Accepted spellings were per day: `x86-64` for ELF, `PE-CLI` and `0x014c` for d03, `JAR`, `[0,8]` for compression methods, `ET_EXEC`, `little endian`. Mastery set 2 delivered d02/d04/d10 as PE32+ while sets 1 and 3 were ELF, so classification has to use magic bytes. `d01_analysis_symbols` is every non-empty `.symtab` name including FILE entries. |
| Warmups | The slug is the last non-empty decoded line; the blob's padding was stripped; Mastery members are 11-record chains, not single blobs. |
| Mobile | Credential-looking `SECRET-` constants and the `proof` parameter in the DEX are plumbing; the answer is only in `assets/huntress_puzzle.json`. |
| Metadata | The value must be taken verbatim from the version resource, and varies with the specimen's architecture. |
| All lanes | `incorrect` on a multi-answer call has no per-field feedback, and every answer value must be a string. |

CLAUDE.md carries the lasting versions: the Python-vs-Bash/Node.js staged-chain note, the Batch `for_once` vs
`call_subroutine` fixture dependence (Day 11 SS/TT fixtures took `for_once`, Mastery and a later TT took
`call_subroutine`), and the "Days 11-15 `T1105` for Bash/Node.js" and BRE spelling lessons.

## Mistakes in our own tooling or process

Collected here from the challenge pages, with the evidence for each.

- **By Any Other Name (Script).**
  - The Single Solve mapper checked `acquire_remote` before `dispatch_process`, so the two Python staged
    chains came out as `T1105`. That left 126 Python ids `incorrect` through the whole first 20-submission
    authorization (685 of 811 ids confirmed by family). Diffing against Day 5's accepted Python carrier map
    found it, and the second authorization went CORRECT on call 1. Diffing against an earlier accepted map
    first would have cost no submissions.
  - Time Trial lost four submissions to an expired 50 s window because id lists were built inside the window;
    a full-map submission was also `incorrect` (9 calls in total). Everything had to be precomputed before
    `retry`.
  - Mastery used 26 submissions against a raised cap of 38; set 2 carried four Batch controls that stayed
    unconfirmed until an Opus review proposed `call_subroutine` for the one-item `FOR /L` launcher.
  - The journals say `challenges/submission-ledger.json` "has no Day 11 entries" for Script Time Trial and
    Mastery, and the Single Solve journal's 20 rounds appear only in the leaf's `submission-attempt-log.json`.
    The ledger today holds only the 2026-10-26 onward ranked calls for those two modes, so the original 9 and 26
    counts cannot be re-checked. Unlike Day 10, there is no `submission-attempts.md` for this day.
  - The Script page dates the last Mastery rerun 2026-11-01; its `ledger` entry reads 2026-10-31 21:13 local
    (the result file's deadline is 2026-11-01 UTC). Same attempt, different clock.
- **D-Day (Binary).**
  - Both the Time Trial (80 s) and Mastery (200 s) first attempts expired while the mapper was being adapted,
    with `unique_attempts_remaining: 0` from the first `start`. The graded attempts were the untimed fallbacks,
    so neither is Hall-of-Fame eligible, and the later ranked sweep found both pools spent
    (`retry_variants_unavailable`). No ranked time exists for this lane's Time Trial or Mastery.
  - `tools/gm.py artifact` could only fetch the first delivery of a multi-artifact Mastery; it gained
    `--capability`/`--name` (commit `1d82b9d5`, 2026-10-23) during this solve. Each delivery also needed a
    fresh download after `retry` (`409 download_required`).
  - `objdump` was missing on the host, so the d01 disassembly was redone through `capstone`; the first PE
    section-header parse for Mastery set 2 unpacked fields in the wrong order. The CRC check caught it in a
    dry run, before any submission.
  - The ledger shows the 503s (one on the Time Trial, two on Mastery). They are transport errors and were not
    wrong guesses.
- **Safe Travels (Warmups).** The Mastery journal first described the 53 members as single Base64 blobs and
  corrected itself on review (11-record chains). The shared codec tool handled both, so the answer was unaffected.
- **Local Secrets (Mobile).** One Mastery delivery hit a transient HTTP 500 on download; a second request
  matched the start response's SHA-256. No process mistake.
- **What's My Name Again? (Metadata).** The ranked Mastery result is attributed on the public board to a solo
  account, not the team; a fresh ranked retry to fix that returned `rotation_exhausted` on 2026-11-01
  (`progress.md`), so it cannot be redone. Two Time Trial reruns were `practice_only`.

## Tooling and process changes around this day

The repository history starts on 2026-10-22, after the Single Solves (2026-10-20 rows). What the history shows:

- `de4ea7e4` (2026-10-22): first commit of the Day 11 work, i.e. the five Single Solve journals, the BRE answer
  map and candidates, and the Script artifacts and Time Trial/Mastery working files.
- `1d82b9d5` (2026-10-23): the Time Trial and Mastery backlog for Days 5, 7, 8, 10 and 11. This is where the
  Warmups, Metadata, Mobile and Binary Time Trial/Mastery solves land; its message records the
  `gm.py artifact` change above. It says the modes were "confirmed via read-only `gm.py status`"; the labels
  here do not rely on that.
- `c9906b50` (2026-10-25): the Days 11-15 ranked pass. For Day 11 it added `ranked-1` results, the per-challenge
  solvers under `tools/ranked/solvers/` (Script uses the shared `bank_d11.py`), and the `practice_only` handling.
  Its message and `tools/ranked/PROCESS.md` record that Day 11 Binary Time Trial and Mastery were unavailable
  (`retry_variants_unavailable`) and nothing was issued for them; these are the Day 11 modes behind the
  Independent labels. The Script Time Trial rerun of 2026-10-26 took 12 calls (see the Script page).
- `021814b6` (2026-10-31), `38668151` and `f0e20aa1` (2026-11-01): later gap-closure sweeps. For Day 11 they
  added Script Mastery `ranked-2`/`ranked-3`, Script Time Trial `ranked-2`, Warmups Mastery `ranked-2` to `ranked-4`
  and the two Metadata `practice_only` logs.

## Effort

Submission calls, from the journals, `progress.md` and the ledger where it covers the call:

| Lane | Single Solve | Time Trial (original) | Mastery (original) | Ranked reruns that went CORRECT |
|---|---|---|---|---|
| Warmups | 1 | 1 | 1 | 1 TT, 4 Mastery |
| Metadata | 1 | 1 | 1 | 1 TT, 1 Mastery |
| Mobile | 1 | 1 | 1 | 1 TT, 1 Mastery |
| Binary | 1 | 4 calls (3 were 503, expiry, `download_required`; CORRECT on the last) | 3 calls (two 503; CORRECT on the last) | none (pools spent) |
| Script | 20 + 1 (two authorizations) | 9 | 26 (cap 38) | 2 TT (12 calls, then 1), 3 Mastery (1 each) |

Ranked-rerun total times per pass (`t_total` in `result.json`): Warmups 1.7-4.8 s, Metadata 2.1-2.7 s, Mobile
1.5-3.7 s, Script 2.1-5.1 s. Windows were 20 s (Warmups TT), 50 s (Metadata, Mobile and Script TT), 70 s (Warmups
Mastery), 80 s (Metadata and Script Mastery) and 110 s (Mobile Mastery), so every ranked pass finished in a few
seconds. The server-side rank is not recorded in the leaves.
