# Day 20 overview

**At a glance:** 7 challenges across 6 lanes, 21 modes (three non-web lanes' worth of bank/bundle work, plus
two live-target web challenges: the web lane had **2** challenges this day, not 3). By provenance label,
after reconciling against local files: 14 Independent, 7 Journal-recorded, 0 Local mark only, 0 Team, 0 n/a.
Theme: a day of cumulative recaps and prompt flavor that does not name the technique. Four lanes
reused an earlier mechanism (Day 19's bundle extractor, the Day 15 binary mapper, the Day 15-19 script
descriptor bank, the numbered warmup chain) and added one new field or codec; the costs were format
guesses and fixture variation, not finding the idea.

## Challenges

| Lane | Challenge | Technique | Single Solve | Time Trial | Mastery |
|---|---|---|---|---|---|
| Warmups | [Big Entrance](warmups_big-entrance.md) | Hex of UTF-16 with a big-endian BOM; numbered codec chain (record 20, plus a 5-character record-18 block) | Journal-recorded | Independent | Independent |
| Metadata Forensics | [Universal Appeal](metadata-forensics_universal-appeal.md) | Mach-O `cputype` (thin vs fat), padded-hex JSON array answer | Journal-recorded | Independent | Independent |
| Script Analysis | [The Butler Did It](script-analysis_the-butler-did-it.md) | Descriptor decoding over a 1,329-id bank (Jenkins/Groovy host plus older families); structural Batch controls | Journal-recorded (page says Independent, see below) | Independent | Independent |
| Mobile Android | [Loaded Later](mobile-android_loaded-later.md) | `ANS-` constant in a shipped-but-never-loaded `payload.dex` (`dynamic` codec) | Journal-recorded | Independent | Independent |
| Binary Reverse Engineering | [Resourceful](binary-reverse-engineering_resourceful.md) | d01-d20 recap bundle (240 ids); `PK19`/`RS20` trailing overlays carrying an HCFG record | Journal-recorded | Independent | Independent |
| Web Exploitation | [Stay in Your Directory](web-exploitation_stay-in-your-directory.md) | PHP LFI, unfiltered `include(base . $param)` | Journal-recorded | Independent | Independent |
| Web Exploitation | [Administratively Speaking](web-exploitation_administratively-speaking.md) | Missing function-level authorization on an admin route | Journal-recorded | Independent | Independent |

Web challenge IDs embed `curriculum_day` 4 and 5 (`web-exploitation-day-04-lfi`,
`web-exploitation-day-05-administratively-speaking`); both sit in `day20` because that is the release day.
A third ID, `web-exploitation-day-20-serialized-offender`, carries "day-20" in its name but is a Day 26
release (`progress.md` rows dated 2026-10-26), so it is not part of this day.

How to read the labels (each page gives its own evidence):

- **Journal-recorded (all seven Single Solves).** No Single Solve leaf holds a `result.json`,
  `accepted_plan.json` or `manual_solve.json`. Each journal records our own submission as CORRECT and each
  has a `progress.md` row (non-web 2026-10-20, web 2026-10-23). For the five non-web lanes plus Script,
  `artifacts/submission-attempt-log.json` also reads CORRECT and `challenges/submission-ledger.json` logs a
  CORRECT, with two exceptions noted below; web Single Solves rest on the journal and the ledger row only.
  An attempt log and the ledger are call logs, not saved graded results, so the label stays Journal-recorded.
- **Independent (every Time Trial and Mastery).** The original attempts (non-web 2026-10-20, Metadata and
  Script Mastery re-solved 2026-10-23, web 2026-10-23) are journal-recorded. The label rests on saved
  `result.json` files from later reruns, all CORRECT with `calls: 1`: `ranked-N/` for every mode (2026-10-28 to
  2026-11-01) and, for the web lane, `untimed-N/` regular reruns too. Counting graded reruns: Warmups 2 TT +
  1 Mastery, Metadata 1 + 1, Script 1 + 1, Mobile 2 + 2, Binary 2 + 1, each web challenge 1 ranked + 1 untimed
  per mode. Ranked runs were bragging rights only and are not the prize ranking.

### Label disagreements found in the pages (reported, not edited)

- **Script Analysis page, Single Solve:** reads "Independent (2 submission calls)". The leaf has only
  `submission-attempt-log.json` (two entries, the second CORRECT) and no result file, so by the brief's rule it
  is Journal-recorded. Time Trial and Mastery as written are fine (`ranked-1/result.json`).
