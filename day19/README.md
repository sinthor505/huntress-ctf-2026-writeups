# Day 19 overview

**At a glance:** 8 challenges (five static-artifact lanes plus the first three Web Exploitation challenges),
24 modes. By provenance label: 16 Independent, 8 Journal-recorded, 0 Local mark only, 0 Team, 0 n/a.
Theme: recap bundles and decoy prompts again, plus the debut of the web lane. The five artifact lanes
reused earlier machinery (a declared APK codec, the cumulative binary bundle, the Day 15 script-descriptor
mapper, the evidence-bundle extractor, the numbered warmup codec chain) and the cost was in one new field
or placement per lane. The three web challenges were small, one-recipe-each puzzles whose Mastery bundles
all recycled the same `robots.txt` backup-disclosure target.

## Challenges

| Lane | Challenge | Technique | Single Solve | Time Trial | Mastery |
|---|---|---|---|---|---|
| Warmups | [Little Things](warmups_little-things.md) | Hex of UTF-16LE with BOM; numbered-record codec chain in Mastery (4 new codecs) | Journal-recorded | Independent | Independent |
| Metadata Forensics | [Little Endian on the Prairie](metadata-forensics_little-endian-on-the-prairie.md) | ELF `e_machine` honouring `EI_DATA`, zero-padded hex; mixed-format evidence bundle | Journal-recorded | Independent | Independent |
| Mobile Android | [Reflection Maze](mobile-android_reflection-maze.md) | APK puzzle-asset codec (`reflection`, `base64`, `xor`, `split`); DEX is a decoy | Journal-recorded | Independent | Independent |
| Binary Reverse Engineering | [Jack in the Box](binary-reverse-engineering_jack-in-the-box.md) | Static unpack of a `PK19` XOR/ROR overlay; Day 1-19 recap bundle (228 fields) | Journal-recorded | Independent | Independent |
| Script Analysis | [Sharp Practice](script-analysis_sharp-practice.md) | Inert descriptor markers in two new placements (Apps Script SPLIT, ASPX string literal); rubric mapping | Journal-recorded | Independent | Independent |
| Web Exploitation | [Back Up a Minute](web-exploitation_back-up-a-minute.md) | `robots.txt` Disallow path with a `.json.bak` backup | Journal-recorded | Independent | Independent |
| Web Exploitation | [Permission to Slip](web-exploitation_permission-to-slip.md) | IDOR via an activity feed that leaks a cross-workspace id | Journal-recorded | Independent | Independent |
| Web Exploitation | [Query Me This](web-exploitation_query-me-this.md) | Paren-wrapped SQL injection, error and empty result look the same | Journal-recorded | Independent | Independent |

How to read the labels:

- **Journal-recorded (all eight Single Solves).** Each Single Solve journal records our own submission as
  CORRECT, and `progress.md` has a "Solved" row for each (2026-10-20 for the five artifact lanes, 2026-10-22
  for the three web challenges). No Single Solve leaf holds a `result.json`, `accepted_plan.json`,
  `manual_solve.json` or a submission-response file. Six leaves have a `submission-attempt-log.json` (a
  hand-kept log) and the three web leaves have an `accepted_map.json` holding only the proof string. Neither
  is a saved graded result, so we did not count them as one.
- **Independent (every Time Trial and Mastery).** The original attempts are journal-recorded (artifact lanes
  2026-10-20, web 2026-10-23, all untimed, per `progress.md`). The label rests on saved `result.json` files,
  all `CORRECT`, from later reruns: ranked reruns for every mode (Metadata Mastery twice, Mobile Time Trial
  twice) and, for the web lane, an untimed rehearsal `result.json` as well. Each rerun was 1 submission call.
  Ranked reruns were bragging-rights runs for the public speed board, not the prize basis.
- **No team solves.** Nothing on this day carries `team_solved.json`, and no `progress.md` row says
  `Team solve (not ours)`.

## What the day had in common

