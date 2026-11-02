# Day 6 overview

**At a glance:** 5 challenges, 15 modes (one per lane: warmups, metadata, mobile, binary RE, script analysis; the `challenges/day06/` folder holds no web or welcome entry). Provenance by mode: 12 Independent, 3 Journal-recorded, 0 Team, 0 Local mark only, 0 absent. Theme: a day of cumulative "recap" content where the flavor text of three of the five titles and prompts points away from what was actually asked (an exported provider that does nothing, Bash graphs that are six other languages, a Java class-file prompt over a 48-field multi-day recap), and the one literal title (octal) was the easiest solve.

## Challenges

| Lane | Challenge | Technique | Single Solve | Time Trial | Mastery |
|---|---|---|---|---|---|
| Warmups | [Eight Is Enough](warmups_eight-is-enough.md) | Octal-ASCII blob; Mastery adds numbered multi-codec rows (Base64, ROT13, hex, binary, decimal, octal) | Journal-recorded | Independent | Independent |
| Metadata Forensics | [Portable Deception](metadata-forensics_portable-deception.md) | PDF header version read from raw member bytes, including nested ZIPs | Journal-recorded | Independent | Independent |
| Mobile Android | [Provider Paths](mobile-android_provider-paths.md) | Static APK triage; answer is a hex-coded asset, the exported-provider story is a decoy | Journal-recorded | Independent | Independent |
| Binary Reverse Engineering | [Pool Party](binary-reverse-engineering_pool-party.md) | Cumulative recap of ELF, PE/CLR and JAR specimens; HCFG config in a class-file Base64 constant | Independent | Independent | Independent |
| Script Analysis | [Shell We Begin](script-analysis_shell-we-begin.md) | Static reconstruction of encoded behavior descriptors in six script languages; graph-to-role/ATT&CK mapping | Independent | Independent | Independent |

What backs each label (details are on each page):

| Challenge | Single Solve | Time Trial | Mastery |
|---|---|---|---|
| Eight Is Enough | Journal only (CORRECT recorded, no response file) | `submission-response.json` CORRECT, plus `ranked-1` | `submission-response.json` CORRECT, plus `ranked-1`..`ranked-3` |
| Portable Deception | Journal only | Journal for the original; `ranked-1/result.json` CORRECT | Journal for the original; `ranked-1/result.json` CORRECT |
| Provider Paths | Journal only | Journal for the original; `ranked-1/result.json` CORRECT | `submission-response.json` CORRECT, plus `ranked-1` |
| Pool Party | Saved CORRECT response (the 24th submission) | `attempt-01` CORRECT response, plus `ranked-1`, `ranked-2` | `attempt-01` CORRECT response |
| Shell We Begin | Saved response 68 CORRECT | Only `ranked-1/result.json` CORRECT; the original timed attempts never reached CORRECT | Only `ranked-1`, `ranked-2` CORRECT; no graded response from the original play-through |

Two caveats on the Independent label for Shell We Begin: for Time Trial and Mastery the graded evidence is a later scripted ranked pass, not the original play-through (see "Mistakes" below). Pool Party Mastery has no ranked rerun because its rotation pool was exhausted (`CLAUDE.md`, "Timed ranking on older days"), and Single Solve modes are never ranked.

## What the day had in common

- **The title and prompt were a weak guide to the question.** Portable Deception talked about renamed impostors and false promises but asked for one literal field (`pdf.header_version`); Provider Paths described abusing an exported provider whose `query()` returns null; Shell We Begin said "six Bash behavior graphs" over a bundle of PowerShell, JScript, Batch, VBScript, Python and Bash; Pool Party said Java constant pools but 40 of its 48 fields covered the Day 1-5 specimens. Count the answer IDs and read each ID's own name first. Eight Is Enough was the exception: its "eight fingers" line did mean octal.
- **Answers were decoded from the artifact, never inferred from the story.** Warmups and mobile answers were sitting in the payload (octal bytes; `assets/huntress_puzzle.json`); metadata was a regex over the first 1024 bytes; BRE and script analysis were extraction plus a rubric.
- **Recap content compounds.** Pool Party's 48 fields were Day 1-6 fields, and its Day 3 and Day 4 accepted maps already held the answer to the field that cost 24 submissions. Later metadata fixtures (ranked reruns) were also cumulative: 10 and 20 fields next to the original single `pdf.header_version`.
- **Time Trial and Mastery windows were short and the fixtures changed per attempt.** Windows were 20 s and 45 s (warmups), 50 s and 80 s (metadata, script), 50 s and 110 s (mobile), about 48 s and 78 s (BRE). Each retry could issue different bytes, so a map from another attempt fails. Every mode that went CORRECT did so from a script that downloaded, verified the hash, solved and submitted in one pass.
- **Mastery bundles scaled a Single Solve map, not its method.** 28 warmup blobs, 2 nested PDFs, 3 APKs, 2 BRE archives (96 fields) and 2 script sets (798 fields) all reused the Single Solve decoder.

