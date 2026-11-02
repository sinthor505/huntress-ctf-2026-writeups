# Day 30 overview

Day 30 released at 09:00 ET (13:00 UTC) on 2026-10-30: eight challenges across six lanes (Binary
Reverse Engineering got a second, one-off challenge), 22 attempted modes. Five Single Solves were
scored by a teammate within about ten minutes of unlock; everything else, including every Time
Trial and Mastery, we solved and graded ourselves. The day's one real reversing challenge
(SHA1-Hulud) has no Time Trial or Mastery at all and was never re-gradable for us once a teammate
solved it, so it stands apart from the rest of the day.

## Challenges

| Lane | Challenge | Technique | Single Solve | Time Trial | Mastery |
|---|---|---|---|---|---|
| Warmups | [Percent Percent](warmups_percent-percent.md) | double percent-decoding (`%25%HH`) | Team solve (not ours) | Independent | Independent |
| Metadata Forensics | [Seeing Double](metadata-forensics_seeing-double.md) | exact-identity grouping by content hash | Independent | Independent | Independent |
| Script Analysis | [Spill the Beans](script-analysis_spill-the-beans.md) | BeanShell + JSP-hosted descriptor | Team solve (not ours) | Independent | Independent |
| Mobile Android | [Layered Protections](mobile-android_layered-protections.md) | v1 `layered` codec (SS) / v2 Dalvik-VM-run codec (TT, Mastery) | Team solve (not ours) | Independent | Independent |
| Binary RE | [Campaign Trail](binary-reverse-engineering_campaign-trail.md) | Neko bytecode classification + HGRF trailer | Team solve (not ours) | Independent | Independent |
| Binary RE | [SHA1-Hulud](binary-reverse-engineering_sha1-hulud.md) | self-decrypting VEH walls + MBA-chain collapse (flag checker) | Team solve (not ours) | Absent | Absent |
| Web Exploitation | [Pass the Identity](web-exploitation_pass-the-identity.md) | confused deputy via unsigned delegated-identity header | Independent | Independent | Independent |
| Web Exploitation | [Queue the Confusion](web-exploitation_queue-the-confusion.md) | job-queue owner-field mass assignment | Independent | Independent | Independent |

By provenance label across the 22 attempted modes: **17 Independent**, **5 Team solve (not ours)**
(all Single Solve), **2 Absent** (SHA1-Hulud has no Time Trial or Mastery leaf; nothing was invented
for those rows).

## How to read the labels

- **Team solve (not ours), confirmed at submission time.** Percent Percent, Spill the Beans, Layered
  Protections and Campaign Trail each have a journal entry timestamped between 13:01 and 13:11 UTC on
  2026-10-30 — minutes after unlock — that records our own first submission returning `already_solved`
  directly from the Game Master, before any later marker existed. SHA1-Hulud is the same but with a
  `solved_by` timestamp (16:01 UTC) and a confirmed-live `retry` 405 / `practice` 409. None of these five
  rest on a later resume pass.
- **A later resume pass's `team_solved.json` marker is not itself evidence, and two pages correctly
  overrode it.** A resume run wrote a `team_solved.json` (`reason: "start_already_correct"`) on six Single
  Solve leaves on 2026-10-31 (23:06-23:08 UTC) and 2026-11-01 (06:45-06:46 UTC) — a later pass re-checking
  `start` and finding it already read CORRECT. For four challenges that marker agrees with the original
  2026-10-30 `already_solved` journal entry above, so it changes nothing. For **Seeing Double** and **Pass
  the Identity**, it does not agree: both leaves' own journals and `_run/status.json`'s `ss` field (`CORRECT`,
  not `team-solved`) show our own submission was graded CORRECT at round 0 on 2026-10-30, and the live
  target had simply idled out by the time the resume pass re-checked it. `start_already_correct` fires
  identically whether the correct answer on file is a teammate's or our own — it is bookkeeping, not a
  verdict on whose solve it was. Both pages checked the journal timestamp and the `progress.md` "Solved"
  row before writing "Independent," which is the right call: this is the day's clearest case of "a status
  read is not evidence" applying to our own automation's artifacts, not just to `gm.py status`.
