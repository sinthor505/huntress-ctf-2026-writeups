# Day 7 overview

## At a glance

- **5 challenges, 15 modes** (every challenge has Single Solve, Time Trial and Mastery): Warmups, Metadata Forensics,
  Script Analysis, Mobile Android, Binary Reverse Engineering. There was no Web Exploitation lane yet.
- **Provenance:** 11 modes Independent, 4 Journal-recorded, 0 Local mark only, 0 Team, 0 n/a.
- **Theme:** four of the five titles and prompts promised something the data did not contain (a WebView bridge, Kotlin
  coroutine state machines, a ZIP "census that does not add up", "eight Node.js graphs"). In each case the real work was a
  static read of a declared field, and the traps were scope (cumulative answer sets) and validator format, not cleverness.

Provenance labels: **Independent** = the leaf holds our own CORRECT file (`result.json`, `accepted_plan.json`,
`manual_solve.json` or a `submission-response.json` reading CORRECT). **Journal-recorded** = the journal records our own
submission as CORRECT but no graded response file was saved. **Local mark only** = only a `progress.md` mark.
**Team** = a teammate's graded solve. **n/a** = mode absent.

## Challenges

| Lane | Challenge | Technique | Single Solve | Time Trial | Mastery |
|---|---|---|---|---|---|
| Warmups | [Percent Chance](warmups_percent-chance.md) | Percent-decoding to a fixed point; per-line codec dispatch in the Mastery bundle | Journal-recorded | Independent | Independent |
| Metadata Forensics | [Seven-Zip, No Relation](metadata-forensics_seven-zip-no-relation.md) | Raw ZIP End-of-Central-Directory read (`<4s4H2IH`), plus note/JPEG/PNG/PDF facts | Journal-recorded | Independent | Independent |
| Script Analysis | [Node by Node](script-analysis_node-by-node.md) | Static descriptor recovery from inert scripts (`Material_`/`Codec_`), then rubric mapping | Independent | Independent | Independent |
| Mobile Android | [WebView Bridge](mobile-android_webview-bridge.md) | Static APK triage; declared Caesar asset codec; confirming the "bridge" does not exist | Journal-recorded | Independent | Independent |
| Binary Reverse Engineering | [Routine Check](binary-reverse-engineering_routine-check.md) | XOR/CRC-protected HCFG record in a JAR constant pool, plus ELF/PE/CLR facts across Days 1-7 | Journal-recorded | Independent | Independent |

What backs each label (all paths are under `challenges/day07/<lane>/<challenge-id>/`):

- **Script Analysis Single Solve:** `single-solve/artifacts/authorized-extra-20-20261020/response-attempt-20.json` reads CORRECT.
  Its Time Trial and Mastery originals are journal entries (first-submission CORRECT); the Independent label rests on the later
  `ranked-N/result.json` files (CORRECT, 1 call each).
- **Warmups Time Trial:** the original is a journal entry; the label rests on `ranked-1/result.json`. Warmups Mastery has
  `retry-submission-response.json` (CORRECT) as well as `ranked-1/result.json`.
- **Metadata, Mobile and BRE Time Trial and Mastery:** each of these six modes has a saved `submission-response.json`
  reading CORRECT from the original attempt; all of them except BRE Mastery also have ranked reruns.
- **The four Journal-recorded Single Solves** (Warmups, Metadata, Mobile, BRE): the journal and `progress.md` (2026-10-20)
  say CORRECT, but the leaf keeps no graded response. BRE keeps `submission-request.json` only.

## What the day had in common

- **The prompt is flavor; the answer is a declared static field.** The Mobile APK never calls `addJavascriptInterface`; the
  answer is a Caesar-shifted string in `assets/huntress_puzzle.json` with its key stored beside it. The BRE title names Kotlin
  coroutines, yet no question touches coroutines; the new specimen is a JAR whose `Main-Class` constant pool carries a Base64
  HCFG record. The metadata prompt hints at a mismatched ZIP census, but in every fixture we saw the raw EOCD total equalled
  the parsed central-directory count. The Script Analysis prompt says eight Node.js graphs, and the archive holds 61 scripts.
- **Answer sets are cumulative.** BRE asked 56 fields (8 per day for Days 1-7), 112 in Mastery; Script Analysis asked 469
  fields over the whole Day 6 bank plus the eight new Node files, 938 in Mastery. Metadata Time Trial and Mastery re-asked
  earlier days' fields and added one new one. Counting `answer_ids` before reading the prompt shows the real scope.
- **Verify against a check value before trusting a decode.** BRE's HCFG record has a CRC32; Mobile's decoded string had to
  match the stated `ANS-` plus 16 hex pattern; Warmups' per-line decoders were strict enough to fail loudly. These were
  what told us a decode was right before submitting.
- **Time Trial and Mastery regenerate the fixtures.** Fresh bytes, new keys, new slugs; BRE Mastery set 2 swapped two ELF
  specimens for PE32+ ones; Script Analysis Time Trial reused a file name with a different Batch structure. Extractors keyed
  on the start response and on magic bytes carried over; anything keyed on file names or earlier answers did not.
