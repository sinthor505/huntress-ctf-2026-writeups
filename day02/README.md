# Day 2 overview

**At a glance:** 5 challenges, 15 modes (Single Solve, Time Trial and Mastery for each lane; no web or welcome
challenge this day). By provenance: 9 Independent, 5 Journal-recorded, 1 Local mark only, 0 Team, 0 absent.
Theme: the first "orientation" day, where the prompt is flavor text and the graded questions are plain facts
(ROT13, magic bytes, a puzzle asset, an obfuscated descriptor, symbol-table queries), with Time Trial and Mastery
windows of only 20 to 110 seconds.

## Challenges

| Lane | Challenge | Technique | Single Solve | Time Trial | Mastery |
|---|---|---|---|---|---|
| Warmups | [Half a Turn](warmups_half-a-turn.md) | ROT13 over a text blob; numbered base64/ROT13 records in Mastery | Local mark only | Independent | Independent |
| Metadata Forensics | [Looks Can Be Deceiving](metadata-forensics_looks-can-be-deceiving.md) | First 16 bytes of a named ZIP member as hex; nested ZIPs | Journal-recorded | Independent | Independent |
| Script Analysis | [Host With the Most](script-analysis_host-with-the-most.md) | Static descriptor recovery from obfuscated JScript/PowerShell; source-shape classifiers | Journal-recorded | Independent | Independent |
| Mobile Android | [Deep Link Drift](mobile-android_deep-link-drift.md) | Puzzle-asset decode (`reverse` codec) behind a decoy deep-link / `client_secret` surface | Journal-recorded | Independent | Independent |
| Binary RE | [Mangled](binary-reverse-engineering_mangled.md) | ELF/PE symbol-table queries (`_ZTV`/`_ZTI`) and an XOR+CRC32 config record | Journal-recorded | Independent | Journal-recorded |

What the labels rest on (the table gives the strongest evidence per mode):

- **Independent** means the leaf holds our own `result.json` reading CORRECT. For every Day 2 Time Trial and
  Mastery that is a file from the later ranked sweep (2026-10-25/26). The original live attempts of 2026-10-20
  (the audit re-solves) are backed only by their journals and `progress.md`; each of those modes is Independent
  because a ranked rerun file exists. Warmups Time Trial has four such files (kept under `untimed-{1,2,5,6}/`,
  a mislabeled directory name, see below), BRE Time Trial three, Warmups Mastery two, the rest one each.
- **Journal-recorded** means the journal records our own submission as CORRECT but no result file was saved:
  the four Single Solves (metadata, script, mobile, BRE) and BRE Mastery (CORRECT on its second submission).
- **Local mark only** is Warmups Single Solve: the journal says outright that no submission, solver or response is
  stored, and the only basis is the `progress.md` row marked "Player-directed local mark". BRE Mastery has no
  ranked rerun because the pool was `rotation_exhausted`.
- No mode is a team solve; there is no `team_solved.json` in any Day 2 leaf.

## What the day had in common and taught

- **The prompt is not the question.** Deep Link Drift talks about deep-link verification and Mangled about
  demangling and PE imports; neither needs any of it. The mobile answer sits in `assets/huntress_puzzle.json`
  and the BRE answers are hashes, header facts, a config record and symbol-name regexes. Read the question list
  (field names and formats) before the story text.
- **One bank, three modes.** Each lane's Single Solve mapper carried to Time Trial and Mastery unchanged apart from
  per-set handling: Mastery bundles are an outer ZIP of inner ZIPs/APKs, with `set1.` / `set2.` / ... prefixed IDs.
  Mapping deliveries to answer IDs had to come from the question text and the start response, not file order.
- **Day 1 carry-over.** Script Analysis builds on Day 1's PowerShell triad, and the first five BRE fields are a Day 1
  recap (`c_sha256`, `c_marker`, `c_has_symtab`, `c_recover_code`, `c_machine`).
- **Short windows regardless of the "untimed" default.** Time Trial and Mastery starts that did not ask for a timer
  still came back with deadlines: 20 s / 25 s (Warmups), about 50 s / 80 s (Metadata, Script, BRE), 50 s / 110 s
  (Mobile). Every pass was therefore scripted as one download, map, submit run after the mapper had been proved
  offline on the Single Solve copy.

### Validator and format traps seen across lanes

- **Batch-level `incorrect`.** The Game Master returns no per-field feedback on a failing batch. Splitting by family
  (Script: 10 submissions for 87 fields) or by `setN.` prefix (BRE Mastery: set 2 accepted first, then set 1) isolated
  the problem.
- **Fixtures differ per attempt and per mode.** Metadata's live Time Trial and Mastery had 1 and 2 questions; the later
  ranked ones had 6 and 12 (filename, size, MD5, SHA-1, SHA-256, prefix). Mastery sets can change container type:
  BRE Mastery set 1 swapped the C++ ELF for a MinGW PE, which changed `cpp_format` and kept section prefixes on the
  accepted symbol names. Warmups Mastery arrived as one ZIP on the first attempt and as eight separate blobs on the
  ranked ones.
- **Answer spelling.** All answer values are strings. Metadata hex is lowercase, no `0x`, trim-only normalization;
  BRE hex values carry `0x`; BRE `cpp_format` accepted `ELF64` and `PE32+`. Script style classifiers
  (`function-style`, `property-style`) are about source shape, scoped to a specific function and variable.