## Validator and format traps seen on this day

- **Answer spelling.** Warmups: submit the slug alone (text after the last `:`), not the whole sentence. Metadata: `x.y` only, not `PDF-1.4`. BRE: `PE32_CLR` as a compound container token, `x86` for the PE32 machine, `little endian` / `little` spellings varied by field, and the section list without the leading empty name.
- **BRE `d01_analysis_symbols`.** The question read only "What is the exact analysis symbols value". Many filtered readings were rejected (function-only, global-only, without `FILE` records, and others); the accepted one was every nonempty `.symtab` name in table order including `FILE` entries and duplicates. This is the 24-submission item recorded in `CLAUDE.md` ("Binary").
- **Script analysis Batch one-item `FOR` controls.** The prompt said classify as `for_once`; the validator accepted `call_subroutine` for four specimens in Single Solve, and `direct` for a fifth. `CLAUDE.md` cites Day 6 in its Batch-control lessons.
- **Script analysis ATT&CK for Python.** `T1105` was rejected for two Python chains ending in `dispatch_process`; the language sub-technique `T1059.006` was accepted, while the Bash `staged_chain` in the same bundle took `T1105` (`CLAUDE.md` lists Day 6 among the days where `T1105` was rejected for Python). The prompt's precedence text does not explain the split.
- **`incorrect` carries no per-field detail** for composite submissions. Pool Party and Shell We Begin had to be bisected by day, family, specimen and field. Only `accepted`/`accepted_ids` and `remaining` are signals.
- **Mode-suffixed IDs.** Time Trial and Mastery use `<id>/time_trial` and `<id>/mastery`; the bare ID is the Single Solve. All answer values are strings (arrays as compact JSON text).
- **`prerequisite_required`.** Time Trial and Mastery starts on Pool Party and Shell We Begin returned the question list but no attempt, artifact or answer IDs until the earlier mode was CORRECT. Only a response with an attempt ID and a hash is a usable start.
- **Missing hashes.** Provider Paths Mastery returned `download: null` and `sha256: null`, so the APKs could only be paired to answer IDs by package name.

## Mistakes in our own tooling and process

These are deferred from the challenge pages. They are about our workflow, not the challenges.

- **Shell We Begin, Time Trial: the mapper was built from the wrong bytes.** The first timed attempt's candidate values were generated from the Single Solve archive, not the Time Trial ZIP, and the submission was `incorrect`. The journal admits this in its audit. Fifty specimen-group diagnostics were then sent after that attempt's deadline and all returned `expired`, so they graded nothing.
- **Shell We Begin, Time Trial: three timed attempts, none CORRECT.** The second retry, used with 50 newly authorized submissions as grouped diagnostics, confirmed 307 of 399 fields; the rest were Batch and VBScript controls and expansions. A source review produced a locally audited full map, but no graded CORRECT response was saved for it.
- **Shell We Begin: progress rows without local backing.** The `progress.md` rows for Time Trial and Mastery say the Game Master returned CORRECT. The leaf journals' own 2026-10-20 audit notes say nothing saved supports that (the Mastery leaf held only `prerequisite_required` starts and an offline pipeline; the notes suggest the work may have been done elsewhere and never copied back, which we could not determine). The graded evidence in this repo is the later scripted ranked pass. This is the "never write a row without our own graded result" rule that `CLAUDE.md` later made explicit.
- **Pool Party, Mastery: the solver failed under the timer.** It treated every PE file as a managed assembly and crashed on a native PE32+ member (CLI header RVA zero). Nothing was submitted by that run, but the start had already consumed the last unique attempt (`remaining 0`), and the corrected solver submitted with about 36 s left of 78 s. Testing the parser on all Day 1-5 member types before starting the clock would have caught it.
- **Pool Party, Single Solve: 24 submissions for one field.** Comparing member SHA-256 values against the previously accepted Day 3 and Day 4 maps would have given the answer directly. The default cap of 20 was exhausted on attempt 20 and four more were made after an extension.
- **Pool Party, Time Trial: driver path bug.** The one-shot driver's first run stopped when its start-lock path repeated `time-trial`. It sent no request and consumed nothing.
- **Pool Party: provenance by inference.** A status response attributed a shared CORRECT to a handle; the journal inferred "almost certainly our identity" from a matching timestamp. Later project rules say a Game Master `status` is never evidence of our own solve (here our own saved responses cover it).
- **Single Solve journal-only records.** Warmups, metadata and mobile Single Solve have no saved response or result file; the journals record CORRECT and the retroactive sweep marked them recorded. Hence Journal-recorded, not Independent.
- **Warmups, Mastery: decoder not kept.** The original 28-blob script was not saved in the leaf (the journal says so); the later ranked reruns used the shared `codec_chain` solver.
- **Mobile: noise in the leaf.** The Androguard parse used a vendored `artifacts/pydeps` tree of roughly 82 MB, which the journal's deep review calls noise.

