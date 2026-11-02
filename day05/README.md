# Day 5: overview

## At a glance

- **5 challenges, 15 modes** (Warmups, Metadata Forensics, Script Analysis, Mobile Android, Binary Reverse Engineering; the Web Exploitation lane did not exist yet).
- **Provenance by mode:** 11 Independent, 1 Journal-recorded, 3 Local mark only, 0 Team, 0 n/a.
- **Theme:** every challenge was a static decode of a delivered file (decimal bytes, a PNG header, descriptor-bearing scripts, an APK asset, ELF/PE headers), so the real cost was never the technique. It was the server clock in Time Trial and Mastery (20-110 s), multi-delivery Mastery bundles, and, for the two largest maps, validator feedback that only says `incorrect`.

## Challenges

Provenance labels: **Independent** = the leaf holds our own CORRECT result file; **Journal-recorded** = our own CORRECT is recorded in the journal (and local ledger) but no result file was saved; **Local mark only** = only the local `progress.md` row exists; **Team** = graded solve was a teammate's; **n/a** = mode absent.

| Lane | Challenge | Technique | Single Solve | Time Trial | Mastery |
|---|---|---|---|---|---|
| Warmups | [By the Numbers](warmups_by-the-numbers.md) | Decimal-ASCII decode; Mastery chains five numbered-record codecs | Local mark only | Independent | Independent |
| Metadata Forensics | [Chunk Norris](metadata-forensics_chunk-norris.md) | PNG chunk walk with CRC check, IHDR dimensions; cumulative note/JPEG/PNG field family | Local mark only | Independent | Independent |
| Script Analysis | [Hatch Careful](script-analysis_hatch-careful.md) | Descriptor-in-code decoding across five languages; batch-level-`incorrect` bisection | Independent | Independent | Independent |
| Mobile Android | [Backup Spill](mobile-android_backup-spill.md) | Static APK asset read; `split` fragment codec (permutation reassembly) | Local mark only | Independent | Independent |
| Binary Reverse Engineering | [Rust Belt](binary-reverse-engineering_rust-belt.md) | Static ELF/PE header facts, symbol tables, CRC-validated XOR `HCFG` config records | Independent | Independent | Journal-recorded |

Notes on the labels, checked against the leaf files:

- **Single Solve for Warmups, Metadata and Mobile is Local mark only.** Those leaves hold the candidate and `submission.json`, but no graded response. The Warmups journal says the candidate was "not submitted"; the Mobile journal waits on a "Game Master submission for service-side confirmation" and later records the mode as CORRECT only through a 2026-10-20 retroactive sweep. Single Solve submissions for these lanes also have no entry in the local submission ledger. We cannot show our own graded CORRECT for them.
- **Time Trial and Mastery are Independent** where a `ranked-N/result.json` reads CORRECT. Each of those modes also has a first-pass CORRECT in the ledger from 2026-10-23, so the original attempt and the ranked rerun agree. The original attempts are backed by the ledger and journals, the reruns by result files.
- **Script Analysis Single Solve** is Independent because `probe-final-1-response.json` is a graded CORRECT. **BRE Single Solve** is Independent because `submission-response-2.json` is CORRECT.
- **BRE Mastery is Journal-recorded.** The timed attempt expired, `retry` returned `rotation_exhausted`, and the mode was finished through untimed `practice`. The journal and the ledger both show one CORRECT submission of 80 fields, but no response file was kept. A later ranked Mastery attempt was refused with the same `rotation_exhausted`, so no result file can be produced by a rerun.

## What the day had in common

**Short windows made the solve a scripting problem.** On Days 1-5 a plain `retry` is already the timed attempt, and the clocks seen on this day were 20 s (Warmups Time Trial), 40 s (Warmups Mastery), 50 s (Metadata, Script, Mobile and BRE Time Trial) and roughly 80-110 s (Mastery elsewhere). Nothing could be fetched or run by hand. Every mode that went CORRECT did so from a single in-process chain of retry, download of every delivery, solve, key-set check and submit.

**Mastery deliveries are not one ZIP.** Three lanes shipped Mastery as several separate deliveries under their own capability links (23 `blob.txt` files, three APKs, two `evidence.zip` files). The Game Master client's default `artifact` fetch returns only the first one, and Warmups named all 23 files identically, so the delivery index is the only link from question to file. Metadata is the exception: it shipped one outer ZIP holding two inner ones.

**Time Trial and Mastery ask for more than Single Solve did.** Metadata's Single Solve was one field; Time Trial and Mastery asked the cumulative nine-field template (note family, JPEG and PNG). Warmups Mastery was a five-record codec chain, not the single decimal blob of the other modes. BRE's later prompts spelled out rules (stripped specimens answer `[]`, the `.hcfg` CRC rule) that Single Solve left implicit. Reading the later modes' wording confirmed the Single Solve reading.