- **Script Analysis page, Mastery:** says "the original Mastery's second call has no recorded result". The
  attempt log's second entry has no parsed result, but the ledger records a CORRECT at 2026-10-20 21:39 for
  `.../mastery` immediately after an `accepted`. Same evidence on a different file; the label is unaffected.
- **Stay in Your Directory page:** does not mention that the 2026-10-29 regular Mastery rehearsal
  (`mastery/untimed-1/pass.log`) failed with "no known class matched" (see mistakes below). The
  Administratively Speaking page does record the equivalent failure.
- **All pages:** write "Solved (journal-recorded)" for what this overview calls Journal-recorded; same
  meaning, different wording.

## What the day had in common

- **Cumulative banks.** Binary was Day 15's mapper over 150 of 240 ids (d01-d15 specimens byte-identical)
  plus 90 new/extra fields. Script's ids were a strict superset of the Days 10-19 banks (80 new ids).
  Metadata re-used Day 19's 23-field generic bundle plus the Mach-O field and one synthesis question.
- **Prompt flavor, not mechanism.** Metadata promises "what truly supports the advertised fleet" but no specimen
  was fat. Mobile's title and prompt say a secondary DEX is loaded later, but the app never loads it. Script's prompt
  about controller vs agent semantics is flavor text. Warmups was the one lane whose riddle ("the big one goes
  first") matched the mechanism (big-endian).
- **Mastery mixed fixtures.** Mobile Mastery was four earlier-day codecs plus the Day 20 `dynamic` one; Binary
  Mastery delivered d02/d04/d10 as PE in sets 1, 3 and 5 against ELF in the other modes; Metadata Mastery's
  per-set manifest varied between variants; both web Mastery bundles paired the challenge's own class with an
  unrelated Node.js robots.txt to `.json.bak` backup disclosure target.
- **Windows were short.** `tools/release/README.md` records the Day 20 timing windows: Warmups TT 30 s / Mastery
  115 s, Script TT 50 s / Mastery 170 s, Binary TT 80 s / Mastery 265 s (the `timing_meta.json` it cites is not in
  this day's `_run`). The regular attempts were untimed. Ranked reruns finished in 1.27-18.1 s (`t_total`
  across all non-web and web ranked results).
- **Web Time Trial hidden clock.** Both web Time Trials returned `deadline: null` from `start`, but the live target's
  response headers carried `X-CTF-Time-Trial-Deadline-Ms: 7200000` (~2 h). The Stay in Your Directory journal
  records it first; the Administratively Speaking journal records it too. The ranked reruns confirm the windows
  were real (124.7 s and 127.7 s `hall_of_fame_eligible: true` windows for the two Time Trials).

### Validator and format traps seen across lanes

| Lane | Trap |
|---|---|
| Metadata | `answer_format` said only "compact valid JSON array". `["arm64"]` and `[16777228]` were `incorrect`; `["0x0100000c"]` (lowercase, `0x`, 8 digits) was CORRECT on the third call. The padded-hex convention already existed in earlier fields. |
| Metadata | Mastery variants differ: set 5 lacked `evidence-note-copy.txt`; the magic-prefix/BOM questions named `roster-...` instead of `evidence-note-...`. Extractors must be manifest-driven. |
| Script | Day 15's per-id Batch control overrides were wrong for Day 20's fixtures (`if_same`, `direct`, `if_same`); classify controls structurally. One-item `FOR /L ... DO CALL` was `call_subroutine`. Only 45 of Mastery's 90 `-control` ids were Batch. |
| Script | New marker containers: Google Apps Script `SPLIT` arrays and C# string-literal hosts with a trailing `\n` escape. |
| Binary | Mastery PE builds: COFF symbol count zero (Go's `.symtab` section has no COFF symbols), `/<decimal>` section names via the string table, MinGW `.text$`/`.rdata$` prefixes; Zig `.relro_padding` is NOBITS and overlaps the `.hcfg` file range. |
| Binary | `hcfg_resource_size` is the u32 at overlay +8, not the HCFG payload length; PK19 hashes the unpacked bytes, RS20 hashes the scratch bytes. |
| Mobile | `client_secret=SECRET-...` and the loopback evidence URL are decoys. A later Day 24 Mastery drew a Day 20 build whose `payload.dex` was loaded by an `InMemoryDexClassLoader`, so a dead `payload.dex` is not a safe assumption. |
| Warmups | Record 18 is a 3-rail rail fence, not a fixed 6-block permutation; the 5-character blocks (8 of 98 blobs) were an inference when submitted. Mastery has `5N-2` blobs on Day N. |
| Web | The reader path differs per target (`/reader.php`, `/notes.php`); `/api/clues` hands out the privileged route; proofs are `exact_text`/`trim` as echoed. Mastery target order is not fixed. |
| All lanes | Every answer value is a string; multi-answer `incorrect` has no per-field feedback. |

## Mistakes in our own tooling or process

Collected here from the pages, journals and `challenges/submission-ledger.json` (call logs).

- **Universal Appeal Single Solve: a scripted format loop did not stop at CORRECT.** The third call scored CORRECT;
  a fourth (unpadded hex) went out and returned `already_solved`. The ledger shows all four within one minute
  (2026-10-20 21:46). It graded nothing and used a ledger entry.
- **Universal Appeal Mastery: nine calls to an ungraded variant.** A 2026-10-21 untimed re-start left an open,
  ungraded attempt on a new artifact. The ledger shows 1 `accepted` and 8 `incorrect` calls against it (2026-10-21
  00:30 local), then a re-solve on a fresh variant, CORRECT on 2026-10-23 (ledger entry 11 of 12). The journal
  calls it "the 2026-10-21 release rehearsal". The extractor had also assumed one file manifest and one
  magic-prefix file name; both broke on the fresh variant.
- **The same re-start left Script Mastery ungraded too.** Per `progress.md`, the 2026-10-20 solve was "re-confirmed
  2026-10-23 on a fresh variant"; two more calls (`accepted` then `CORRECT`) went to the new variant.
- **Warmups Single Solve: a wasted `already_solved`.** The ledger holds a second call for `warmups-day-20` at
  2026-10-21 00:26 returning `already_solved`, during the same rehearsal window. The Warmups page does not mention
  it; the cause (a rehearsal re-submitting a solved Single Solve) is inferred from the timing, not stated in a file.
- **Rehearsal duplicates inflate the ledger.** In that 00:26-00:33 window the ledger also logged extra CORRECT calls:
  Warmups TT 2 and Mastery 2, Mobile TT 2 and Mastery 2 (the Mobile journal confirms "two extra CORRECT calls from
  the 2026-10-21 release rehearsal"), one each for Metadata TT and Script TT. Ledger counts for these modes therefore
  exceed the number of solves. Not mistakes in the answers, but a caution when reading counts as effort.
- **Script's original Single Solve and Time Trial calls are missing from the ledger.** `submission-attempt-log.json`
  records both (2026-10-21 04:37-04:38 UTC), but `script-analysis-day-20-jenkins-pipeline-intrusion` and its
  `/time_trial` first call have no ledger rows (the Time Trial key starts at 2026-10-21 00:33 local). `CLAUDE.md`
  documents an earlier ledger clobber from concurrent runs; whether that is the cause here is not determined.
- **Script Single Solve/Time Trial mapper carried stale Batch overrides** from Day 15. The page records them
  being removed in favor of structural classification; it does not say whether any submission was spent on them.
- **Web Mastery class detector missed a target (2026-10-29).** The regular (untimed) Mastery rehearsals for **both**
  web challenges failed with "solver failed: no known class matched" (`mastery/untimed-1/pass.log` in each leaf);
  the next attempt drew a pair the detector knew and went CORRECT in one call (`untimed-2`, 6.49 s and 7.11 s).
  Nothing was submitted for the failed ones. Commit `596e438f` says the fix was a broader three-class detector
  (backup disclosure, activity-feed IDOR, paren-wrapped SQLi) with the challenge's own class tried first, and that a
  rehearsal fixture "turned up a paren-wrapped-SQLi target under the LFI challenge's own Mastery". The class that
  failed the Administratively Speaking rehearsal is not recorded. The failure is attributed to the solver having
  been written from one delivery's targets, as Mastery targets are re-issued per attempt.
- **Mobile Mastery: index-zipped codec script (superseded).** A first script paired questions with deliveries by
  position; the final mapper matches by filename and SHA-256. `gm.py artifact` fetched only the first Mastery
  delivery, so a small helper downloaded all five.
- **Warmups Mastery record 18** was submitted on a plausibility guess rather than a derivation; it was right and was
  later generalized to a rail fence.
- **Binary: no `objdump`** on the host, so the d01 `recover_code` disassembly was decoded from the six bytes
  directly (`b8 imm32 c3`) in objdump spacing.

## Tooling and process changes around this day

`_run/` for Day 20 holds only `release_list.json` and `watch.log`: no `orchestrator.log`, lane logs, librarian
notes, `status.json`, `timing_meta.json` or supervisor report. What they and the other files give:

- **Release and watcher.** `release_list.json` has 21 entries (7 challenges x 3 modes); the web entries carry
  `curriculum_day` 4 and 5, the other lanes none. `watch.log` shows a watcher started with a target release of
  2026-10-21 07:27:41 UTC, a preflight listing day 19 (24 rows), and day 20 detected "after 1 polls at T-3.41s" and
  handed to `orchestrate.py --day 20 ... --authorized --sandbox --cap 100`. Because the day's solves were already in
  the ledger and the list was visible at the first poll, this is a rehearsal of the release tooling against an
  already-released day (`tools/release/README.md` names it "the Day 20 rehearsal" and notes "`--sandbox`: Single
  Solve cannot be re-issued once solved"), not the real unlock. The real release time is not in any file here.
- **Timeline from the files (local times, apparently UTC-7, inferred from the attempt logs' UTC stamps).**
  2026-10-20 21:37-21:48: all five non-web Single Solves, Time Trials and (for Script, Binary, Warmups, Metadata,
  Mobile) Mastery calls land in the ledger/attempt logs within about eleven minutes. This is the span of the
  submissions, not solve effort, which the files do not time. 2026-10-21 00:26-00:33: the rehearsal calls above.
  2026-10-23 22:13-22:32: the Metadata and Script Mastery re-solves and, interactively, all six web modes (the web lane
  first ran unattended on Day 22, so these were hand-driven). 2026-10-28 to 2026-11-01: ranked reruns.
- **Commits.** `29c733f4` (2026-10-22) added the Game Master client and release automation (the code the Day 20
  rehearsal exercised). `fd47928c` (2026-10-23) "Add Day 20 challenge work" added the two Mastery re-solves.
  `d73435b3` (2026-10-28) added Warmups ranked results. `34e7f1a0` (2026-10-29) added Day 17-20 ranked solvers for the
  non-web lanes (Metadata, Script and Binary reruns ran from a second machine, Mobile from the main one, because the
  second machine lacks `capstone`). `596e438f` (2026-10-29) added the Day 19-20 web solvers and reruns, including the
  broader class detector. `38668151` and `f0e20aa1` (2026-11-01) are the last gap-closure sweeps; the ledger shows
  2026-11-01 ranked calls for Warmups TT, Mobile TT and Mastery, and Binary TT.
- **Librarian.** There is no `_run/librarian` note. The leaf journals carry per-leaf "Library promotion" sections from
  a 2026-10-20 curation sweep; entries named across the pages are META-MACHO-CPUTYPE-001, META-BUNDLE-TOOLKIT-B101,
  BRE-OVERLAY-B103, BRE-OVERLAY-TAGGED-001, BRE-HCFG-STORAGE-001, BRE-GOTCHAS-B106, SCRIPT-TECHNIQUE-D20-GROOVY,
  MOBILE-ANDROID-DYNAMIC-DEX-001, WARMUP-DAY19-HEX-UTF16-001, WARMUP-NUMBERED-CODEC-CHAIN-001, WEB-LFI-001 and
  WEB-BAC-001. `CLAUDE.md` carries the lasting Day 20 lessons: padded-hex Mach-O `cputype`, the Metadata Mastery
  manifest variation, BRE Mastery sets mixing PE and ELF, and the web Time Trial header clock. No escalation to a
  stronger model, supervisor report or closeout file exists for this day.

## Effort

Submission calls from attempt logs and the ledger (the original attempts; reruns counted separately):

| Lane | Single Solve | Time Trial (original) | Mastery (original) | Later graded reruns (all `calls: 1`) |
|---|---|---|---|---|
| Warmups | 1 | 1 | 1 | TT 2 ranked; Mastery 1 ranked |
| Metadata | 4 (2 `incorrect`, CORRECT, wasted `already_solved`) | 1 | 1, then 9 to an ungraded variant, then 1 re-solve | TT 1, Mastery 1 (ranked) |
| Script | 2 | 2 | 2, then 2 on the re-solve variant | TT 1, Mastery 1 (ranked) |
| Mobile | 1 | 1 | 1 | TT 2, Mastery 2 (ranked) |
| Binary | 1 | 1 | 1 | TT 2, Mastery 1 (ranked) |
| Stay in Your Directory | 1 | 1 | 1 | TT: ranked + untimed; Mastery: ranked + untimed (one earlier untimed rehearsal failed, nothing submitted) |
| Administratively Speaking | 1 | 1 | 1 | same as above |

Ranked `t_total` ranged from 1.27 s (a Mobile ranked run) to 18.1 s (Binary Mastery); regular web reruns took 6.49-13.23 s.
No Time Trial or Mastery original needed more than the calls listed; the only `incorrect` verdicts anywhere in this
day's ledger rows are the two Metadata Single Solve format guesses and the eight ungraded-variant Mastery calls.
