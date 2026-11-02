# Day 18 overview

**At a glance:** 5 challenges (one per lane, no web lane yet; the web challenge whose ID contains `day-18`
is a later release and is covered under its own day), 15 modes. By provenance label: 7 Independent, 8
Journal-recorded, 0 Local mark only, 0 Team, 0 n/a. Theme: a day of cumulative banks and decoy prompts. Each
lane reused the previous days' validated mapper and added one small new piece (a Morse/nibble/rail-fence record
trio, an MP4 geometry reader, a `SPLIT` marker for Google Apps Script, a WebAssembly branch and a few new HCFG
fields), and everything was accepted on the first full submission. The risk was never in the validator; it was
in the clocks and in our own submission tooling.

## Challenges

| Lane | Challenge | Technique | Single Solve | Time Trial | Mastery |
|---|---|---|---|---|---|
| Warmups | [Off the Rails](warmups_off-the-rails.md) | 3-rail rail fence; numbered-record chain extended with Morse-of-hex, a/b nibbles, rail fence | Journal-recorded | Journal-recorded | Journal-recorded |
| Metadata Forensics | [Frame of Reference](metadata-forensics_frame-of-reference.md) | MP4 `tkhd` vs `avc1` vs H.264 SPS crop inside the pooled evidence bundle | Journal-recorded | Independent | Independent |
| Mobile Android | [Encrypted Strings](mobile-android_encrypted-strings.md) | Declared XOR-over-Base64 asset; DEX is a decoy; Mastery = five earlier-day APKs with five codecs | Journal-recorded | Independent | Independent |
| Binary Reverse Engineering | [Flow State](binary-reverse-engineering_flow-state.md) | Static parsing of a d01-d18 recap bundle; HCFG records, WebAssembly carrier, `hcfg_storage` enum | Journal-recorded | Journal-recorded | Journal-recorded |
| Script Analysis | [Orphan Process](script-analysis_orphan-process.md) | Inert-marker descriptor decoding; cumulative bank; new Apps Script `SPLIT` marker | Independent | Independent | Independent |

Answer counts per mode (from the start responses and journals): Single Solve 1 + 1 + 1 + 216 + 1,194 = 1,413
fields; Time Trial 1 + 23 + 1 + 216 + 1,194 = 1,435; Mastery 88 + 115 + 5 + 864 + 5,970 = 7,042.

How to read the labels (each page gives the evidence):

- **Independent, Script Single Solve.** No `result.json`, but the leaf saves the `gm.py submit` output of the
  second call (`submission-attempt-02-output.txt`) reading `CORRECT`, and the attempt log records it. That is a
  saved graded response, so it counts as Independent under the brief. Script Time Trial and Mastery also have
  saved `CORRECT` outputs for the original untimed attempts, and `ranked-1/result.json` from later reruns.
- **Independent, Metadata and Mobile Time Trial and Mastery.** The original attempts (2026-10-20 in
  `progress.md`) are journal-recorded only. The label rests on saved `ranked-N/result.json` files, all CORRECT:
  Metadata 1 TT + 1 Mastery, Mobile 2 TT + 2 Mastery, Script 1 TT + 1 Mastery. Each rerun was one submission call.
- **Journal-recorded, everything else.** Warmups (all three modes), Binary (all three) and the Metadata and Mobile
  Single Solves have no result file, no saved response and no ranked rerun. Each journal records our own
  submission as CORRECT with the flag returned, and `challenges/day18/submission-attempts.md` /
  `submission-attempts-binary.md` and the `progress.md` rows (dated 2026-10-20) agree. The ledger holds CORRECT
  entries for seven of these eight (below), but the ledger is a call log, so, as on Day 11, we kept the label.
  Binary has a ranked solver (`tools/ranked/solvers/`) and a case list, but no `ranked-N` result in the leaf, so
  no rerun is recorded.

## What the day had in common

- **Cumulative banks.** Script re-asked the Day 15 bank (all 1,026 Day 15 ids, identical wording) plus 55 new
  Apps Script fields. Binary asked d01-d18 fact blocks, 12 per day, and the d01-d15 specimens matched the
  Day 15 and Day 17 archives byte for byte. Warmups Mastery extended the previous warmup's 15 record types to
  18. Metadata added one field (MP4 dimensions) to the 23-field bundle, with the ELF member still absent.
  The lesson on every lane was to diff the new start response against the last validated one before writing code.
