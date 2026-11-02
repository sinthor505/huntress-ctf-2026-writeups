# Day 12 overview

**At a glance:** 5 challenges (one per lane, no web lane yet), 15 modes. By provenance label: 3
Independent, 12 Journal-recorded, 0 Local mark only, 0 Team, 0 n/a. Theme: a bank-and-recap day. Warmups
chained 12 codecs, Binary re-asked eleven earlier days plus one new block, Script Analysis added a new
language to a 95-specimen bank, and the two small lanes each added one new reader (a .NET MVID, a `hex`
asset codec). Everything was solved offline from the delivered bytes. The cost was almost entirely in the
Script lane's validator probing and in the short timed windows, which the Script and BRE lanes overran.

## Challenges

| Lane | Challenge | Technique | Single Solve | Time Trial | Mastery |
|---|---|---|---|---|---|
| Warmups | [Back at You](warmups_back-at-you.md) | Character-order reversal; 12-codec numbered fragment chain in Mastery (58 blobs) | Independent | Independent | Independent |
| Metadata Forensics | [Managed Expectations](metadata-forensics_managed-expectations.md) | .NET `Module.Mvid` from CLR metadata (`#~` Module row, `#GUID` heap, little-endian GUID) | Journal-recorded | Journal-recorded | Journal-recorded |
| Mobile Android | [SQLite Assumptions](mobile-android_sqlite-assumptions.md) | Declared `hex` codec in `assets/huntress_puzzle.json`; SQLite theme is a decoy | Journal-recorded | Journal-recorded | Journal-recorded |
| Binary Reverse Engineering | [Pascal's Wager](binary-reverse-engineering_pascals-wager.md) | Carving a CRC-checked, XOR-encoded `HCFG` record from a stripped Free Pascal ELF; cumulative d01-d12 recap (120 fields) | Journal-recorded | Journal-recorded | Journal-recorded |
| Script Analysis | [AutoIt Takes Two](script-analysis_autoit-takes-two.md) | Static descriptor reconstruction over 99 specimens (855 fields); AutoIt added; family-by-family ATT&CK/Batch-control probing | Journal-recorded | Journal-recorded | Journal-recorded |

How to read the labels (each page gives the evidence):

- **Independent (Warmups, all three modes).** Each mode leaf holds a saved `submission-attempt-01.json`
  with the Game Master response (`http: 200`, `result: CORRECT`). Warmups was CORRECT on call 1 in all
  three modes.
- **Journal-recorded (the other twelve).** The journals record our own submission and `CORRECT`, and
  `progress.md` has a `Solved` row for each (all dated 2026-10-20). No `result.json`,
  `accepted_plan.json` or raw response file survives for them. Script Analysis and BRE do keep a
  `submission-attempt-log.json` whose last entry reads `CORRECT`; it is a log we wrote locally, not the
  verbatim response, so we did not promote those to Independent.
- **No ranked rerun backs any mode.** Seven of the fifteen leaves hold a `ranked-1/result.json`, and every
  one reads `practice_only` (Warmups TT and Mastery, Metadata TT and Mastery, Mobile TT and Mastery, BRE
  TT). The other three timed modes (Script TT, Script Mastery, BRE Mastery) have no ranked folder. See
  "Pools already spent" below.
- **No team solves recorded.** No `team_solved.json` exists in any Day 12 leaf, and the `progress.md` rows
  are our own.

## What the day had in common

- **Cumulative-day structure.** Warmups Mastery used 12 numbered fragments, the record number fixing the
  codec, ending in the plain reversal that is the Single Solve transform. Binary's 120 fields were ten per
  day for d01-d12, so 110 re-asked earlier shapes and only the d12 block (an `HCFG` record in a Free Pascal
  ELF) was new. Script Analysis answered 855 questions over a bank in which only four AutoIt files were new
  (44 of the 855 fields). The reusable result in each case was a general reader, not a one-day extractor.
- **Prompts as flavor text.** Mobile's "reconstruct authorization from a SQLite cache" has no SQLite code
  behind it, Script's "decide which pairing is genuine" has no matching question, and Binary's "Pascal is
  betting you won't read its record layouts" needs no Pascal-specific analysis (the question text states the
  record layout). Only the Warmups prompt ("started at the wrong end") and the Metadata prompt
  (module identities distinct from names and versions) pointed at the real mechanism.
- **Time Trial and Mastery changed fixtures and scale, not mechanism.** Mastery bundles arrived as a ZIP of
  58 blobs (Warmups), nested ZIPs (Metadata: two, Script: two sets of 99), three separate APKs (Mobile) and
  three AES ZIPs (BRE, 360 fields, with set 2 delivering PE specimens for d02/d04/d10).
