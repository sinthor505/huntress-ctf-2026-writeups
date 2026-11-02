# Day 15 overview

**At a glance:** 5 challenges (one per lane, no web lane yet), 15 modes. By provenance label: 13
Independent, 2 Journal-recorded, 0 Local mark only, 0 Team, 0 n/a. Theme: a day of cumulative recap
bundles with misleading prompts. The new technique in each lane was small (ROT47, OOXML `.rels`, a JNI
getter, a stripped/symbolized ELF pair, Batch control classification), and nearly all the effort went
into per-variant details: which file a question names, which spelling the validator accepts, and which
earlier-day answers must not be carried over.

## Challenges

| Lane | Challenge | Technique | Single Solve | Time Trial | Mastery |
|---|---|---|---|---|---|
| Warmups | [Room Forty-Seven](warmups_room-forty-seven.md) | ROT47; 15-codec numbered record chain in Mastery | Independent | Independent | Independent |
| Metadata Forensics | [A Word to the Wise](metadata-forensics_a-word-to-the-wise.md) | OOXML external relationship targets, validator-defined (sorted) order; 20-field evidence bundle | Independent | Independent | Independent |
| Mobile Android | [Native Library Secrets](mobile-android_native-library-secrets.md) | Static JNI getter to `.rodata` string in a bundled `.so`; per-APK codec dispatch in Mastery | Journal-recorded | Independent | Independent |
| Binary Reverse Engineering | [Stripped](binary-reverse-engineering_stripped.md) | Cumulative recap bundle (`d01` to `d15`): ELF/PE facts, `.symtab` presence, HCFG records | Journal-recorded | Independent | Independent |
| Script Analysis | [Crumb Trail](script-analysis_crumb-trail.md) | Static descriptor reconstruction (116 specimens, 1,026 fields per set); Batch control classification by structure | Independent | Independent | Independent |

How to read the labels (each page gives the evidence):

- **Independent, Single Solve (Warmups, Metadata, Script).** The leaf holds a saved submission response
  reading CORRECT (a `.txt` or `.json` response for Warmups and Metadata; `submission-attempt-31-output.txt`
  for Script).
- **Journal-recorded, Single Solve (Mobile, BRE).** The journal and `progress.md` record our own CORRECT, but no
  response was saved. The BRE leaf has a `submission-attempt-log.json` with one `CORRECT` entry. It is a
  local log, not a saved server response, so we kept the weaker label.
- **Independent, Time Trial and Mastery (all ten).** Every one has at least one saved `ranked-N/result.json`
  reading CORRECT, in one submission call each. The original 2026-10-20 attempts are recorded in the
  journals and `progress.md` (Metadata and Script also keep per-attempt response files or logs), and those
  are the attempts the pages describe. Ranked reruns per mode: Warmups 1 TT + 2 Mastery, Metadata 2 + 2,
  Mobile 1 + 1, BRE 6 + 5 result files (including two recoveries, below), Script 1 + 2.
- Nothing on this day was team-scored. The `web-exploitation-day-15-fetch-quest` entries that appear in the
  submission ledger belong to a later release day (Day 24), not here.

## What the day had in common

- **Mastery replayed earlier days.** Warmups Mastery was 73 blobs of 15 records, with record N using Day N's
  codec. Mobile Mastery bundled five APKs with case ids `MOB-03` to `MOB-15`, only the last using the native
  library. BRE carried `d01` to `d14` as recap blocks beside the new `d15`. The Script decoder and mapper came
  from the Day 14 bank. The new piece was always a single block.