**The disguise is flavor.** "Unexpected structural passenger" (Metadata), "backup-admitted state" (Mobile), "carriers" such as wheels and PyInstaller bundles (Script) and "Rust runtime shapes" (BRE) all described something the data did not contain. The Metadata PNG had no passenger, the APK had no backup file, the Script archive held only text, and 32 of BRE's 40 fields were about earlier days' specimens. In each case the answer was the literal field the question named.

### Validator and format traps seen across lanes

- **Batch-level `incorrect` only.** Script Analysis Single Solve took 14 full-map rejections for two wrong fields out of 357. A subset made only of correct fields returns `accepted` with `accepted_ids`, which is how the two bad Batch `*-control` values were isolated. BRE Single Solve showed the same blindness: one wrong symbol list was `incorrect` with no per-field hint.
- **String-only answers.** Native JSON arrays and integers returned `invalid_answers` and graded nothing (BRE Single Solve), so Metadata's size and BOM fields and BRE's array answers had to be strings.
- **Descriptor field beats plan text; Batch controls vary by fixture.** The Script adapters took the descriptor's literal value, and `for /l ... do call :LABEL` was `call_subroutine`, not the prompt's `for_once`. The Time Trial map took different control values for the same profiles, so each fixture was classified from its own code.
- **Spelling and enum quirks (BRE).** `little endian`, `I386`, `PE32_CLI`, `x86-64` for ELF and `AMD64` for PE were accepted in pattern fields; enum-style fields were exact.
- **Strip state and format change per mode (BRE).** d01 had a symbol table in Single Solve and none in Time Trial, and d02/d04 arrived as PE32+ rather than ELF. Classify each delivery by magic bytes.
- **`split` direction (Mobile).** `order[i]` is the output slot of `parts[i]`. Every saved split asset used the same three-part `[2,0,1]` permutation, whose inverse gives a different string, so the direction mattered and the forward reading was accepted in every mode.
- **Expired is not wrong.** A submission to an expired attempt returns `expired`, grades nothing, and still counts in the ledger (see the timeline).
- **Saved ZIPs and APKs were CRLF-damaged** on our disk (also noted in CLAUDE.md for "Day 5 `artifact-response.bin`"). Check against the start response's SHA-256 before analysis. The PNG signature contains a genuine `\r\n`, so a blanket undo corrupts it and the repair has to be selective and hash-checked.

## Mistakes in our own tooling and process

These are ours, not the challenges'. The challenge pages point here.

- **Warmups, By the Numbers.** The first Mastery attempt expired while we fetched 23 deliveries by hand. The first journal entry also called the deliveries "single decimal blobs"; each was a five-record chain (base64, rot13, hex, binary, decimal), a correction recorded in the journal's 2026-10-24 deep review. The retry's scripted pass went CORRECT. Separately, the Game Master client's `artifact` command could fetch only the first of several deliveries.
- **Metadata Forensics, Chunk Norris.** The first Time Trial `start` expired (about 50 s) before the mapper, adapted from an earlier day, had been run, and the submission to it came back `expired` and used a ledger entry. Mastery's first attempt was lost to Game Master 503/500 responses on artifact fetches; that was an outage, not our error, and a `retry` after recovery went CORRECT.
- **Script Analysis, Hatch Careful.**
  - Single Solve spent 14 full-map submissions before switching to chunked, bank-what-is-accepted submissions. Two of the earlier submissions (the first-half and second-half maps) still carried stale values for other suspect fields, so they proved nothing.
  - One test of `T1496.001` was sent inside a map that still held the two wrong controls, so it said nothing.
  - A journal claim that an earlier day had accepted `set` as the Batch benign adapter could not be verified anywhere and should not have been relied on.
  - The Time Trial journal's prose table of controls does not match its accepted map; we relied on the accepted map.
  - The Time Trial's first attempt expired while the mapper was still being built (one `expired` ledger entry).
- **Binary Reverse Engineering, Rust Belt.**
  - Single Solve spent one call on a malformed payload and one on native JSON arrays (`invalid_answers`), before the corrected submission was CORRECT. The malformed first call left no response file.
  - Time Trial attempts 1 and 2 both returned `expired`: the first pipeline shelled out per file and was too slow, and attempt 2 also lost `answer_map.py` to a self-inflicted `rm -rf artifacts`. Attempt 3 imported the inspector in-process and went CORRECT.
  - Mastery's only timed attempt was spent validating the pipeline inside the live window, so the submit returned `expired` and `retry` then returned `rotation_exhausted`. Staging the multi-set pipeline before the first `start` would have kept the timed attempt.
  - In the later ranked sweep, `ranked-3` failed before issuing a submit because of a missing dependency in our environment.
- **Mobile Android, Backup Spill.** No process error beyond the day-wide ones: the Single Solve candidate was never submitted with a saved response, and the stored APK copies were CRLF-damaged.
- **Day-wide.** We kept no graded response for three of the five Single Solves, which is why those modes can only be labelled Local mark only. No Single Solve for these lanes appears in the ledger.