## Tooling and process changes around this day

Day 6 predates version control: the repository history starts on 2026-10-22, and the one commit that adds the day's challenge files (`380aff56`, "Add Day 6 challenge work") records files, not when the work happened. The journals date the original play-through to 2026-10-19; the commit date should not be read as the solve date. There is no release automation for this day (no `_run` folder).

What the history and notes support:

- **Library promotion (2026-10-20 retroactive sweep, from the journals; `3bcba4e6`, 2026-10-22, "Add technique library").** Journals record generalizing Day 6 work into library tools: this leaf's `static_decode.py` into `material_codec_decode.py`, rubric logic into `behavior_rubric.py` and `gotchas-d05-d09.md`; the octal blob into `decode_numeric_text.py` and `codec_chain.py`; the class-file reader into `classfile_inspect.py` and `answer_lint.py`; the PE/CLR check into a library entry (BRE-PE-CLR-001); the PDF regex into `pdf_inspect.py`; the mobile decoder into `asset_codecs.py`.
- **Ranked pass over Days 6-10 (commit `1df4ec2c`, 2026-10-25; `tools/ranked/PROCESS.md`).** It added per-challenge solvers for all five Day 6 challenges, the case lists (`bre-d06-10.json`, `metadata-d06-10.json`, `mobile-d06-10.json`, `scriptA-d06-08.json`) and the first `ranked-1` results. The Day 6 deadlines in those results are 2026-10-26 UTC. All nine ranked-1 runs went CORRECT on one call, each 1.4 to 5.0 s end to end; BRE Mastery was not attempted (`rotation_exhausted`).
- **Later reruns.** `021814b6` (2026-10-31) added Shell We Begin Mastery `ranked-2`; `38668151` (2026-11-01) added Eight Is Enough Mastery `ranked-2` and Pool Party Time Trial `ranked-2`; `f0e20aa1` (2026-11-01) added Eight Is Enough Mastery `ranked-3`. All were CORRECT on one call. These are speed-ranking attempts, not the original solves.
- **`CLAUDE.md` lessons citing Day 6:** the `d01_analysis_symbols` rule (full `.symtab` table including `FILE` entries), the Batch one-item `FOR` rule (`call_subroutine`, Days 6, 7, 10), and the Python `T1105` rejection. Local-evidence-only rules (2026-10-27/28) and the staged-chain table postdate this day.

## Timeline and effort

All dates are from journals or result files; times are UTC where the file gives one. The script analysis Single Solve completion time is not stated in the files we read.

| When | Event | Source |
|---|---|---|
| 2026-10-19 03:02:50 | Portable Deception Single Solve CORRECT (submission 1) | metadata SS journal |
| 2026-10-19 03:04:18 | Provider Paths Single Solve CORRECT (submission 1) | mobile SS journal |
| 2026-10-19 03:31:33 | Pool Party Single Solve CORRECT (24th submission) | BRE SS journal and saved response |
| 2026-10-19 03:35 to 03:39 | Shell We Begin Time Trial: three 50 s attempts, none CORRECT | script TT journal |
| 2026-10-19 06:31 to 06:33 | Pool Party Time Trial (submitted 06:31:48) and Mastery (submitted 06:32:36, about 36 s left) CORRECT | BRE journals |
| 2026-10-26 | Ranked pass: 9 modes CORRECT, 1 call each | `ranked-1/result.json` files |
| 2026-10-31 to 2026-11-01 | Later reruns (script Mastery, warmups Mastery x2, BRE Time Trial) | `ranked-2`/`ranked-3` files, commits |

Submissions per mode (journals, progress rows and ranked results):

| Challenge | Single Solve | Time Trial | Mastery |
|---|---|---|---|
| Eight Is Enough | 1 | 1 (+1 ranked) | 1 (+3 ranked) |
| Portable Deception | 1 | 1 (+1 ranked) | 1 (+1 ranked) |
| Provider Paths | 1 | 1 (+1 ranked) | 1 (+1 ranked) |
| Pool Party | 24 | 1 (+2 ranked) | 1 |
| Shell We Begin | 68 | original: 1 full + 50 expired + 1 full + 50 diagnostics, no CORRECT; ranked: 1 | original: none; ranked: 1 (x2) |

The ranked counts are separate fresh attempts, each CORRECT on one call per its `result.json`.