- **Prompts were decoys or only loosely tied to the graded fields.** Warmups was literal ("the punctuation went
  for a spin" meant ROT47). The Metadata prompt ("find the document reaching somewhere it should not")
  suggested one suspicious link, but the validator wanted every external target. The BRE prompt asks to
  "reconstruct the stripped functions", yet no question needs a disassembler. The Script prompt about browser
  credential stores was flavor, and we never found a question covering its "store that was never there".
- **Short windows, several lost to expiry.** Ranked windows: 30 s (Warmups TT), 50 s (Metadata, Mobile and
  Script TT), 80 s (BRE TT), 90 s (Warmups Mastery), 170 s (Metadata, Mobile, Script Mastery) and 320 s (BRE
  Mastery). Metadata TT, Metadata Mastery and Script TT each lost timers to `expired` before a correct map
  was ready, and the Metadata solves finished on timed retry variants. The Script Time Trial and Mastery
  solves ran on untimed attempts, which is what allowed group testing.
- **Fixtures changed per variant, not the mechanism.** Every `retry` issued a new archive (the Script TT journal
  lists seven archives) and earlier acceptances did not carry over.

### Validator and format traps seen across lanes

| Lane | Trap |
|---|---|
| Warmups | None; the plain slug (last non-empty decoded line) was accepted in all modes. Per-record decoding must follow the record number. |
| Metadata | OOXML external targets must be a JSON-array string in ascending lexical order, including the generic-looking link. Synthesis `Files:` lists differ per set (Mastery set 4 named a JPEG where the others named a PNG). `incorrect` is batch-level only. |
| Mobile | Mastery APKs use their own declared codecs (`base64`, `caesar`, `native`); dispatch on each asset. DEX strings are decoys. |
| BRE | PE vs ELF varies per Mastery set for `d02`/`d04`/`d10` (classify by magic bytes); `d15` machine is the integer `62`; accepted spellings this day were `I386`, `PE32_CLR`, `AMD64` for PE labels and `elf` for `d09`/`d11` format. |
| Script | The prompt's "one-item `FOR /L ... DO CALL` is `for_once`" is rejected; the validator took `call_subroutine`. Batch `*-control` answers follow each specimen's structure, and per-ID overrides from one mode or variant do not carry over. |

CLAUDE.md carries the lasting versions: the Day 13 and "Batch controls are structural (Days 13-15)" bullets
(Day 15 Time Trial and Mastery were completed that way), the Mobile bullet that Mastery bundles are earlier-day
APKs, and the BRE spelling lessons.

## Mistakes in our own tooling or process

Collected here from the challenge pages, with the evidence for each.

- **Crumb Trail (Script).**
  - Single Solve spent attempt 1 on the full 1,026-field map and attempt 10 on a 105-field mixed group; both were
    `incorrect` with no per-field feedback. It finished on attempt 31 after splitting into small groups.
  - Time Trial reused the three Single Solve per-ID Batch overrides on fresh variants. Attempts 13, 14, 18 and 20
    were `incorrect` for that reason; one freshly generated control was a bare `set` that is truly `direct`.
  - Three Time Trial submissions returned `expired` (attempts 15, 16 and 21); the journal notes that its
    earlier belief that late submissions still worked did not hold.
  - Mastery's structure-signature regex matched `if defined X set ...` but not `if defined X call set ...`, so
    that specimen landed in the nine-ID `direct` class. Two grouped submissions failed and seven singleton
    submissions followed. The regex was fixed in the driver and the library helper.
  - The Time Trial driver passed a 1,017-ID `--ids` list on the Windows command line, which exceeded the length
    limit before anything was sent; attempt 25 was submitted by hand from the saved answer file and the driver
    now omits `--ids`.
  - Ledger count: Single Solve 31, Time Trial 25 in the original work (the journal says "player-authorized
    extension to 40"), Mastery 14. We could not determine from the files how the Single Solve count past 30
    was authorized; its journal records batches of 10. The Time Trial journal's diagnostics text says "attempts
    1-10 were spent before" while its table starts at attempt 11; we could not reconcile that numbering.
- **A Word to the Wise (Metadata).** A later helper download overwrote the bytes of one Time Trial variant, so
  the cause of that variant's rejected full-map submission could not be pinned down. The Mastery mapper
  hashed `panel-001.png` for the synthesis field on set 4, where the question names a JPEG; five fields were
  rejected and the timer expired before the cause was found. The corrected re-derivation went CORRECT on a retried
  variant (attempt 11).
- **Native Library Secrets (Mobile).** The first Time Trial start was untimed because the helper was called
  without the timed flag, and a `retry --timed` while it was open returned 409 `attempt_active` (nothing
  consumed). The untimed and ranked downloads shared a file name, so the untimed APK was lost and its map
  could not be re-verified.
- **Stripped (BRE).** `gm.py artifact` fetched only the first Mastery archive; a `--from-start` option was
  added so every `artifacts[]` entry is downloaded and hash-checked (journal entry in the TT and Mastery
  journals). Later ranked reruns lost or stranded attempts to the harness: one Mastery rerun on 2026-10-29
  (ranked-2) got HTTP 500 on the download after the attempt was issued, and a rerun ran under a Python mode
  that hid `pyzipper` (a Time Trial pass, `ranked-6`, 2026-11-01; and a Mastery rerun, per `PROCESS.md` and
  commit `0d922736`). The attempts were already live, so they were recovered in place (`ranked-2-recover`,
  `ranked-4-recover`) or the pass was simply lost (`ranked-6`, no result file).
- **Room Forty-Seven (Warmups).** Nothing went wrong beyond the library lacking a ROT47 entry; the day added one.

## Tooling and process changes around this day

The first Day 15 commit is `ddc1ba1d` (2026-10-22); the solves it records are dated 2026-10-20 in `progress.md`.
From the journals, the original timed attempts clustered between 2026-10-20 23:37 and 23:59 UTC (Metadata TT
began 23:37; BRE TT and Mastery deadlines 23:44 and 23:49; Mobile TT and Mastery 23:53-23:57; Script TT's last
variant 23:59). Script Mastery was started later (server clock 2026-10-21 04:07 UTC).

- **Library entries added or updated by this day's work** (named in the pages): WARMUP-ROT47-001; OOXML
  relationships (META-OOXML-RELATIONSHIPS-B104) and SHA-256 order (META-SHA256-ORDER-B105) for Metadata;
  MOBILE-ANDROID-JNI-001; BRE-RECAP-B105; and for Script SCRIPT-TECHNIQUE-004 with
  `batch_control_signature.py`, the class-level group-testing method behind the Time Trial and Mastery solves.
- `c9906b50` (2026-10-25): the Days 11-15 ranked pass. Day 15 went 10 of 10 Time Trial and Mastery modes
  CORRECT in one call each, except the Script Time Trial, which was blocked by the 20-submission ledger cap
  (ledger 25) before anything was issued. Per `tools/ranked/PROCESS.md`, the player then authorized `--cap 40`
  and the rerun went CORRECT in one call (`time-trial/ranked-1`; CLAUDE.md records rank 5 on the speed
  scoreboard, 1108 ms; the local `t_total` was 1.99 s). The ledger now reads 26 for that mode.
- `0fdd8957` (2026-10-28): team-attributed reruns of Days 13-15 (`run_modes.sh`). Day 15 BRE Mastery's HTTP 500
  was recovered with `finish_open.py` (`ranked-2-recover`).
- `0d922736` (2026-10-29): reauthorized BRE Mastery rerun; the `pyzipper` problem above recovered in place; the
  scoreboard row for the team improved from rank 48 (67,115 ms) to rank 36 (35,196 ms), per the commit message.
  `PROCESS.md` documents the recovery pattern.
- `6af31628` (2026-10-31), `38668151` and `f0e20aa1` (2026-11-01): the gap-closure sweeps. The Day 15 results
  from them are the later BRE ranked folders, the second Warmups and Script Mastery reruns, and the
  `ranked-6` failed pass.

## Effort

Submission calls from the journals, attempt logs and `challenges/submission-ledger.json` (ledger counts include
the ranked reruns):

| Lane | Single Solve | Time Trial (original) | Mastery (original) | Ledger totals (SS / TT / Mastery) |
|---|---|---|---|---|
| Warmups | 1 | 1 | 1 | 1 / 2 / 3 |
| Metadata | 3 | 8 (of 10 allowed) | 11 | 3 / 10 / 13 |
| Mobile | 1 | 1 (untimed first variant) | 1 | 1 / 3 / 2 |
| BRE | 1 | 1 | 1 | 1 / 6 / 6 |
| Script | 31 | 25 | 14 | 31 / 26 / 16 |

Ranked-rerun passes took (`t_total` in `result.json`): Warmups 1.4-2.2 s, Metadata 1.6-3.1 s, Mobile 1.4-1.7 s,
Script 2.0-4.7 s, BRE Time Trial 4.9-10.6 s and BRE Mastery 17-21 s (dominated by the large encrypted
archive downloads).
