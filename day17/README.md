# Day 17 overview

**At a glance:** 5 challenges (one per lane, no web lane yet), 15 modes. By provenance label: 10
Independent, 5 Journal-recorded, 0 Local mark only, 0 Team, 0 n/a. Theme: a day of reuse. Nothing
in any lane needed a new method. Each challenge was an earlier day's decoder or bank plus a small
new piece (one new codec, one new field name, two new binary formats), and every mode went CORRECT
on the first submission call. The traps were in the prompts, which described tasks that were not in
the data.

## Challenges

| Lane | Challenge | Technique | Single Solve | Time Trial | Mastery |
|---|---|---|---|---|---|
| Warmups | [Bacon Bits](warmups_bacon-bits.md) | a/b five-letter groups read as hex nibbles, not Baconian; 17-record numbered chain in Mastery | Journal-recorded | Independent | Independent |
| Metadata Forensics | [Hear Me Out](metadata-forensics_hear-me-out.md) | WAV `fmt ` chunk to an ffprobe-style codec name; 22-field evidence bundle, 5 nested copies in Mastery | Journal-recorded | Independent | Independent |
| Mobile Android | [Name Collapse](mobile-android_name-collapse.md) | new `indexed` permutation codec in `assets/huntress_puzzle.json`; shrunk DEX is a decoy | Journal-recorded | Independent | Independent |
| Binary Reverse Engineering | [Decoder Ring](binary-reverse-engineering_decoder-ring.md) | XOR + CRC32 HCFG records, scanned statically across a d01-d17 recap bundle (ELF, PE, JAR, WASM) | Journal-recorded | Independent | Independent |
| Script Analysis | [Fetch Quest](script-analysis_fetch-quest.md) | descriptor-bank reuse (127 specimens, 1,139 fields) plus structural Batch-control classification | Journal-recorded | Independent | Independent |

How to read the labels (each page gives the evidence):

- **Journal-recorded (all five Single Solves).** Each journal records our own submission as CORRECT, and
  `progress.md` has a "Solved" row dated 2026-10-20 for each. No `result.json` or Game Master response
  was saved for any Single Solve. The `submission-attempt-log.json` files in each leaf are our own
  hand-written logs (three different formats across lanes), not graded responses, so we did not count them
  as result files.
- **Independent (all ten Time Trial and Mastery modes).** The original attempts were untimed
  (`deadline: null`) and are journal-recorded only. The label rests on saved `ranked-N/result.json` files
  from later timed reruns, all of which read CORRECT in one call. Counting the saved reruns: Warmups 2 TT +
  2 Mastery, Metadata 1 + 3, Mobile 2 TT + 2 Mastery result files (one of the two Mastery files is a recovery,
  see below), BRE 1 + 1, Script 1 + 1: 16 files in total.
- **Which mode's provenance is thinnest.** Mobile Mastery: its 2026-10-29 rerun lost its download and
  the saved `ranked-recover/result.json` has no window or timing fields; the clean timed evidence is the
  2026-11-01 `ranked-1`.

## What the day had in common

- **Everything was cumulative.** Script Analysis was a strict superset of the Day 15 bank (113 new IDs: Ruby and
  AutoHotkey polyglots, a Ruby Rack host, two more advanced-Bash features), and the Day 15 decoder and mapper ran
  with no code changes. BRE's d01-d15 members were byte-identical to the Day 15 bundle, so only the
  WebAssembly (d16) and d17 ELF blocks were new. Mobile Mastery delivered five APKs, one per earlier day's codec
  (xor, reverse, hex, caesar, indexed). Warmups Mastery was a 17-record chain reusing Days 1-16's codecs.
  Metadata Time Trial and Mastery reused the library's mixed-evidence extractor.
- **Prompts were decoys.** Warmups said "Bacon" (the data was nibbles; the "sixteen guests" line was the real
  hint). Metadata talked about transcoded voice notes and "the stream that breaks a sender's story" (the one
  audio question was a single codec field). Script promised reconciling source with network evidence (no such
  question or file existed). Mobile promised reversing identifier shrinking (the answer sat in the asset). BRE
  said "recover the decoder", but the question text spelled out the whole record layout, so a scan, XOR, CRC32
  and JSON loop was the decoder.
