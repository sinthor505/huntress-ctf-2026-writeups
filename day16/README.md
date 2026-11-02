# Day 16 overview

**At a glance:** 5 challenges (one per lane, no web lane yet), 15 modes. By provenance label: 15
Independent, 0 Journal-recorded, 0 Local mark only, 0 Team, 0 n/a. Theme: a day where the prompt and the
challenge ID were thematic cover and the real work was reuse. Four of the five prompts describe something
the question list never asks about (mail threading, a JNI authorization bug, WebAssembly imports and
indirect calls, obfuscation equivalence); the fifth (Warmups) hid its codec in the title ("Dot Com", dot and
dash). Each lane was solved by extending the previous day's mapper by one small block.

## Challenges

| Lane | Challenge | Technique | Single Solve | Time Trial | Mastery |
|---|---|---|---|---|---|
| Warmups | [Dot Com](warmups_dot-com.md) | Morse whose symbols spell hex digits; 16-record numbered codec chain in Mastery | Independent | Independent | Independent |
| Metadata Forensics | [Return to Sender](metadata-forensics_return-to-sender.md) | `.eml` `Message-ID` header (brackets kept) inside the cumulative 21-field evidence bundle | Independent | Independent | Independent |
| Mobile Android | [JNI Boundary](mobile-android_jni-boundary.md) | Asset value cross-checked against the one `ANS-` string in a native library's `.rodata`; decoy client secret | Independent | Independent | Independent |
| Binary Reverse Engineering | [Wasm Up](binary-reverse-engineering_wasm-up.md) | Day 15 recap mapper plus a new d16 WebAssembly block (HCFG record in a data segment) | Independent | Independent | Independent |
| Script Analysis | [Same Same, But Different](script-analysis_same-same-but-different.md) | Inert descriptor markers decoded offline; Day 15 bank plus 69 new Ruby / advanced-Bash questions | Independent | Independent | Independent |

How to read the labels (each page gives the evidence):

- **Independent (all 15).** Every mode directory holds `artifacts/submission-attempt-01-response.txt`
  reading `CORRECT` (checked for all 15), and each Single Solve, Time Trial and Mastery was CORRECT on the
  first submission call. This follows the Days 1-10 rule that a saved submission response counts. None of
  the leaves has a `result.json`, `accepted_plan.json` or `manual_solve.json` for the original attempts, so
  that is the only per-mode graded evidence for them.
- **Ranked reruns.** Time Trial and Mastery additionally have `ranked-N/result.json` files that read
  CORRECT: 19 in total (Warmups 2 TT + 1 Mastery, Metadata 1 + 1, Mobile 2 + 2, BRE 5 + 3, Script 1 TT + 1
  Mastery). There is also one failed ranked Script Mastery attempt (`ranked-1`, `pass.log` only, see below).
- **Original attempts were untimed.** The journals and `progress.md` call them untimed; `progress.md` dates
  them 2026-10-20. The ranked reruns are dual-route (`route: dual` in every `result.json`), with windows of
  30 s (Warmups TT), 50 s (Metadata, Mobile, Script TT), 80 s (BRE TT), 95 s (Warmups Mastery), 170 s
  (Metadata, Mobile, Script Mastery) and 320 s (BRE Mastery).

## What the day had in common

- **Cumulative reuse.** Warmups Mastery is the first 16 entries of the numbered codec chain that Day 19
  later extends, so the mapper needed only a record-count change. BRE copied the Day 15 mapper and added the
  d16 block (160 fields in Single Solve and Time Trial, 800 in Mastery). Script Single Solve was a strict
  superset of Day 15's 1,026 IDs with 69 new ones, and the Day 15 structural mapper ran unchanged.
  Metadata's 21-field bundle is the previous days' template with an EML added.
- **The question list, not the prompt, defines the task.** Metadata asked only for one header; Mobile never
  needed the native logic traced; BRE asked nothing about wasm imports, memory or indirect calls; Script
  asked nothing about equivalence or diffs. Diffing the question IDs against the previous day was the
  cheapest first step in every lane.
- **No lane needed a rejected submission.** The journals and logs record zero rejected answers for the
  original attempts, so the usual validator-quirk material is thin this day.
- **Time Trial and Mastery changed fixtures, not mechanism.** Script Time Trial differed from Single Solve in
  318 of 1,095 values, so nothing carried over as a literal map; Mastery bundles arrived as several artifacts
  (Mobile 5 APKs, BRE 5 archives) or nested ZIPs (Metadata 5, Script 5), and Mobile Mastery mixed five
  codecs (`plain`, `xor`, `xor`, `base64`, `native`) in one bundle.

### Validator and format traps seen across lanes

| Lane | Trap |
|---|---|
| Metadata | The `Message-ID` answer keeps its angle brackets (`compat32` and `policy.default` both preserve them). Roster/note answer IDs in Mastery are resolved by regex on the question ID, with the set number from the prefix. |
| Mobile | The `SECRET-` client secret in the DEX is a decoy; the answer is the `ANS-` asset value. Mastery delivers several APKs and `gm.py artifact` fetches only the first. |
| BRE | The stripped d01 specimen makes symbol and disassembly answers `[]` with `has_symtab` false; Mastery deliveries 1, 3 and 5 carried PE specimens for d02, d04 and d10, so classification has to follow the member, not the day. |
| Script | Polyglot and host specimens' own `attack` field wins over the graph rule; Batch `*-control` answers are structural (the same shape had been `for_once` on another day). |
| Warmups | `E` is a single dot, and Mastery record 11 is unpadded base64. The answer is the slug at the end of the concatenated decode, in plain form. |

