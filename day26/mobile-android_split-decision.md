# Split Decision

| | |
|---|---|
| Day / Lane | 26 / Mobile Android |
| Challenge ID | `mobile-android-day-26-split-decision` |
| Technique | split-APK (`.apks`) bundle: answer piece in a feature split, later computed by running the app's own bytecode |
| Single Solve | Independent |
| Time Trial | Independent |
| Mastery | Independent |

## Summary

Day 26 was the first mobile challenge to ship an `.apks` bundle (a ZIP of `base.apk` plus a feature
split) instead of a single APK. In Single Solve the answer was a v1 `split` codec whose one missing
piece sat in the feature split's JSON; reassembling it gave the code. Time Trial and Mastery moved
to the v2 `analysis` schema, where nothing is stored and the base APK's own `Release.code()` reads
the feature split through an app helper. We ran that bytecode in our Dalvik interpreter with a
split-asset hop. All three modes went CORRECT on the first submission of the release run.

## The challenge

Prompt: "Reassemble evidence distributed across base and configuration splits." Each question asked
for the "exact backend answer code" for a case id and package, formatted as `ANS-` plus 16
uppercase hex characters (`pattern`, `exact` normalization).

- **Single Solve:** one answer (`answer`), one delivery, `split_decision.apks`. No deadline.
- **Time Trial:** one answer, `split_decision_time_trial.apks`. The ranked window was 50 s.
- **Mastery:** ten answers (`set1.answer` ... `set10.answer`), ten deliveries. Sets 1-9 were plain
  `.apk` files whose case ids carry earlier-day numbers (`MOB-02-...`, `MOB-05-...`), and set 10
  was an `.apks`. The ranked window was 320 s.

## Solution

1. **Triage.** Unzipping the `.apks` gave `base.apk` and `feature.apk`. The base APK's
   `assets/huntress_puzzle.json` used `codec: "split"` with `parts` and `order`, but one entry of
   `parts` was an empty string. The base `classes.dex` only forwarded a `proof=` value beside a
   decoy `client_secret`, as in earlier mobile days, so it held nothing useful.
2. **Find the missing piece.** `feature.apk` carried `assets/huntress_feature.json` with a `part`
   string and a `part_index`. `part_index` pointed at the empty slot in the base `parts`.
3. **Reassemble.** Fill `parts[part_index]` with the feature piece, then place each part with the
   library's split rule: `order[i]` is the output slot of `parts[i]`. The solver walked nested ZIPs
   (an `.apks` contains APKs), so it did not depend on the bundle's layout:

   ```python
   for j, f in enumerate(feats):
       k = f.get("part_index")
       if isinstance(k, int) and 0 <= k < n and parts[k] == "":
           parts[k] = f["part"]
   out = [""] * n
   for i, slot in enumerate(order):
       out[slot] = parts[i]
   return "".join(out)
   ```

4. **Submit.** The result matched `ANS-` + 16 uppercase hex and was accepted on the first call:
   `ANS-7E37D134754DEA01`.

## Time Trial and Mastery

The Single Solve solver failed on the first regular Time Trial delivery with `unexpected codec
'analysis'`. The Time Trial `.apks` used the v2 puzzle schema, which stores no answer data. The
answer is computed on device:

- `Release.code()` calls `Util.mergeFeature41`, which calls the app's `Assets.readSplit(name)`.
- `readSplit` scans `getApplicationInfo().splitSourceDirs` and reads `assets/<name>` from the first
  split that holds it into a 16-byte buffer.
- The library's Dalvik interpreter (MOBILE-ANDROID-DALVIKVM-001) raised on
  `Context.getApplicationInfo()`, because it had no notion of splits.

We taught the interpreter that one hop. It serves `readSplit` from the bundle's other split APKs
(behind a call allowlist), and it takes the buffer size from the bytecode. To confirm the split
asset was really consumed, we flipped one bit in the feature asset, and the result changed. The
same hop also covered the `.apks` in Mastery set 10. The other nine Mastery sets are single APKs, so
they ran through the normal asset-codec/VM paths with no split involved.

The first regular Time Trial and Mastery rehearsals were CORRECT in one call each, and then the
ranked attempts went CORRECT in one call each (solve times 2.65 s and 6.3 s locally). The Mastery
bundle mixed `.apk` and `.apks` deliveries, so the code that gathered deliveries could not filter by
the `.apk` suffix.

## Dead ends and gotchas

- **Delivery extension.** The `.apks` suffix was new, and our triage tooling at the time reported
  "no input files" or no findings on it. Selecting deliveries by `.apk` would have dropped Mastery
  set 10 and the whole Single Solve and Time Trial deliveries.
- **The codec changed between modes.** Single Solve was v1 `split`, while Time Trial and Mastery
  were v2 `analysis`. A solver written against one mode's start response fails on the next. Dispatch
  on the `codec` field in `huntress_puzzle.json`, not on the challenge or day.
- **Decoy path.** The `client_secret`/`proof=` forwarding in `classes.dex` is a decoy. The real
  value comes from the puzzle asset, or, in v2, from running `Release.code()`.
- **Alternate reading, never needed.** The Single Solve solver kept a second candidate that
  inserted the feature piece at its `part_index` in the final text instead of in the parts list. The
  primary reading was accepted first, so the alternate was never submitted.
- **Our own tooling, one line.** A later ranked re-run against the same bundles failed with
  `split asset ... not found` because a leaf-local subclass of the interpreter shadowed the
  library's native split support. We fixed it by delegating to the library, and the re-runs went
  CORRECT (see the day overview).

## Takeaways

- **Split APK bundles:** treat an `.apks` as a ZIP of APKs and walk it recursively. Look for the
  feature split's own JSON/asset and for an empty placeholder in the base.
- **Do not hand-decode a v2 `analysis` APK.** Run the app's own bytecode and supply only what it
  reaches for. For split bundles, that is the `splitSourceDirs` lookup. Check that the new input is
  really consumed (a one-bit flip is enough).
- **Dispatch on the schema field** (`codec`), because modes of the same challenge can change schemas.
- We promoted `.apks` support into `asset_codecs.py`, `dalvik_mini_vm.py` and `apk_triage.py`, so a
  future split bundle needs no per-leaf solver.

## Evidence

Leaf base: `challenges/day26/mobile-android/mobile-android-day-26-split-decision/`

- `single-solve/journal.md`, `single-solve/start_response.json`, `single-solve/accepted_plan.json`
- `single-solve/artifacts/cand-adapter/solve.py` (v1 reassembly, then v2 via the library VM)
- `time-trial/ranked-1/result.json`, `time-trial/regular/result.json`, `time-trial/ranked-3/result.json`
- `mastery/ranked-1/result.json`, `mastery/regular/result.json`, `mastery/ranked-3/result.json`
- `challenges/day26/_run/librarian/mobile-android-day-26-split-decision.md`,
  `challenges/day26/_run/lane-mobile-android-day-26-split-decision.log`
- Library: MOBILE-ANDROID-CODECS-001 (split codec), MOBILE-ANDROID-DALVIKVM-001 (`readSplit` hop)
- `progress.md` (grep `^| 26 .*split-decision`)