- **Timer windows were short.** The saved start responses show 20 s (Warmups Time Trial), 50 s (Warmups Mastery) and 55/90 s
  (BRE Time Trial / Mastery), so those modes were scripted. Some Day 7 modes were untimed on the first pass
  (`deadline: null` for Metadata, Mobile and Script).

### Validator and format traps seen on this day

- **Strings only.** Script Analysis Single Solve's first POST carried integers for `stage_count` and was refused with
  `invalid_answers`. Every value has to be a JSON string (also integers, and arrays as JSON text).
- **Batch-level feedback only.** Script Analysis returned `incorrect` for a batch without naming a field; only
  `accepted` counts narrowed the search. Submitting by format, then by specimen, then per field worked.
- **Prompt rule versus validator.** The Batch uploader control's prompt said a one-item `FOR /L ... DO CALL :LABEL` wrapper is
  `for_once`; the validator accepted `call_subroutine` and rejected `for_once` in isolation. The benign control was
  `if_guard` (not `goto_dispatch`, `for_once` or `direct`). The persistence ATT&CK ID followed the concrete `/etc/cron.d/` path
  (`T1053.003`) over the `systemd_descriptor` adapter name.
- **Structure is per fixture.** Time Trial's `outbound-transfer.cmd` had a different structure from Single Solve's file of the
  same name and a different control class.
- **Language sub-technique on chains.** Chains ending `dispatch_process` took the language sub-technique (Python `T1059.006`,
  Node.js `T1059.007`, Bash `T1059.004`) on this day; CLAUDE.md records that this does not hold on every later day.
- **BRE spellings accepted here:** `little endian`, `PE32_CLR`, `Intel 80386`, `ELF` / `PE32+` for the Day 4/5 container
  question, section-name list with the leading empty string, `.symtab` names in table order with duplicates. These vary by
  day, so they were not carried forward.
- **Caesar has no wraparound** (plain `ord(c) - key` over code points) and the key differs per APK (10, 18, 8, 5, 7).
- **Archive handling.** AES ZIPs need `pyzipper`; the password is the event's `infected`, not the ZIP comment BRE tried first.
- **Mastery mapping.** Mobile Mastery answers were mapped to questions by the package name in the question text, not by file
  order.

## Mistakes in our own tooling and process

The challenge pages deferred these here.

- **Script Analysis (Node by Node), numeric answer values.** The first Single Solve POST used JSON numbers for `stage_count`
  and was refused with `invalid_answers` (see CLAUDE.md "String-only answers", which cites Days 7 and 8). The request still
  counted against the 20-submission budget. A string-only preflight did not exist yet.
- **Script Analysis (Node by Node), spending the first 20 submissions.** The first authorization (20) ended unsolved. The
  batch bisection reached 295 of 469 fields accepted by submission 18, and submissions 19 and 20 then sent the same 174
  unconfirmed fields twice, changing one field between them, which could not show which field was wrong (the journal's own
  later review says so). Our Batch control classifier followed the prompt's rule or generic shapes; the uploader control was
  only found with per-field isolation under a second 20-submission authorization (CORRECT on its 20th, 40 submissions in all).
  The journal for this one mode runs to about 12,000 lines.
- **Script Analysis (Node by Node), Time Trial.** An earlier Time Trial attempt hit `prerequisite_required` because Single Solve
  was not yet CORRECT; that leaf was stale and the Time Trial was started again after Single Solve passed (0 submissions
  consumed by the lock).
- **BRE (Routine Check), Time Trial.** The first POST was wrapped incorrectly and was rejected with `unknown_property`
  before any answer was graded; the corrected request was accepted. It happened inside a 55 s window.
- **Warmups (Percent Chance), Mastery.** The first attempt's candidate map was complete but was submitted after the 50 s
  deadline and the attempt returned `expired`. A `retry` issued a different ZIP, so none of that map could be reused; rerunning
  the extractor on the new bytes was graded CORRECT.
- **Mobile (WebView Bridge), tooling gap.** The shared asset inspector knew only Base64, XOR/Base64 and position-indexed
  fragments; a Caesar branch was added after the Single Solve. No wrong answer was submitted.
- **Saved evidence.** Four Single Solves (Warmups, Metadata, Mobile, BRE) have a journal line but no saved graded response, so
  they are Journal-recorded rather than Independent. The Time Trial and Mastery files for Warmups and Script Analysis came
  from the later ranked reruns, not from the original attempts.

No Metadata mistake of this kind appears in its page or journals.

## Tooling and process changes around this day

Day 7 predates version control: git history starts 2026-10-22, after the solves (progress rows 2026-10-20 for
all Single Solves and most Time Trial/Mastery; 2026-10-23 for Script Analysis Time Trial and Mastery). The commits below
are the earliest records of the material, not the solve dates, and there is no earlier tooling history to cite.