- **Time Trial and Mastery changed fixtures, not mechanism.** Script's Time Trial changed 325 of 1,139 values;
  BRE Time Trial brought a stripped d01 and Mastery mixed PE (sets 1, 3, 5) with ELF (sets 2, 4); Metadata's
  asked note/roster attribute varied per set; Mastery bundles were 5 nested archives or 5 separate deliveries in four
  lanes (Warmups: one ZIP of 83 blobs). `gm.py artifact` fetches only the first delivery, so Mobile and BRE had to
  download the rest from `artifacts[]`.
- **Original solves were untimed.** Every journal records `deadline: null` for the first Time Trial and Mastery
  attempts, so those solves carry no speed ranking; the timed reruns are described under "Tooling and process".

### Validator and format traps seen across lanes

No submission was rejected on Day 17, so these are conventions that were accepted, not corrections.

| Lane | Trap or accepted convention |
|---|---|
| Warmups | Last record number fixes the codec, and the chain length is 17, not the longer chain the old decoder assumed. The bare slug was accepted under `trim`. |
| Metadata | `pcm_s16le`, the ffprobe codec name (not `wav`, not format tag `1`), validated only for 16-bit PCM. The note/roster attribute asked differs per Mastery set, and the PE member's SHA-256 differs per set, so every set's hashes have to be recomputed. |
| Mobile | `order[i]` is the output slot of `data[i]`; only the `ANS-[0-9A-F]{16}` pattern distinguishes the direction. Pre-unlock Mastery responses listed different case IDs than the unlocked one. |
| BRE | Spellings carried over from Day 15: `ELF64`, `x86-64`, `ET_EXEC`, `little-endian`, `PE32+`/`AMD64` for native PE, `PE32_CLR`/`I386` for managed, lowercase enum tokens, a stripped ELF answering `[]` and `none`. All 204 and 1,020 fields were accepted in one call. |
| Script | Descriptor `attack` field beats the graph reading for the new polyglot and host families. A one-item `FOR /L ... DO CALL :label` Batch control was `call_subroutine` here (Day 11 had taken `for_once`), so Batch controls are classified structurally and never copied across days. |
| All batch submissions | Every answer value is a string; the complete map was one call per mode (15 calls in total for the original attempts, from the attempt logs). |

CLAUDE.md carries the lasting versions of the Script and BRE items (structural Batch controls, the
descriptor-field rule, `SHT_SYMTAB` naming, Mastery PE/ELF mixing). Day 17 has no bullet of its own there.

## Mistakes in our own tooling or process

Collected from the challenge pages and checked against the leaves.

- **Isolated interpreter hides `pyzipper` (Script, BRE).** Both lanes use AES ZIPs (password `infected`), and `python -I`
  hides user-site packages. In the original solves this was caught up front: the Script extraction script appends
  the user site path itself and the BRE helper was run without `-I` for the extraction only. No attempt was lost
  to it. The Day 17 `pass.log` files show one line each, all CORRECT, so we found no `pyzipper` failure for Day 17
  reruns (that failure mode is documented in `tools/ranked/PROCESS.md` for other days).
- **Mobile Mastery rerun lost its download (2026-10-29).** The artifact download returned HTTP 500 after the timed attempt
  was already issued. The commit message for `34e7f1a0` says it was recovered live with `finish_open.py`
  (`ranked-recover`, CORRECT), and a second commit (`73432600`) added the recovery's scratch files later because
  only the result files had been staged. A fresh timed rerun on 2026-11-01 (`ranked-1`, 170 s window, 1.74 s) replaced it
  as the clean evidence.
- **Metadata journal assumption.** The Mastery journal treated the non-image members as identical across sets; a later
  library note found the PE member's SHA-256 differs per set. The extractor recomputes hashes per set, so no answer
  was affected.