- **Mode IDs.** Time Trial and Mastery calls use `<id>/time_trial` and `<id>/mastery`.
- **Multi-artifact Mastery.** The generic artifact fetch returns only the first delivery; the others have to be
  fetched from the per-delivery links (Mobile, BRE).

## Mistakes in our own tooling and process

These are not challenge lessons; the pages point here.

- **Half a Turn (and the whole ranked sweep).** A "rehearsal" Time Trial on this day was a real ranked attempt: on a
  legacy route a plain `retry` is Hall-of-Fame eligible and carries a deadline, and `run_for_time:true` returns
  409 `timed_intent_not_available`. `CLAUDE.md` records that an "untimed rehearsal" on Half a Turn Time Trial was
  in fact four ranked attempts (scoreboard rank 85 -> 39). The scripted ranked passes had also been labeled
  "untimed", so their results sit in `time-trial/untimed-{1,2,5,6}/`, and `progress.md` records that the local
  evidence was only recognized as ranked results on 2026-11-01. The retry pool is shared and finite, so this spent
  attempts that were meant to be free.
- **Half a Turn (Mastery mapper).** The first mastery decoder guessed the codec per line (hex, base64, ROT13 fallback).
  It was CORRECT, but a heuristic; the later codec-chain tool selects by record number instead.
- **All Single Solves: "solved" marks without a graded result.** `progress.md` marked Day 2 Single Solve, Time Trial
  and Mastery as Solved on 2026-10-18 on the player's word ("Player-directed local mark"). The 2026-10-20 audit found
  no graded CORRECT on the server for those (attempts open or never made), so each mode was re-solved from fresh,
  SHA-256-verified bytes. The Warmups Single Solve was not redone and is still only a local mark.
- **Saved artifacts corrupted by line-ending translation.** The locally saved `bundle.zip` / `evidence.zip` files
  (Metadata, Script, Mobile, BRE pages) did not match the API's SHA-256; the cause was LF to CRLF conversion of
  binary files, and undoing it restored the hash. The fix was to re-download and verify the hash and `PK` magic
  before analysis, which `CLAUDE.md` now states.
- **Script Analysis, Single Solve.** The first full map was submitted with two wrong classifiers (`function-style` for
  a named function expression; `property-style` not scoped to the `eval` result), costing 10 of the 20 ledger
  submissions. This was our extractor's bug rather than an ambiguous question, though the validator's
  batch-level feedback made it expensive to find.
- **Metadata, ranked reruns.** The leaf does not hold the solver that produced the 6- and 12-field ranked passes (it
  lives under `tools/ranked/solvers/`), so the page cannot say how the extra digest fields were computed beyond
  "standard digests of the named member".
- **Leaves with no preserved result.** The Single Solve leaves and BRE Mastery stored no submission response, which
  is why they carry only journal-level provenance.

## Tooling and process around this day

Day 2 was played before version control (the repository's history starts 2026-10-22 with the commit that adds the
Day 2 leaves), so nothing about the original solves can be dated from commits. There was also no release
automation: no `_run/` folder, librarian notes or lane logs exist for this day, and none of the solves was
timed against a release. What files do support:

- **2026-10-18 / 2026-10-20 (progress rows, journals):** first "local mark" entries, then the retroactive audit and
  re-solve; the journals' library-promotion sections record what moved into `library/` (codec-chain tool and
  warmup gotchas, the BRE question-facts reader, the mobile asset-codec tool and its accepted-map regression, the
  script variant-corpus regression).
- **2026-10-25 (commit `1160dffa`, "Add ranked-attempt tooling and Day 2-5 ranked results"):** added
  `tools/ranked_pass.py`, per-challenge solvers for Days 2-5 under `tools/ranked/solvers/`, parity checks that run
  the same solvers on two machines, and the `CLAUDE.md` section "Timed ranking on older days". The Day 2 ranked
  results were committed with it.
- **2026-10-26 (`CLAUDE.md`, `progress.md`):** Days 2-5 ranked sweep recorded as done, 36 of 40 modes CORRECT on one
  attempt; BRE Mastery on Days 2-5 was `rotation_exhausted` (409, nothing consumed), which is why Day 2 BRE Mastery
  has no ranked file.
- **2026-11-01 (commits `38668151`, `f0e20aa1`, "top-30 gap closure sweep"):** BRE Time Trial reruns `ranked-2` and
  `ranked-3` (each CORRECT in one call, 1.93 s and 2.86 s locally against a 50 s window). Commit messages describe
  these as a rank-improvement sweep. Separately, the `progress.md` row that recovers the Half a Turn ranked
  evidence is dated 2026-11-01; I did not check which commit carried it.

## Timeline and effort

Only these dates are supported by files: local marks 2026-10-18; audit re-solves 2026-10-20 (about one submission per
mode, except Script Single Solve 10, BRE Mastery 2, per `progress.md`); ranked sweep 2026-10-25/26, one scripted pass
per mode (1.3 to 4.8 s total locally per pass); extra ranked reruns 2026-11-01. No per-challenge time-to-solve is
recorded for the original attempts.