| Date | Commit | What it touched for Day 7 |
|---|---|---|
| 2026-10-22 | `e6aa5373` Add Day 7 challenge work | First commit of the Day 7 leaves |
| 2026-10-23 | `1d82b9d5` Solve Days 5, 7, 8, 10, 11 Time Trial/Mastery backlog | Script Analysis Time Trial/Mastery `answer_map.py`, candidate answers, decoded descriptors and journals; Time Trial journal revised |
| 2026-10-25 | `1df4ec2c` Add Day 1-10 ranked-attempt results, solvers and tooling | Per-challenge ranked solvers (`tools/ranked/solvers/<challenge-id>/`) and `ranked-1` results for the Day 7 modes |
| 2026-11-01 | `38668151` Ranked reruns: top-30 gap closure sweep, Days 2-30 | Metadata Time Trial `open-1` and `ranked-2`, Script Analysis Time Trial `ranked-2` |

Supported by journals and CLAUDE.md:

- The Script Analysis Time Trial and Mastery used the consolidated library decoders (`material_codec_decode`,
  `behavior_rubric`) plus a thin per-leaf `answer_map.py`, instead of the Single Solve's per-challenge `static_decode.py`.
  Both went CORRECT on the first submission.
- CLAUDE.md cites Day 7 in: the string-only answer rule (Days 7, 8); the Batch one-item `FOR /L ... CALL` control as
  `call_subroutine` (Days 6, 7, 10 chain); the ranked-route list (Day 7 Warmups and BRE are `legacy`, where a plain
  `retry` is the ranked attempt; Day 7 Metadata, Mobile and Script are `dual`); and the script-analysis lane pins (Day 7
  Mastery returned `legacy_attempt_pinned` in the 2026-10-28 reruns, per `tools/ranked/PROCESS.md`).
- Mobile: the Caesar branch added to the asset inspector after the Single Solve (see the Mobile page).

### What the later ranked sweep added

The ranked sweep (2026-10-25/26, then reruns on 2026-11-01) re-ran the modes with per-challenge solvers. These are
bragging-rights runs, not the prize ranking, and are recorded here only as evidence. All were one submission call.

| Mode | Route | Window | Total time | Result file |
|---|---|---|---|---|
| Warmups Time Trial | legacy | 20 s | 1.70 s | `ranked-1` |
| Warmups Mastery | legacy | 50 s | 1.95 s | `ranked-1` |
| Metadata Time Trial | dual | 50 s | 1.30 s, 2.03 s | `ranked-1`, `ranked-2` (2026-11-01) |
| Metadata Time Trial (open attempt) | n/a | n/a | n/a | `open-1` (2026-11-01 finish of an untimed open attempt, 1 call) |
| Metadata Mastery | dual | 80 s | 1.24 s | `ranked-1` |
| Mobile Time Trial | dual | 50 s | 1.09 s | `ranked-1` |
| Mobile Mastery | dual | 110 s | 1.66 s | `ranked-1` |
| BRE Time Trial | legacy | 55 s | 3.26 s | `ranked-1` |
| Script Time Trial | dual | 50 s | 2.45 s, 2.23 s | `ranked-1`, `ranked-2` (2026-11-01) |
| Script Mastery | dual | 80 s | 2.57 s | `ranked-1` |

BRE Mastery has no ranked rerun: the original start already reported zero unique attempts remaining, and `progress.md`
records a 2026-11-01 re-check returning 409 `rotation_exhausted` with nothing consumed (we did not repeat that check for this
overview). Script Mastery's later rerun attempt on 2026-10-28 returned `legacy_attempt_pinned` per `tools/ranked/PROCESS.md`.

## Effort and timeline

Only what files support:

| When | What | Source |
|---|---|---|
| 2026-10-20 (about 03:08-03:16 UTC) | BRE Time Trial (55 s window; 2 POSTs, 1 rejected for format), BRE Mastery (90 s), Warmups Mastery first attempt (expired) and retry (CORRECT) | Start responses and journals |
| 2026-10-20 | Single Solve for Warmups, Metadata, Mobile, BRE (1 submission each); Metadata, Mobile Time Trial/Mastery (1 submission each) | Journals, `progress.md` |
| 2026-10-20 | Script Analysis Single Solve: 20 submissions unsolved, then a second authorization of 20, CORRECT on its 20th | Journal, `response-attempt-20.json` |
| 2026-10-23 | Script Analysis Time Trial and Mastery, 1 submission each | Journals, `progress.md`, `1d82b9d5` |
| 2026-10-25/26 | Ranked sweep reruns (table above) | `ranked-N/result.json`, `1df4ec2c` |
| 2026-11-01 | Metadata and Script Time Trial reruns; BRE Mastery `rotation_exhausted` re-check | `ranked-2`, `open-1`, `progress.md` |

Submission counts per mode (original attempts): Warmups 1 / 1 / 2 attempts (Mastery: first `expired`, retry 1 call);
Metadata 1 / 1 / 1; Mobile 1 / 1 / 1; BRE 1 / 2 POSTs (1 rejected) / 1; Script Analysis 40 / 1 / 1. The Single Solve numbers
are submission calls, taken from the journals.