## Mistakes in our own tooling or process

Collected from the challenge pages and checked against the leaves.

- **Script Analysis (ranked Mastery).** The first ranked attempt failed before submitting. The `progress.md`
  row says Windows Defender's real-time protection quarantined an extracted fixture (`staged-handoff.js`) the
  moment it was written; the leaf only holds a `pass.log` ("solver failed", traceback), so the Defender cause
  rests on that row. Per `tools/ranked/PROCESS.md` the attempt is issued before the solver runs, so it was
  spent. The solver was rewritten to decode the archive in memory and the next ranked attempt went CORRECT.
  The original Day 16 solves extracted to disk and were not affected.
- **Mobile and Metadata (ranked reruns).** Neither had a ranked solver when the sweep reached them, so each
  got a thin delegate (Mobile to the Day 15 generic APK solver, Metadata to the library's generic
  `question_facts.py`). Both were checked offline first (Metadata 21/21 and 105/105 against the accepted
  answers). The gap was ours: the per-day mappers had been left in the leaves rather than promoted into
  the library.
- **BRE.** The Day 15 mapper depended on `objdump`, which the workstation lacked. We wrote offline fallbacks
  (constant-return stub decoding, hand-read COFF symbols) and validated them by reproducing the Day 15
  outputs exactly before submitting. The ranked wrapper also has to rebuild the member manifest from each
  fresh zip.
- **Warmups.** The mapper's docstring still says 19 records (it was adapted from the Day 19 chain, count
  changed to 16 in code). Cosmetic.
- **Mobile (documentation).** The three Mobile journals were written retroactively (their closing
  line says "retroactive sweep 2026-10-20"; the Single Solve journal says they were written after Day 19), so
  the narrative was rebuilt from extractors and responses. The native-export analysis (a length-check export
  gating the same rodata string) came from a later disassembly review, not the original solve.
- **Ledger bookkeeping.** `challenges/submission-ledger.json` does not match the saved evidence for every
  mode. It has no entry for the Warmups or Mobile Single Solve and lists Warmups TT 2 and Mastery 1 (saved
  evidence: 1 original + 2 ranked, and 1 + 1). BRE (TT 6, Mastery 4), Script, Metadata and Mobile TT/Mastery
  counts do match original plus ranked CORRECT submissions. We did not establish why the Warmups and Mobile
  Single Solve calls are missing. There is no `challenges/day16/submission-attempts.md`.

## Tooling and process changes around this day

The repository history does not capture the original solving (2026-10-20 per `progress.md`); the first
commit of the Day 16 leaves is later:

- `423b10c2` (2026-10-22): "Add Day 16 challenge work", i.e. the Single Solve, Time Trial and Mastery
  leaves with journals, mappers and saved responses. The message has no further detail.
- `df8f4534` (2026-10-31): carry-over commit that includes the new ranked solvers for the Script and BRE
  challenges and, per `progress.md`, the Metadata and Mobile delegates. This is the first and only commit
  touching those four solver directories. Warmups has no dedicated ranked solver; its reruns used the generic
  warmups codec-chain tool unchanged.
- `38668151` (2026-11-01): "top-30 gap closure sweep, Days 2-30". Its message says the Days 2-20 reruns ran
  from a separate always-on VM through `tools/ranked_pass.py`, which accounts for the Day 16 ranked results
  dated 2026-10-31 to 2026-11-01.
- `f0e20aa1` (2026-11-01): continued the sweep; its message lists Wasm Up among the modes newly CORRECT
  after a cap-reached stall (BRE Time Trial ledger count 6 includes these reruns).

The ranked reruns are speed-scoreboard attempts only; they do not count toward the prize ranking, and the
labels above rest on the original saved responses as well as the reruns.

## Effort

Submission calls, from the saved responses, `submission-attempt-log.json` files and `result.json` files:

| Lane | Single Solve | Time Trial (original) | Mastery (original) | Ranked reruns CORRECT |
|---|---|---|---|---|
| Warmups | 1 | 1 | 1 | 2 TT, 1 Mastery |
| Metadata | 1 | 1 | 1 | 1 TT, 1 Mastery |
| Mobile | 1 | 1 | 1 | 2 TT, 2 Mastery |
| BRE | 1 | 1 | 1 | 5 TT, 3 Mastery |
| Script | 1 | 1 | 1 | 1 TT, 1 Mastery (plus 1 failed pre-submit) |

Every original mode was one call (160 to 5,475 fields in a single submission). Ranked total times
(`t_total`): Warmups 1.8-6.3 s, Metadata 3.5-4.9 s, Mobile 1.5-5.0 s, Script 7.1-8.5 s, BRE 7.7-11.8 s (TT)
and 21.8-26.6 s (Mastery), the BRE figures dominated by the archive downloads. Solve duration for the
original attempts was not recorded in the leaves.