- **Decoy prompts, again.** Binary promised call-graph reconstruction and injection hunting (no question concerns
  either), Script talked about process trees (every question is a descriptor field), Metadata hinted at crops,
  rotation and a spoofed device (the `tkhd`, `avc1` and SPS-cropped sizes all agreed on 320x180), Mobile's title
  and prompt point at string decryption (the only decryptor-looking method is dead code). Warmups was the
  exception, as on Day 11: "Three rails" named the cipher.
- **Clocked windows on plain `start`.** The brief says untimed, but several modes came back with deadlines anyway:
  Warmups Time Trial 30 s, Warmups Mastery about 105 s, Binary Time Trial 80 s, Binary Mastery about 265 s
  (04:39:36 to 04:44:01 UTC). Script, Metadata and Mobile reported `deadline: null` for the original attempts. The
  Warmups and Binary solves were therefore scripted into a single download-map-submit pass. Ranked windows on
  the reruns were 50 s (Time Trial) and 170 s (Mastery).
- **Mastery bundles are not Day 18 builds.** Mobile Mastery was five earlier-day APKs (Days 3, 7, 12, 13 and 18)
  with a different codec each, and Binary Mastery sets 2 and 4 swapped ELF members for PE builds. The same
  one-fixture-per-mode assumption would have failed in both.

### Validator and format traps seen across lanes