- **SHA1-Hulud's offline re-derivation is ours, but it does not change the Single Solve label.** The static
  wall-isolation-plus-MBA-collapse solve was built from our own copy of the binary and reproduces the flag
  cryptographically (coherent decoy plaintexts, byte-exact forward re-encryption). It is real, independent
  work, but the Game Master never graded it: SHA1-Hulud is Single-Solve-only, a teammate's solve blocks any
  fresh attempt (`retry` 405, `practice` 409, confirmed live), and there is no Time Trial or Mastery to fall
  back on. The table marks Single Solve "Team solve (not ours)" and Time Trial/Mastery "Absent" — not
  "Independent" and not invented rows.

## Timeline (UTC)

| Time (2026-10-30 unless noted) | Event |
|---|---|
| 13:00:05 | Day 30 unlocks; 8 challenges across 6 lanes authorized (ledger cap 30, 70 once escalated to Opus) |
| 13:01-13:11 | All seven non-SHA1-Hulud Single Solves submitted; six return `already_solved` and are logged as team-scored at that moment (Seeing Double and Pass the Identity return CORRECT) |
| 13:10:13 | A rescoped orchestrator run (Binary RE only) skips resuming `multi-binary-campaign` — still being driven by the original orchestrator process — to avoid a duplicate worker race |
| 13:12:16 | All 7 non-BRE-flag-checker lanes report finished; Librarian pass starts |
| 13:12-13:56 | Librarian writes a note per finished challenge (Campaign Trail's took 19 min — the longest, over the crystal-symbol-count alternate-group check; the rest 3-9 min) |
| 14:26:33 | Supervisor report: orchestrator no longer alive, SHA1-Hulud `blocked`/`open` on Single Solve; an Opus resolver runs twice (r1 `RESUMED`, r2 `NEEDS_HUMAN`) |
| 19:33-19:50 (10-31) | SHA1-Hulud's independent static re-derivation and library promotion committed, then the CLAUDE.md note documenting it as SS-un-regradable |
| 23:06-23:08 (10-31) | A resume pass re-checks 4 challenges (Metadata, BRE Campaign Trail, both Web); writes the `team_solved.json`/`start_already_correct` markers discussed above |
| 06:45-06:46 (11-01) | A further resume pass re-checks 3 challenges (Mobile, Script, Warmups); same marker pattern |

SHA1-Hulud is the one challenge that never reaches "finished": the supervisor's final verdict for it
stays `blocked`/`NEEDS_HUMAN` through the whole run.

## Submission effort

Submission calls per challenge and mode, from the `progress.md` ledger notes (original/regular
attempts; later ranked reruns on 2026-10-31/11-01 were bragging-rights only under the Day 30 prize
rule and are not counted here):

| Challenge | Single Solve | Time Trial | Mastery |
|---|---|---|---|
| Percent Percent | team (0 graded) | 1 | 1 |
| Seeing Double | 2 | 16 | 1 |
| Spill the Beans | team (0 graded) | 7 | 1 |
| Layered Protections | team (0 graded) | 1 | 1 |
| Campaign Trail | team (0 graded) | 4 | 1 |
| SHA1-Hulud | 1 (`already_solved`) + 2 automation placeholder calls | absent | absent |
| Pass the Identity | 1 | 1 | 11 |
| Queue the Confusion | 1 | 1 | 1 |

Seeing Double's Time Trial (16 calls) and Pass the Identity's Mastery (11 calls) are the day's two
expensive modes; both are explained below under automation mistakes, not validator behavior.

## What the day had in common

- **Most lanes added nothing new to reverse — only to parse.** Campaign Trail's new block was three
  Neko bytecode files whose "campaign graph" was one already-known HGRF/HCFG-layout trailer record;
  Seeing Double's new field was a content-hash grouping, not a new file format; the day's only
  genuine new-language addition was BeanShell (plus a JSP-hosted BeanShell page) in Script Analysis,
  and the declared-descriptor-wins rule held for it unchanged.
- **Unstated answer shape, not extraction, was the expensive part twice.** Seeing Double's grouping
  labelling convention (manifest `id` vs bare basename, depending on whether a manifest shipped) and
  Campaign Trail's single-vs-doubled mutex backslash were both guessed wrong once before the accepted
  spelling was found.
- **Mobile and web both split by schema/target, continuing the Days 28-29 pattern.** Layered
  Protections' Single Solve used a new v1 codec that the Time Trial and all 14 Mastery sets never
  used (they were v2, solved by running the APK's own bytecode); both web Mastery bundles again mixed
  the challenge's own class with four or five unrelated earlier-day classes (backup disclosure, IDOR,
  paren-wrapped SQLi, PHP LFI, and for Queue the Confusion a sixth, bare broken-access-control target).
- **A team-scored Single Solve did not block Time Trial or Mastery.** Five of the six non-web Single
  Solves were team-scored, and every one of those challenges' Time Trial and Mastery modes still went
  CORRECT on our own `retry`. SHA1-Hulud is the exception only because it has no Time Trial or Mastery
  to retry.

### Validator and format traps seen across lanes

| Lane | Trap |
|---|---|
| Metadata | Group-member labelling depends on whether a manifest ships: the manifest's `id` field (no extension) when present, the bare basename when absent; extension-bearing basenames were rejected either way. |
| Binary RE | The decoded mutex value's single backslash (not the JSON-escaped doubled form) was accepted at variant 0 on every Time Trial and Mastery fixture. |
| Binary RE | `d13_analysis_crystal_symbol_count` (`SHT_SYMTAB` only vs. plus dynamic symbols) stayed an open alternate-group question through Days 28-29; Day 30 confirmed symtab-only again. |
| Script Analysis | Node.js staged-chain `attack` follows the question-id schema (hash-id -> `T1059.007`, named-id -> `T1105`), not the day number — the fifth day this distinction has mattered. |
| Mobile Android | The Single Solve's codec (v1 `layered`) never predicted the Time Trial/Mastery schema (v2, run through the Dalvik VM); a solver has to dispatch on the asset's own schema field. |
| Web Exploitation | A full-batch multi-target Mastery submission returns a flat `incorrect` with no partial-credit signal; isolating by `proof-N` group is what actually finds the bad field(s). |
| Web Exploitation | Mastery bundles mix vulnerability classes under one challenge name; every `proof-N` has to be mapped to its own target via the question's own `artifact` field, never list position. |

## Mistakes in our own automation

Collected here per the writeup brief (these are not challenge/validator lessons, so they are not on
the individual pages).

- **Seeing Double (Metadata).** The regular Time Trial burned 14 of its 16 calls because the first
  pass reused the Single Solve's accepted spelling as a variant *index* rather than as a labelling
  *rule*; once the manifest disappeared in the Time Trial fixture, de-duplicating variants in the
  manifest-less shape shifted what that index pointed at, so the correct reading was never tried in
  isolation until a fix round changed the approach to carry the rule, not the index.
- **Pass the Identity (Web, Mastery).** Round 0 queried only `targets[0]` and copied its evidence
  value to all five Mastery `proof-N` answers; four of five came back unaccepted. Fixed with a
  per-question target resolver keyed on the question's own `artifact` field
  (`library/web-exploitation/map_targets.py`). Separately, one full 5-answer batch submission came
  back a flat `incorrect` with no `accepted_ids`; isolating each `proof-N-format` group one call at a
  time is what actually resolved it, which is most of why this mode cost 11 ledger calls against a
  5-target bundle.
- **Queue the Confusion (Web, Mastery).** The same class of bug — a hardcoded `targets[0]`/
  `answer_ids[0]` solver that was correct for the one-target Single Solve — produced a near-empty
  answer set against Mastery's six targets before the same artifact-field fix was applied.
- **Campaign Trail (Binary RE).** The regression tool used to check `d13_analysis_crystal_symbol_count`
  across days assumed a fixed alternate-group variant index per day and reported false mismatches
  until it was changed to read each leaf's own `result.json` for which variant was actually accepted.
- **SHA1-Hulud / `orchestrate.py` recon gate.** `Lane.run()` treated any start response without a
  top-level `questions` key as a failed recon. SHA1-Hulud is a flag-style BRE Single Solve
  (`answer_ids: ["flag"]`, no `questions`), so recon stalled at 13:00:06 before the artifact was even
  downloaded. The Opus resolver's first round (`RESUMED`) found and fixed this.
- **SHA1-Hulud / `resume_lane.py` scope.** While fixing the recon gate, the resolver found that
  resuming by lane (`--lanes binary-reverse-engineering`) would resume every challenge in that lane —
  including the still-in-progress sibling Campaign Trail, which was being actively driven by the
  original orchestrator process at the time. A resume aimed only at SHA1-Hulud would have started a
  second, duplicate worker set against Campaign Trail. This was caught and avoided before it ran.
- **SHA1-Hulud / empty placeholder submissions.** With no real candidate built yet, early automation
  rounds submitted an empty placeholder answer for the `flag` id twice, spending 2 of the 70-call
  escalated cap on nothing. The resolver's second round correctly declined to spend more calls on the
  same empty answer and reported `NEEDS_HUMAN` instead of retrying blindly.
- **Stale `team_solved.json` markers (cross-lane).** See "How to read the labels" above: a resume pass
  on 2026-10-31/11-01 wrote the same `start_already_correct` marker on six leaves regardless of whether
  the CORRECT answer on file was a teammate's or our own from hours earlier. It cost nothing in
  submissions, but it would have mislabeled Seeing Double and Pass the Identity's Single Solves as
  team solves had each page not cross-checked the marker's timestamp against its own journal and
  `progress.md` row before writing the provenance table.

## Tooling and process changes around this day

From the commit history (`git log`, local time shown as committed):

- `56d03c66` (2026-10-30 13:51 -07:00, shortly after unlock): challenge-strategy guide and a
  JDK-API-modelling update to the Dalvik mini VM. Layered Protections' Time Trial and Mastery ran
  through this VM unchanged — no Day-30-specific VM work was needed.
- `0bccd687`: imported a teammate's (`tunwinlat`) intel on an MBA flag-checker technique, Day 31
  predictions, and forensic recipes. This import is the specification SHA1-Hulud's static
  re-derivation ported the MBA-collapse idea from, per that page's own account.
- `a05ede20` (2026-10-30 22:32 -07:00): Day 30 solved lanes (all but SHA1-Hulud), Librarian pass,
  `progress.md` rows.
- `997c6702` (2026-10-31 02:21 -07:00): the SHA1-Hulud leaf and run logs, plus the
  `orchestrate.py` Day 30+ default and ledger changes that include the recon-gate fix described above.
- `c058303c` (2026-10-31 19:33 -07:00): SHA1-Hulud's independent static re-derivation and library
  promotion (BRE-MBA-FLAGCHECK-001, `wall_isolate.py`).
- `5aef21f0` (2026-10-31 19:50 -07:00): the `CLAUDE.md` note documenting SHA1-Hulud as
  Single-Solve-only and un-regradable once team-solved.
- `7620651a`, `df8f4534`, `f0e20aa1` (2026-10-31 into 2026-11-01): Day 31 and later gap-closure work
  that carried forward `progress.md` and `orchestrate.py` fixes; not specific to Day 30's own
  challenges.