- **Warmups Mastery `answer_map.py` docstring is stale.** It still names an older day although the code enforces 17 records.
- **Bookkeeping.** There is no `challenges/day17/submission-attempts.md`; counts here come from per-leaf attempt logs and
  journals. Those logs use three different layouts. `progress.md` dates all rows 2026-10-20, while the BRE journals
  say "started 2026-10-21T04:34Z" and "04:39Z" (UTC); we assume a time-zone difference but no file says so.
- **No solver wrapper existed for BRE on the first ranked run (2026-10-28).** The Day 15 BRE solver was run against the
  saved accepted Single Solve and Time Trial maps offline first (204/204, lint clean) before the timed attempt. This is
  a process step, not an error, but the other lanes' Day 17 wrappers were only written the next day (`34e7f1a0`).
- **Slow single ranked pass.** Mobile Time Trial `ranked-1` took 10.66 s total, 8.51 s of it in the submit call. The
  next rerun took 1.54 s. We did not investigate the cause.

## Tooling and process changes around this day

The first commit containing Day 17 is `0e61154c` (2026-10-22, "Add Day 17 challenge work"), after the
2026-10-20 solves, so the original solving has no commits of its own. What the history shows:

- `3bcba4e6` (2026-10-22): the technique library, which the five journals cite (for example the Day 17 entries
  for the indexed codec, the a/b nibble codec, WAV codec names, the BRE recap bundle and descriptor-bank reuse).
- `356df6b0` (2026-10-28): first Day 17 ranked results (BRE Time Trial rank 15 at 4,598 ms per its journal; Warmups
  Time Trial and Mastery).
- `34e7f1a0` (2026-10-29): thin per-challenge ranked solvers for Days 17-20 built from the library engines, each
  matched to its saved submitted map offline before any live attempt. Per the message, metadata, script and
  BRE ran on a second machine and mobile ran on the main workstation because the second machine lacks
  `capstone`. Its message says 37 of 39 issuable non-web modes went CORRECT on the first submission.
- `73432600` (2026-10-29): the missing recovery artifacts for Mobile Mastery (above).
- `38668151` and `f0e20aa1` (2026-11-01): the "top-30 gap closure" sweep. For Day 17 it added `ranked-2` for Warmups
  (Time Trial and Mastery), Mobile Time Trial and Metadata Mastery, `ranked-1` for Mobile Mastery and `ranked-3` for
  Metadata Mastery. All CORRECT in one call. These were scoreboard attempts, not the prize measure.
- Day 17 produced no `CLAUDE.md` change that we could find (no "Day 17" or "d17" mention there).

## Effort

Submission calls from the attempt logs and journals. Every original attempt was one complete-map call.

| Lane | Single Solve | Time Trial (original) | Mastery (original) | Timed reruns saved (all CORRECT, 1 call each) |
|---|---|---|---|---|
| Warmups | 1 (1 field) | 1 (1 field) | 1 (83 fields) | 2 TT, 2 Mastery |
| Metadata | 1 (1 field) | 1 (22 fields) | 1 (110 fields) | 1 TT, 3 Mastery |
| Mobile | 1 (1 field) | 1 (1 field) | 1 (5 fields) | 2 TT, 1 Mastery + 1 recovery |
| BRE | 1 (204 fields) | 1 (204 fields) | 1 (1,020 fields) | 1 TT, 1 Mastery |
| Script | 1 (1,139 fields) | 1 (1,139 fields) | 1 (5,695 fields) | 1 TT, 1 Mastery |

Timed windows seen in the rerun `result.json` files: Warmups 30 s (TT) and 100 s (Mastery); Metadata, Mobile and
Script 50 s and 170 s; BRE 80 s and 325 s. Local `t_total` per pass ranged from 1.3 s (Warmups TT, second rerun)
to 20.1 s (BRE Mastery, dominated by the large archives). The server-side scoreboard rank is recorded only for the
BRE Time Trial.