## Tooling and process around this day

The repository history starts on 2026-10-22, after Day 5 was first played (`progress.md` dates the Single Solves 2026-10-18/19). The Day 5 Single Solve work therefore has one bulk commit, not an incremental history. The commits and files that touch this material:

- **2026-10-22, "Add Day 5 challenge work" (`94552b7e`):** the Single Solve leaves as they stood.
- **2026-10-23, "Solve Days 5, 7, 8, 10, 11 Time Trial/Mastery backlog" (`1d82b9d5`):** the Time Trial and Mastery solutions for this day (the BRE `fast_pipeline.py` and `mastery_pipeline.py`, the lane journals) and `progress.md` rows. The same commit changed `tools/gm.py` so `artifact` accepts `--capability` and `--name`, which fetches a specific delivery by its capability token; this is what the Warmups, BRE and Script Mastery bundles needed. The Script Mastery and Warmups Mastery journals describe that use.
- **`gm.py practice`:** the BRE Mastery journal dates the `practice` subcommand to 2026-10-23 (the untimed continuation after `rotation_exhausted`). In git, the `cmd_practice` code first appears in the 2026-10-24 commit `cedbe666`, so I cannot tell from history whether it existed uncommitted on the 23rd. CLAUDE.md records the lesson under "Rotation exhausted (Day 5 Rust Belt Mastery, 2026-10-23)": `rotation_exhausted` is not a dead end, and `practice` can still complete the mode.
- **CLAUDE.md lines that cite this day:** the rotation-exhausted note above, the CRLF-damage warning (Day 5 `artifact-response.bin`), and the "Days 1-5: a plain `retry` IS the ranked attempt" rule behind the timing section.
- **Ranked sweep of Days 1-10 (`tools/ranked/PROCESS.md`, ranked passes logged 2026-10-25/26):** commit `1160dffa` ("Add ranked-attempt tooling and Day 2-5 ranked results") added the per-challenge solvers under `tools/ranked/solvers/` and a `ranked-1` result for every Day 5 Time Trial and Mastery except BRE Mastery. Each went CORRECT in one call, in 1.2-4.4 s total including download and submit (BRE Time Trial 2.3 s). BRE Mastery was refused with `rotation_exhausted`, as CLAUDE.md records for Days 2-7. The reruns showed that a general solver that rebuilds the answer map from the delivered bytes handles the short windows.
- **Later BRE Time Trial reruns (2026-10-31 and 2026-11-01, commits `38668151` and `f0e20aa1`):** `ranked-2` and `ranked-4` went CORRECT (3.0 s and 3.9 s). `ranked-3` has a pass log but no result file; the ranked-3 failure cause above comes from the BRE page and I did not trace it further. The 2026-11-01 sweep also re-confirmed `rotation_exhausted` for BRE Mastery (progress row).

## Timeline

From the local submission ledger (calls metered by the Game Master client) and `progress.md`. Ledger times are as recorded; they appear to be Pacific time, since the Warmups Mastery journal logs the same first attempt at 15:18Z against a ledger time of 08:19. Single Solve is not in the ledger for these lanes, so those rows use `progress.md` dates and the leaf files.

| Mode | Date and calls | Outcome |
|---|---|---|
| Single Solves (all five) | 2026-10-18 (Script 2026-10-19) | Script Analysis: 14 full-map `incorrect`, then chunked probes, CORRECT. BRE: malformed call, one `incorrect`, then CORRECT (3 calls). Warmups, Metadata, Mobile: local mark only. |
| Warmups TT | 2026-10-23, 1 call | CORRECT |
| Warmups Mastery | 2026-10-23, 2 calls | 1 `expired`, then CORRECT |
| Metadata TT | 2026-10-23, 2 calls | 1 `expired`, then CORRECT |
| Metadata Mastery | 2026-10-23, 1 call | CORRECT |
| Mobile TT / Mastery | 2026-10-23, 1 call each | CORRECT |
| Script TT | 2026-10-23, 2 calls | 1 `expired`, then CORRECT |
| Script Mastery | 2026-10-23, 1 call | CORRECT (714 fields) |
| BRE TT | 2026-10-23, 3 calls | 2 `expired`, then CORRECT |
| BRE Mastery | 2026-10-23, 2 calls | 1 `expired` (08:31), then CORRECT via `practice` (23:26) |
| Ranked sweep | 2026-10-25/26 | CORRECT in 1 call for every Time Trial and Mastery except BRE Mastery |
| Later BRE TT reruns | 2026-10-31, 2026-11-01 | CORRECT; BRE Mastery ranked `rotation_exhausted` |

Six calls on 2026-10-23 returned `expired` and were charged to the ledger without being graded: Warmups Mastery 1, Metadata TT 1, Script TT 1, BRE TT 2, BRE Mastery 1.