- **Short windows on the legacy-style route.** Every Time Trial and Mastery start response already read
  `unique_attempts_remaining: 0` and `hall_of_fame_eligible: false` but still carried a server deadline. The
  deadlines fall within about 15 minutes of each other on 2026-10-20 (UTC, from the start responses):

  | Mode | Deadline | Window stated in the page or journal |
  |---|---|---|
  | Warmups TT | 18:59:24 | 30 s |
  | Metadata TT | 19:00:23 | about 50 s |
  | Warmups Mastery | 19:00:30 | 75 s |
  | Metadata Mastery | 19:01:30 | 80 s (started 19:00:10) |
  | Mobile TT | 19:05:18 | 50 s |
  | Mobile Mastery | 19:06:36 | 110 s |
  | BRE TT | 19:08:17 | 80 s |
  | Script TT | 19:09:31 | 50 s |
  | BRE Mastery | 19:11:48 | 200 s |
  | Script Mastery | 19:14:51 | not stated |

  The windows overlap (Metadata TT's deadline is earlier than Warmups Mastery's), so the lanes were worked
  in parallel. Warmups, Metadata and Mobile finished inside their windows. Script TT, Script Mastery and BRE
  Mastery did not, and were completed on untimed continuations (see below). The Single Solve assignments
  for these challenges show a start time of 2026-10-12 in the journals, eight days before the recorded solve
  date; we did not determine why from the files.

### Validator and format traps seen across lanes

| Lane | Trap |
|---|---|
| Metadata | An answer ID the attempt does not contain is judged `incorrect`, not rejected as malformed (the Time Trial reused the Single Solve ID). The MVID must be formatted with the first three GUID fields little-endian; the `#GUID` heap index is 1-based. |
| Mobile | The declared codec is `hex`, which the library inspector did not support at the time and reported as `supported: false`. The client secret in the DEX is a decoy beside the asset-derived `proof`. |
| Script | Two Python staged-chain ATT&CK fields mapped from the graph as `T1105` were rejected and took `T1059.006`, while Bash and Node.js staged chains took `T1105` in the same attempt. The descriptor `attack` field beat the graph reading for the v3 and host files, including AutoIt (`T1105` for the downloader). Batch `*-control` answers varied per specimen and per set: `call_subroutine` for the benign GOTO dispatch and several one-item wrappers, a rejected `for_once` guess for the dropper, and a direct reconstruction for set 2's benign control. |
| BRE | `hcfg_offset` is the file offset of the `HCFG` marker, not a virtual address or a `.data`-relative offset. Member "1" per day must be resolved by the `dNN--` prefix because the archive holds two d03 members. Mastery set 2 swapped ELF for PE on d02/d04/d10, so format has to come from magic bytes. |
| Warmups | The answer is the last non-empty decoded line of the final record, split on the last colon. No library entry covered reversal when the day started. |
| All batch submissions | `incorrect` on a multi-answer call carries no per-field feedback, so cause isolation costs calls (Script Single Solve: 11, Time Trial: 16, Mastery: 26). Every value must be a string. |

CLAUDE.md carries the lasting versions: the staged-chain `attack` map by day and language, the Batch
control guidance, and the Day 12 note that ranked pools for this day were spent.

## Mistakes in our own tooling or process

Collected here from the challenge pages, with the evidence for each.

- **Managed Expectations (Metadata).** The first Time Trial map carried the Single Solve's answer ID
  (`d12-dotnet-001-...`) instead of the fresh `d12-dotnet-002-...`, so call 1 was `incorrect` and the
  corrected map was CORRECT on call 2. The Time Trial journal records it as a mapping error, and the Single
  Solve journal's lesson is to assert `set(answer_map) == set(answer_ids)` before submitting.
- **AutoIt Takes Two (Script).**
  - All three modes opened with a full-map submission that returned `incorrect` with no field feedback,
    spending a call before any family-level probing. The Single Solve call history shows what the partition
    (family, then suffix) then cost: 11 calls.
  - The two Python ATT&CK fields were mapped from the graph as `T1105`, making the 126-field Python call
    `incorrect`. The fix came from cross-checking Day 11's accepted Python map. The Single Solve journal
    says the workspace it ran in held no `CLAUDE.md`, so the validator lessons recorded there were not
    available on first read.
  - Time Trial and Mastery both ran past their windows. In Time Trial the 15-ID PowerShell call returned
    `expired`; in Mastery the deadline (19:14:51) passed before the work finished and recon retried the
    expired assignment at 19:15:16 (the attempt log has no `expired` entry for Mastery). Both retries returned the same archive hash and IDs with `deadline: null`, so the
    CORRECT results are on untimed continuations of the same bytes, not within the timed window.
  - Mastery spent calls 7-9, 12, 14 and 17 on `incorrect` Batch-control probes (the dropper's `for_once`
    guess among them). The first 20 calls left 1,266 of 1,710 IDs unresolved. The remaining fields were
    closed in calls 21-26 by a higher-reasoning continuation, under the 30-call authorization for this exact
    mode; the journal says no further submissions were needed.
  - The shared submission ledger showed 10 entries for Single Solve while 11 calls were made, because a
    concurrent write omitted the rejected initial Python-family request. The mode's own attempt log kept all
    11. CLAUDE.md later records ledger writes as lock-protected and atomic after a similar clobber on Days
    13-15; we did not establish that this exact Day 12 omission was the trigger.