| Lane | Trap |
|---|---|
| Script | Apps Script `role`/`attack` took the descriptor's own declared value even where the step list pointed elsewhere (a `benign_utility` with `dispatch_process`, a `launcher` returning configuration). Accepted in the first full-map call. |
| Script | One-item `FOR /L ... DO CALL` Batch wrappers mapped to `call_subroutine` by structural signature, not the prompt's `for_once`; accepted in both Time Trial and Mastery. |
| Binary | `hcfg_storage` had no stated definition. The ordered rules (`base64`, `raw_hcfg_section`, `raw_const_array`, else `raw`) were accepted as written. `analysis_format` must be classified from magic bytes because Mastery mixed PE and ELF. A raw `HCFG` hit with a bad CRC is a decoy. |
| Metadata | Dimensions as `WxH` (generic "exact artifact-derived value" format text), PE machine and entry RVA as padded hex, MSI `ProductCode` with braces in upper case, OOXML external targets in ascending order. The synthesis `Files:` list is read from each question. All accepted first try, from conventions earlier days had confirmed. |
| Mobile | The codec and key are per asset, not per day: a hard-coded XOR key fails four of five Mastery sets. |
| Warmups | Record 17 (five-letter `a`/`b` groups) is hex nibbles, not Baconian letters. Submit only the slug, not the instruction sentence. |
| All lanes | Every answer value is a string; `incorrect` on a multi-answer call gives no per-field feedback (Script's first call returned `accepted` with `remaining: 64`, which is the useful signal). |

No answer was rejected on any lane, so Day 18 gave no new evidence about alternate spellings.

## Mistakes in our own tooling or process

Collected from the challenge pages and journals, with the evidence for each.

- **Orphan Process (Script), accidental full-map submission.** The second Single Solve call was meant to send
  only the 55 Apps Script fields. The id list was supposed to come from a temp file written by a heredoc Python
  snippet, but under Git Bash that `/tmp` was not the path the shell later read, so the file never existed.
  `gm.py submit --ids ""` selects every answer, so the call carried all 1,194 answers. It returned CORRECT,
  so nothing was lost beyond one submission, but the 55 new fields were never tested in isolation, which was
  the point of the plan. The journal's lesson is to keep scratch files inside the workspace. Evidence: the
  attempt log's own description of attempt 2 and `submission-attempt-02-output.txt` ("selected 1194 answers").
- **Submission cap shown as 60 for Script.** The saved Script outputs print "attempt N of 60", while the Warmups,
  Metadata and Mobile journals say "of 20" (the standing authorization). We could not find the authorization
  for 60 in the Day 18 files. Only 2 calls were used on Single Solve and 1 each on the others, so the cap was
  never in play, but the discrepancy is unexplained.
- **Submission ledger gaps.** `challenges/submission-ledger.json` has no entry for Warmups Single Solve, Script
  Single Solve, or the original Script Time Trial run (its only Time Trial entry is the ranked rerun), although
  the Script Time Trial journal says "Total attempts: 1 (ledger)". The Binary Time Trial entry is marked
  `reconstructed-after-ledger-race` in its own `ids_sha256` field. The lanes ran in parallel and their ledger
  entries fall within about eight minutes of each other, which matches the concurrent-write clobbering that
  `CLAUDE.md` records for Days 13-15; we could not confirm that cause for the missing entries. The per-leaf
  logs and `submission-attempts.md` are the fallback evidence.
- **Multi-delivery Mastery downloads.** `gm.py artifact` fetched only the first delivery for Binary (four) and
  Mobile (five) Mastery. Each lane wrote its own `download_deliveries.py` that read the token privately. The
  Day 11 overview records the fix (`--capability`/`--name`, commit `1d82b9d5`, 2026-10-23), made after these
  solves.
- **Binary Mastery set 2 failed inside the window.** The first mapper run hit a PE member and the ELF-only
  readers (and the missing `objdump`) broke on it. We wrote a COFF/PE reader and storage classifier inside the
  265 s window and submitted about 110 s after start. It worked, but it was the Single Solve mapper's ELF
  assumption, which the Day 15 and Day 17 archives would not have exercised.
- **Unrequested timed attempts.** The Binary Time Trial journal notes the player had not yet authorized timed
  ranking, yet `start` without `run_for_time` returned an 80 s clock; the Warmups Time Trial journal says its
  attempt "may therefore count as a ranked/timed attempt". Both were completed in-window; neither journal
  records a hall-of-fame check.
- **Interpreter isolation.** The Metadata extractor needs `pefile` and `olefile`, and the Binary d01
  disassembly fallback needs `capstone`; both are hidden by `python -I` and need `-P`. Noted on the pages and
  already in `CLAUDE.md`.
- **Day order.** The Metadata Time Trial and Mastery journals build on "Day 19 validator-confirmed
  conventions", so the lane was not solved strictly in release order. This was by design, not an error, but it
  matters when reading the Day 18 and Day 19 pages together.

## Tooling and process changes around this day

Day 18 has no `_run/` folder, so there are no lane logs, timelines or librarian notes. From the git history:

- `284a8a2b` (2026-10-22): "Add Day 18 challenge work". This is the first commit of the Day 18 leaves; the
  solves themselves date to 2026-10-20 per `progress.md` and the ledger's first entries.
- `3bcba4e6` (2026-10-22): technique library. The Day 18 entries (rail-fence numbered codec, MP4 geometry, the
  Apps Script `SPLIT` decoder, Encrypted Strings, Flow State) were promoted from the journals' "Library promotion"
  sections; the Script `SPLIT` branch was later folded into the consolidated descriptor decoder.
- `34e7f1a0` (2026-10-29): Day 17-20 ranked solvers and reruns for the non-web lanes. This added the four
  per-challenge solvers for Day 18 (Binary, Metadata, Mobile, Script; Warmups has none), the first `ranked-1`
  results for Metadata, Mobile and Script, and the case list for Binary. The Script rerun used the shared
  descriptor-bank solver with the `T1105` staged-chain attack style, and `result.json` records 1 call per mode
  with the `staged-chain-attack` group on variant 0.
- `38668151` (2026-11-01): the top-30 gap-closure sweep. For Day 18 it added Mobile `ranked-2` for Time Trial
  and Mastery.

Ranked reruns were bragging-rights only and are not what any Independent label above rests on except where
stated; `result.json` `t_total` was 1.23 to 3.26 s for Metadata and Mobile, 2.06 s (Script TT) and 4.11 s (Script
Mastery).

## Effort

Submission calls, from the journals, attempt logs and `submission-attempts*.md`:

| Lane | Single Solve | Time Trial (original) | Mastery (original) | Ranked reruns that went CORRECT |
|---|---|---|---|---|
| Warmups | 1 | 1 | 1 | none |
| Metadata | 1 | 1 | 1 | 1 TT, 1 Mastery |
| Mobile | 1 | 1 | 1 | 2 TT, 2 Mastery |
| Binary | 1 | 1 | 1 | none recorded |
| Script | 2 (1,130-field subset accepted, then full map CORRECT) | 1 | 1 | 1 TT, 1 Mastery |

The ledger's timestamps for the original attempts run from 21:38 to 21:46 (local ledger time) on 2026-10-20, so
all twelve ledger-recorded Day 18 original submissions fit in about eight minutes. That is a statement about
submission time only; the files do not record how long each mapper took to adapt.
