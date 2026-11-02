# White Box Key Recovery

| | |
|---|---|
| Day / Lane | 29 / Mobile Android |
| Challenge ID | `mobile-android-day-29-white-box-key-recovery` |
| Technique | Static asset decoding (new v1 `table` codec, step recovered from a known prefix); v2 builds solved by running the APK's own `Release.code()` in a Dalvik interpreter |
| Single Solve | Independent (1 submission call, CORRECT) |
| Time Trial | Independent (regular and ranked attempts, 1 call each, CORRECT) |
| Mastery | Independent (regular and ranked attempts, 1 call each, CORRECT) |

## Summary

The title suggests white-box cryptanalysis, but the Single Solve was a small static-decoding task:
the APK's puzzle asset stored the answer as hex bytes under a per-index additive "table" transform,
and the missing step could be recovered from the known `ANS-` answer prefix. The Time Trial and the
13-APK Mastery used a different build style (no stored answer; the value is computed at runtime),
which our existing Dalvik interpreter handled without changes. All three modes were CORRECT on the
first submission.

## The challenge

Prompt: "Recover an equivalent key operation from challenge-owned table transformations, not a
third-party protected binary." Each question named a case ID and Android package
(`MOB-29-...`, `com.huntress.mobile.d29...`) and asked for "the exact backend answer code returned
for this case".

- Single Solve: one APK (`white_box_key_recovery.apk`), one answer id `answer`.
- Time Trial: one APK, one answer id, 50 s server window on the ranked attempt.
- Mastery: 13 APKs delivered as `set1`..`set13`, 13 answer ids, 410 s window on the ranked attempt.
  The case IDs and packages in Mastery carried earlier-day numbers (`d01`, `d02`, ...), i.e. older
  daily builds repackaged into one bundle.
- Answer format in every mode: `ANS-` followed by exactly 16 uppercase hex characters, exact match.

## Solution

Single Solve:

1. Triage the APK. `classes.dex` carried theme constants (`CODEC=table`, `CONTROL=white_box_table`)
   and only forwarded the answer value beside a decoy `client_secret`; the real data was in
   `assets/huntress_puzzle.json` (schema `huntress.mobile.puzzle.v1`).
2. The asset held a `key` of 41 and 20 bytes of hex `data`, with codec `table`. 20 bytes matches
   the length of `ANS-` plus 16 hex digits, so this was the answer stored under a transform.
3. The per-index step was not in the asset. Subtracting the known plaintext `ANS-` from the first
   four data bytes gave offsets 41, 44, 47, 50, i.e. `key + 3*i`. So
   `plain[i] = (data[i] - key - 3*i) mod 256`.
4. Applying that to all 20 bytes produced a string that fit `ANS-[0-9A-F]{16}` exactly, which is
   strong confirmation since a wrong step would not land in the hex alphabet. The result was
   `ANS-AB1D78DC8BA4D98C`, accepted on the first call.

The core of the decoder (taken from the rubric candidate) recovers the step from the prefix instead
of hard-coding 3:

```python
raw = bytes.fromhex(a["data"]); key = int(a["key"]); prefix = b"ANS-"
s = (raw[1] - raw[0] - (prefix[1] - prefix[0])) % 256      # step from the known prefix
txt = bytes((b - key - s * i) % 256 for i, b in enumerate(raw))
assert re.fullmatch(r"ANS-[0-9A-F]{16}", txt.decode("ascii"))
```

## Time Trial and Mastery

Time Trial and Mastery did not use the `table` codec. Per the librarian note and lane log, every
delivery (1 for Time Trial, 13 for Mastery) was a v2 `analysis` build: the asset carries no stored
answer, and the value is produced on-device by the APK's own `Release.code()`. We ran that method
in a small Dalvik interpreter instead of decoding anything by hand. The interpreter needed no
changes for this day.

The submitted solver was a thin wrapper that routes by schema: v1 assets go to the asset-codec
library, v2 builds go to the VM, and it also looks for a `Release` class in dependency DEX files.
Wall-clock for the ranked attempts, including download and submission, was 3.08 s for the Time
Trial (50 s window) and 8.77 s for Mastery (410 s window). Both regular (untimed) rehearsals and the
ranked attempts drew fresh APKs, so each had different answers; the same solver went CORRECT on
call 1 each time.

## Dead ends and gotchas

- The title and prompt point at white-box key extraction, but nothing needed that. The answer was
  never derived from a cipher; it was only obfuscated by a trivial per-index offset.
- A v1-only candidate (the `table` decoder alone) passed Single Solve but failed the pre-submission
  lint for Time Trial and Mastery, because it produced no value for the v2 builds. A solver written
  from the Single Solve alone would have broken on the other two modes; the schema split matters.
- The step is not stored anywhere in the asset, so the decoder depends on the known `ANS-` prefix.
  The rubric candidate also tried neighbouring steps and plain subtract/xor as lower-priority
  alternates, but only one reading matched the answer pattern, so no alternates were needed.
- The Game Master timing data for the ranked attempts showed a negative solve duration (server
  `solved_by.at` earlier than `started_at`); we treated it as clock skew and relied on the local
  timing above.

## Takeaways

- When an answer has a rigid format, a known prefix (`ANS-`) is a free crib: it gives you the
  unknown step or key of a simple additive/xor transform directly, and a full-pattern match on the
  rest of the output confirms it.
- Decide the decoding path by asset schema, not by challenge title or mode: v1 `table` data is
  static; v2 `analysis` builds need the APK's own bytecode executed. Running the app's own method
  (Dalvik interpreter) scaled to a 13-APK Mastery in about 9 s with no per-set work.
- Techniques to search for: "asset codec", "known-plaintext prefix recovery", "Dalvik interpreter /
  run `Release.code()`".

## Evidence

- `challenges/day29/mobile-android/mobile-android-day-29-white-box-key-recovery/single-solve/journal.md`
  (adapter and rubric findings, librarian outcome) and `.../single-solve/accepted_plan.json`
- `.../single-solve/artifacts/cand-rubric/solve.py` (table decoder), `.../cand-adapter/solve.py`
  (schema-routing wrapper used for Time Trial and Mastery)
- `.../time-trial/{regular,ranked-1}/result.json` and `.../mastery/{regular,ranked-1}/result.json`
  (all CORRECT, 1 call each) with their `accepted_plan.json` and `start_response.json`
- `challenges/day29/_run/librarian/mobile-android-day-29-white-box-key-recovery.md`,
  `challenges/day29/_run/lane-mobile-android-day-29-white-box-key-recovery.log`
- `progress.md` rows for Day 29 (Solved, 2026-10-29)