- **Pascal's Wager (BRE).** Mastery set 1 was submitted alone first (call 1, `accepted`), then sets 2 and 3
  returned `expired` (calls 2 and 3) because the 200 s clock had run out. After an untimed continuation,
  sets 2 and 3 were accepted (calls 4 and 5) and set 1 was resubmitted, returning `CORRECT` for 360 of 360
  on call 6. The Mastery completion is therefore an untimed one. The page's lesson is to run all deliveries
  concurrently and submit as each finishes.
- **SQLite Assumptions (Mobile).** Mastery used two calls: a one-ID probe (`set1.answer`, `accepted`,
  `remaining: 2`) and then the other two. The Single Solve journal's own review says one call would have
  sufficed since every value already passed its format check.
- **Warmups and Mobile (library gaps).** No library entry existed for reversal (Warmups), and the mobile
  asset inspector reported the `hex` codec as unsupported; both were handled by hand on the day and
  promoted afterwards (`WARMUP-TEXT-REVERSAL-001`; the `hex` branch of the puzzle-asset inspector).
- **Ranked rerun, Warmups Time Trial (`ranked-1`).** The result file says it was "recorded by hand": the
  first run with the old pass script got a practice payload and the solver failed before any submit. Its
  `pass.log` blames an encrypted ZIP member named `resource-sample.ps1`, which is not a Warmups fixture
  (that file name belongs to the Script Analysis bank). We could not reconcile the log with the leaf and
  treat the pass as a no-op either way.

## Pools already spent

The Days 11-15 ranked sweep (2026-10-25) found Day 12's timed pools exhausted: warmups, metadata, mobile and
BRE Time Trial returned a 200 `practice:true` payload with `unique_attempts_remaining: 0`, and script
Time Trial and Mastery plus BRE Mastery returned 409 `retry_variants_unavailable` (CLAUDE.md and
`tools/ranked/PROCESS.md`). Nothing was issued, `ranked_pass.py` wrote `practice_only` for the former, and
the Day 12 solvers (`tools/ranked/solvers/<challenge-id>/`, cases in `tools/ranked/cases/day12.json`) were
built and validated offline only. This is why the Day 12 labels rest on 2026-10-20 evidence alone, and why
CLAUDE.md scores the day 0 of 10 in that sweep. We did not find in the files why the pools were already at
zero when the 2026-10-20 attempts started.

## Tooling and process changes around this day

The repository history starts on 2026-10-22, two days after the recorded solves, so the original work has no
commits of its own. What the history shows:

- `62396e04` (2026-10-22): first commit of the Day 12 work, 179 files: the journals, extractors, mappers,
  per-attempt submission logs and the start responses.
- `c9906b50` (2026-10-25): the Days 11-15 ranked pass. Its message records Day 12 as having no timed attempts
  left, and adds the per-challenge solvers (including the four Day 12 solvers), `practice_only` handling in
  `ranked_pass.py`, and the PROCESS.md/CLAUDE.md entries for the two spent-pool signals.
- Library promotions credited in the Day 12 journals (the Script journals date their curation sweep to
  2026-10-20; the others carry no date):
  `META-DOTNET-MVID-001` (the CLR Module.Mvid reader), `WARMUP-TEXT-REVERSAL-001`, the `hex` codec in the
  mobile puzzle-asset inspector, the AutoIt marker and duplicate-literal check folded into the descriptor
  decoder, and the Script family/Batch-control playbook entries (`SCRIPT-TECHNIQUE-C101/C103/C104/C105`).
  The BRE journals reuse the existing HCFG, PE/CLR and JAR entries rather than adding new ones.

## Effort

Submission calls per mode, from the journals and attempt logs (72 in total). Journals state a 30-call
authorization for the exact Script, BRE, Metadata and Mobile modes.

| Lane | Single Solve | Time Trial | Mastery |
|---|---|---|---|
| Warmups | 1 | 1 | 1 |
| Metadata | 1 | 2 (ID slip, then CORRECT) | 1 |
| Mobile | 1 | 1 | 2 |
| BRE | 1 | 1 | 6 (two `expired`) |
| Script | 11 | 16 (one `expired`) | 26 |

Script Analysis took 53 of the 72 calls, almost all of them validator probing after a batch-level
`incorrect`; the logged `expired` calls are one in Script Time Trial and two in BRE Mastery. Warmups,
Metadata and Mobile were CORRECT within one or two calls per mode. No per-lane wall-clock time is recorded
beyond the deadlines in the table above.