- **The web lane starts here, and its numbering lags.** The three web IDs are `web-exploitation-day-01-...`,
  `-02-` and `-03-` (the lane's own curriculum numbering) but they released on Day 19, so the leaves live under
  `challenges/day19/`. The saved web start responses carry no `day` or `curriculum_day` field at all, so the
  folder cannot be recovered from the response and has to come from where the challenge was listed. There is
  no artifact: `start` returns live `targets[]` (1 for Single Solve and Time Trial, 2 for Mastery) and one
  `proof-N` answer per target, `exact_text` with `trim`. Time Trial and Mastery originals came back with
  `deadline: null` and `hall_of_fame_eligible: false`, but the live target's response headers carried a
  7,200,000 ms (2 h) clock (recorded in the Back Up a Minute and Query Me This pages).
- **One recipe per web challenge, but the recipes recur across bundles.** Every Mastery had two targets. Back
  Up a Minute: two backup-disclosure targets. Permission to Slip: one backup-disclosure and one IDOR. Query Me
  This: one backup-disclosure and one paren-wrapped SQLi. So a bundle's second class is not the challenge's name,
  and each target has to be reconned on its own.
- **Cumulative recap structure in the artifact lanes.** BRE re-asked 12 fields per day for Days 1-19 (228); Script
  re-asked the Day 13-15 families (136 specimens, 1,249 ids); Metadata Time Trial and Mastery re-asked the fields
  from Days 4-19; Mobile Mastery delivered five APKs themed on earlier days; Warmups Mastery records 1-15 were
  codecs already seen. Only one thing per lane was new: the BRE `PK19` overlay, two Script marker placements,
  the ELF `e_machine` field, the Morse-hex / Baconian-nibble / rail-fence / UTF-16 warmup records, and the
  `reflection` codec name.
- **Titles and prompts that point the wrong way.** Reflection Maze talks about reflective dispatch; no reflection
  exists at runtime and the answer is the asset's `data`. Jack in the Box spelled out the unpacking recipe, so the
  work was exact implementation. The Metadata prompt's "rebuild deployment groups" shows up only as a
  sort-by-SHA-256 synthesis question. Warmups was the honest one: "the little one goes first" names the byte order.
- **Fixtures moved between modes.** Script Time Trial changed 340 of 1,249 answers against Single Solve; BRE Time
  Trial's d01 member was a stripped ELF; BRE Mastery sets 1, 3 and 5 delivered d02/d04/d10 as PE32+ instead of ELF;
  web instances were re-skinned with new endpoint names (`/lookup`, `/explore`, `/search`; `/api/history`,
  `/api/events`, `/api/timeline`) and fresh hashed paths.

### Validator and format traps seen across lanes

| Lane | Trap |
|---|---|
| Metadata | `e_machine` as `0x3e` was `incorrect`; the zero-padded four-digit `0x003e` was `CORRECT` on attempt 2. This is the only rejected submission of the day. Earlier days' padded PE/Mach-O fields had pointed that way. |
| Metadata | Per-set Mastery variation: which of the note, copy and roster is asked about, and whether by filename, size or digest, changes per set. |
| Script | The ASPX marker sits inside an escaped C# string literal, so a line-wide regex swallows the `\n...` tail and breaks `fromhex`; match only the payload alphabet. The Apps Script `encoded_` array is a prefix-less copy, not an independent check. |
| Script | Descriptor fields (`role`, `attack`, `adapter`, `next_language`) are independent of `steps`; Batch controls were classified structurally and the one-item `FOR /L ... CALL` wrapper was `call_subroutine`. All three modes were accepted on call 1. |
| Binary | Overlay offset uses the last `PK19` (`rfind`); overlay size is file length minus that offset, not the packed length; a stripped d01 turns the symbol and disassembly fields into empty lists and `recover_code` into `none`. |
| Binary | Accepted label spellings went through on the first submission of each mode, so no alternates were ever tested. |
| Mobile | `client_secret` in the WebView URL is a decoy; only the asset `data` is graded. Mastery keys results by delivery number and `artifacts[]` has to be fetched per capability. |
| Warmups | The slug is the bare last token; fragment boundaries do not line up with codec boundaries, so only the concatenation of all 19 records is meaningful. |
| Web | In Query Me This the app renders a SQL syntax error exactly like an empty result, so a bare `' OR 1=1 --` looks like a dead end; close the paren first. Endpoint names and hashed directory/file names vary per instance and have to be read from the live responses. |

## Mistakes in our own tooling or process

Collected here from the challenge pages and the files behind them.

- **Query Me This ranked solver (web).** On 2026-10-29 an untimed Time Trial rehearsal (`untimed-1`) ended with
  `solver failed: no known web-exploitation class matched`. The scripted solver hard-coded one search endpoint,
  but the SQLi instances name it per skin (`/lookup`, `/explore`, `/search`, `/query`). Commit `596e438f` notes
  that this rehearsal "caught one gap" and added discovery from the homepage form `action`; all ten reruns of the
  three challenges then went CORRECT on call 1. The leaf holds a `pass.log` for the failed rehearsal and no
  `result.json`. Nothing ranked was spent on it (the untimed rehearsal runs in a separate pool, per the commit).
- **A misfiled ranked rerun (the stray leaf).** A ranked Time Trial rerun of Query Me This (2026-11-01, attempt
  `6eabf997-bbf7-43c1-8212-8ed41dae0f8d`, `CORRECT`, 1 call, 4.93 s) was saved as
  `challenges/day03/web-exploitation/web-exploitation-day-03-sqli/time-trial/ranked-1/` instead of under
  `challenges/day19/`. The `day03` leaf holds only that one `ranked-1` folder (no journal, no start response, no
  other mode), and its `result.json` names the right challenge id (`web-exploitation-day-03-sqli/time_trial`).
  It entered the repository in the 2026-11-01 sweep commit `38668151`, whose message says the Days 2-20 reruns
  were dispatched through `tools/ranked_pass.py`. The `03` embedded in the id matches the folder, so it looks
  like the leaf folder was taken from the id's number rather than from the release day, the confusion that
  `CLAUDE.md` warns about for this lane; we could not find the line that did it. The real Day 19 leaf has its own
  `ranked-1` (13.16 s), so this is one extra CORRECT rerun in the wrong place, not another challenge.
- **Single Solve evidence was never saved as a result file (all eight).** The artifact lanes kept hand-written
  attempt logs in three different shapes (a bare list, a list with `n`, a dict with `attempts`), and the web
  leaves kept only an `accepted_map.json` of the proof string. None of that is a saved graded response, so
  every Single Solve here is Journal-recorded. The Metadata log even stores the two guesses as descriptions
  (`"unpadded 0x3e"`), not as the submitted values.
- **`gm.py artifact` fetched only the first delivery of a multi-artifact Mastery** (Mobile five APKs, BRE five
  archives). The others were fetched from the capability links in `artifacts[]`. This was already known from
  Day 11, where the flag was added; the Day 19 pages still describe working around it by hand.
- **The mobile APK inspector did not support the `reflection` codec** and handled one APK at a time, so a
  per-challenge mapper was written, since folded into the lane's general codec decoder (Mobile page).
- **The isolated interpreter hides user-site packages.** `python -I` hides `pyzipper` (Script, BRE) and ignores
  `PYTHONUTF8`; extraction had to run without `-I`. This is the host gotcha already in `CLAUDE.md`.
- **Warmups record 18 was inferred, not identified.** The block permutation was reconstructed from two blobs by
  putting known slug words back together and then gated by the strict slug regex. A later review recognised it as
  a 3-rail rail fence, confirmed on all 93 payloads. The answer was right, but the first reading was guessed.
- **No ledger or cap problems.** The only cost on the day was the one rejected Metadata Single Solve submission;
  every other mode was one call.

## Tooling and process changes around this day

No `_run/` folder exists for Day 19 (it predates release automation), so there is no timeline, lane log or
librarian note. From `git log` and the files:

- `27a56e46` (2026-10-23): "Add Day 19 challenge work". Its message says Day 19 was completed with the three web
  Time Trial and Mastery solves (web Single Solves were already done) and that it promoted
  `library/web-exploitation/paren-wrapped-sql-injection.md` and extended `idor-and-info-disclosure.md`. It also
  touched `CLAUDE.md` (the Query Me This header-clock note is in its "Web Exploitation" section).
- `8a492bcc` (2026-10-28): Warmups Time Trial and Mastery `ranked-1` results.
- `34e7f1a0` (2026-10-29): thin contract-v1 ranked solvers for the Day 17-20 Metadata, Mobile, Script and BRE
  challenges, built from existing library engines; its message says Day 19 BRE needed its own spelling spec
  (`ELF64`/`x86-64`, unlike Days 15, 17 and 20). Day 19 reruns came from a second machine (Metadata, Script, BRE)
  and from the main workstation (Mobile, which needs `capstone`).
- `596e438f` (2026-10-29): the Day 19-20 web solvers, with `web_engine.py` implementing the three Day 19 classes,
  and the ten web reruns including the `untimed-1` gap above.
- `38668151` (2026-11-01): the top-30 gap-closure sweep; for Day 19 it added Metadata Mastery `ranked-2`, Mobile
  Time Trial `ranked-2` and the stray Day 3 leaf.
- Local rules touching this day: the 2026-10-29 rule that GM `status` is never evidence and the `manual_solve.json`
  marker (`27a80dee`). None of Day 19 needed the marker; no Day 19 Single Solve was team-scored.
- Library entries from the day: `WARMUP-DAY19-HEX-UTF16-001`, `META-BUNDLE-001`, `MOBILE-ANDROID-REFLECTION-001`,
  `BRE-OVERLAY-PK19-001` and `BRE-RECAP-D19-001`, `SCRIPT-TECHNIQUE-D19-MARKERS`, `WEB-SQLI-001`, `WEB-IDORDISC-001`.

## Effort

Submission calls, from the journals, attempt logs, `progress.md` and `result.json` (`calls`):

| Lane | Single Solve | Time Trial (original) | Mastery (original) | Reruns that went CORRECT (all 1 call) |
|---|---|---|---|---|
| Warmups | 1 | 1 | 1 | 1 TT, 1 Mastery |
| Metadata | 2 (`0x3e` rejected, `0x003e` accepted) | 1 | 1 | 1 TT, 2 Mastery |
| Mobile | 1 | 1 | 1 | 2 TT, 1 Mastery |
| Binary | 1 | 1 | 1 | 1 TT, 1 Mastery |
| Script | 1 | 1 | 1 | 1 TT, 1 Mastery |
| Web: Back Up a Minute | 1 | 1 | 1 | 1 ranked + 1 untimed each, TT and Mastery |
| Web: Permission to Slip | 1 | 1 | 1 | 1 ranked + 1 untimed each, TT and Mastery |
| Web: Query Me This | 1 | 1 | 1 | TT 1 ranked + 1 untimed + 1 stray ranked; Mastery 1 ranked + 1 untimed |

Local `t_total` per rerun (seconds): Warmups TT 2.1, Mastery 2.5; Metadata TT 1.6, Mastery 4.8 and 3.0; Mobile TT
3.0 and 2.0, Mastery 3.2; Script TT 2.5, Mastery 4.1; Binary TT 8.8, Mastery 18.4 (the 32 MB downloads dominate).
Web ranked reruns took 9.9-17.9 s (Back Up a Minute 10.3 and 10.7, Permission to Slip 9.9 and 10.9, Query Me This
13.2 and 17.9; the stray rerun 4.9 s); the untimed web rehearsals took 10.2-44.1 s. The original solves were all
untimed attempts, so no original solve time is on file for any mode, and the server-side rank is not recorded in
the leaves.
