# Pending Intent Relay

| | |
|---|---|
| Day / Lane | 4 / Mobile Android |
| Challenge ID | `mobile-android-day-04-pending-intent-relay` |
| Technique | Static APK triage; the answer sits in a XOR + Base64 asset, not in the code the prompt describes |
| Single Solve | Solved (journal-recorded): the journal records our own `CORRECT` response; no graded response file saved |
| Time Trial | Independent (`time-trial/submission-response.json` CORRECT; ranked rerun `ranked-1/result.json` CORRECT) |
| Mastery | Independent (`mastery/submission-response.json` CORRECT; ranked rerun `ranked-1/result.json` CORRECT) |

## Summary

The prompt talks about recovering and redirecting a mutable `PendingIntent`. The APK has no such
logic. The case's answer code is stored in `assets/huntress_puzzle.json` as Base64 data XORed with a
single-byte key, so decoding that asset gives the answer without running anything. The same
decoder solved Time Trial and Mastery, where the only pressure was a 50 to 110 second window.

## The challenge

Prompt: "Recover and redirect a mutable PendingIntent while preserving every earlier component
check." Each question names a case (`MOB-04-<10 hex>`) and a package
(`com.huntress.mobile.d04.v<12 hex>`) and asks for "the exact backend answer code returned for this
case". The answer format is `ANS-` plus 16 uppercase hex characters, normalization `exact`.

- Single Solve: one APK (`pending_intent_relay.apk`, about 12.7 KB), one answer ID `answer`, untimed.
- Time Trial: one fresh APK for a different case and package, one answer, 50 second server countdown.
- Mastery: three APKs delivered together, answer IDs `set1.answer` .. `set3.answer`, 110 second
  countdown.

## Solution

1. Recon. The APK is a normal Android package; its SHA-256 matched the Game Master's value. The
   manifest package equals the one named in the question. The manifest declares an exported
   `ChallengeActivity` with a browsable filter (scheme `huntress04`, host `pending-intent-relay`,
   path prefix `/unlock`), plus `MainActivity` as launcher and the INTERNET permission. The app is
   not debuggable.
2. Reading the activity. `ChallengeActivity.onCreate` takes `answer` and `case_id` from the intent
   extras, overrides them from URI query parameters when the intent carries data, shows an error if
   either is missing, and otherwise builds a local evidence URL and loads it in a WebView. That fits
   the "relay" theme, but it only forwards a value. It does not compute one. The DEX contains no
   `PendingIntent` API reference at all, so the prompt is flavor text.
3. The real data. `assets/huntress_puzzle.json` is the schema `huntress.mobile.puzzle.v1`:

   ```json
   {"schema":"huntress.mobile.puzzle.v1","day":4,"codec":"xor","data":"XFNOMCstXCxeXCVYJSQkKiUpKy8=","key":29}
   ```

4. Decoding. Base64-decoding `data` gives 20 bytes (`5c534e30...`); XOR each byte with `0x1d`
   (the `key`, 29) and the result is `ANS-60A1CA8E89978462`, which already matches the required
   format. That was the Single Solve answer and the Game Master accepted it.
5. Solver. We wrote a small standalone script that opens the APK as a ZIP (recursing into nested
   `.apk`/`.zip` members), finds `assets/huntress_puzzle.json`, checks `codec == "xor"`, and applies
   the decode above:

   ```python
   ciphertext = base64.b64decode(puzzle["data"], validate=True)
   key = int(puzzle["key"])
   clear = bytes(byte ^ key for byte in ciphertext).decode("utf-8")
   ```

   It reads the codec and key from each APK's own asset rather than hard-coding them.

## Time Trial and Mastery

Both modes serve fresh builds with new case IDs, package names and keys, so nothing from Single
Solve carries over as an answer. The codec was the same (`xor`), so the script ran unchanged:

- Time Trial: one APK, one answer, decoded and submitted inside the 50 second window; accepted.
- Mastery: three APKs (`set1`..`set3`); the script was run once per APK and the three decoded values
  were submitted together under `set1.answer`, `set2.answer`, `set3.answer`; accepted.

Because the early Day 1 to 5 windows were so short, the solves were scripted end to end (issue,
download, decode, submit). A later ranked rerun with a per-challenge solver, which pairs each
question to its APK by package name, finished Time Trial in about 1.2 s and Mastery in about 1.5 s
(both CORRECT, one submission each). The decode itself took well under 0.1 s; issue, download and
submit dominated.

## Dead ends and gotchas

- The prompt describes a vulnerability class (mutable `PendingIntent` redirection) that is not
  present in the build. We did not need to exploit anything; checking which APIs the DEX actually
  references showed this immediately.
- Locally saved Day 4 APKs (Single Solve, Time Trial, Mastery) were later found to be damaged by
  LF to CRLF translation on save (zip reports a bad central directory, +22 to +35 bytes). The
  original solves ran on the live downloaded bytes and were fine; the damage only matters if you
  re-run the old script on the saved copies. Re-download, or repair in memory and check the SHA-256
  against the Game Master's value.
- The Mastery journal printed `None from None` for each solver run. That was a logging-template
  bug (the template read different keys than the script wrote). The real per-APK output is in
  `mastery/delivery-evidence.json`. See the day overview for tooling mistakes of this kind.

## Takeaways

- For this lane's `puzzle.v1` family, go straight to `assets/huntress_puzzle.json`, read the `codec`
  field, and decode. The activity code only forwards the value (as `proof=` beside a decoy), and the
  prompt text describes the theme, not the mechanism.
- List the DEX's API references before believing the prompt's vulnerability name; a missing API is
  quick evidence that the theme is cosmetic.
- Read codec and key from each delivery, and pair Mastery questions to APKs by the package name in
  the question, not by order.

## Evidence

Leaf base path: `challenges/day04/mobile-android/mobile-android-day-04-pending-intent-relay/`

- `single-solve/journal.md`, `single-solve/artifacts/solve_pending_intent_relay.py`
- `time-trial/journal.md`, `time-trial/solver-output.json`, `time-trial/submission-response.json`,
  `time-trial/ranked-1/result.json`
- `mastery/journal.md`, `mastery/delivery-evidence.json`, `mastery/solver-output-1.json` .. `-3.json`,
  `mastery/submission-response.json`, `mastery/ranked-1/result.json`
- Later solver: `tools/ranked/solvers/mobile-android-day-04-pending-intent-relay/solve.py`
- Answers: Single Solve `ANS-60A1CA8E89978462`; Time Trial attempt `ANS-FC8EF157E3911AE7`
  (values are per attempt and worthless now).
