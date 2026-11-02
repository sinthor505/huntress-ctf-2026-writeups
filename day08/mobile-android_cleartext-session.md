# Cleartext Session

| | |
|---|---|
| Day / Lane | 8 / Mobile Android |
| Challenge ID | `mobile-android-day-08-cleartext-session` |
| Technique | Static APK asset decode (`reverse` text codec); the network story is a decoy |
| Single Solve | Independent (saved graded submission response, `CORRECT`; no `result.json`) |
| Time Trial | Independent (original untimed attempt `CORRECT` on submit 2, journal entry only; the ranked-1 `result.json` is the saved graded result, `CORRECT` on call 1) |
| Mastery | Independent (original untimed attempt `CORRECT` on call 1, journal entry only; the ranked-1 `result.json` is the saved graded result, `CORRECT` on call 1) |

## Summary

The prompt describes recovering a session from cleartext traffic and replaying it against a backend,
but the backend is never reachable and never needed. The answer is stored in the APK's
`assets/huntress_puzzle.json` as a reversed string; reversing it yields the `ANS-` code. The same
one-line decode solved Single Solve, Time Trial and the five-APK Mastery.

## The challenge

Prompt: "Recover a synthetic session from allowed cleartext traffic and replay it against the backend."
Each question names a case ID (`MOB-08-...`) and a package (`com.huntress.mobile.d08.v...`) and says to
submit "only the exact backend answer code returned for this case". The answer format is a pattern:
`ANS-` followed by exactly 16 uppercase hex characters, `exact` normalization, one field `answer`.

- Single Solve: one APK (`cleartext_session.apk`), one answer, untimed.
- Time Trial: one APK, one answer. The untimed attempt had `deadline: null`; the ranked attempt later
  had a 50 s window.
- Mastery: five APK deliveries, answer IDs `set1.answer` to `set5.answer`, one APK per set. The
  untimed attempt had no deadline; the ranked attempt had a 170 s window.

## Solution

1. **Recon the APK as data.** `aapt dump badging` and `xmltree` showed the INTERNET permission,
   `usesCleartextTraffic=true`, `allowBackup=false`, `debuggable=false`, and an exported
   `ChallengeActivity` with a browsable deep link `huntress08://cleartext-session/unlock`. All of this
   matches the cleartext-session theme.
2. **Read the DEX strings.** `strings -a classes.dex` shows a loopback evidence URL on
   `127.0.0.1:19090` with `case_id=`, `client_secret=` and `proof=` parameters, and calls to
   `getStringExtra`, `getQueryParameter`, `Uri.encode` and `WebView.loadUrl`. That is the intended-looking
   attack path: feed a proof through the deep link and let the app issue the cleartext request. The
   endpoint is Android-local, so a request from the host could not connect. The app was never installed
   or run.
3. **Look at the asset instead.** Every puzzle APK in this lane carries `assets/huntress_puzzle.json`.
   Here it was:

   ```json
   {"schema":"huntress.mobile.puzzle.v1","day":8,"codec":"reverse","data":"6667D24E8BC6DE39-SNA"}
   ```

4. **Decode.** `reverse` means the string reversed by Unicode code point:

   ```python
   data[::-1]   # "6667D24E8BC6DE39-SNA" -> "ANS-93ED6CB8E42D7666"
   ```

   The result is `ANS-` plus 16 uppercase hex characters, so it fits the answer pattern. Submitted once
   as `answer`, it returned `CORRECT`. Single Solve answer: `ANS-93ED6CB8E42D7666`.

We never observed the backend replay, so we could not confirm that the reversed value is literally what
the endpoint would have returned. The evidence is the format match and the graded result.

## Time Trial and Mastery

Nothing structural changed. A fresh APK (new case ID, package, secret) carried the same `reverse` codec,
and the decoder reads the codec name from each APK's own asset rather than assuming it.

- **Time Trial.** First submit returned HTTP 503 `temporarily_unavailable`, which is infrastructure and
  not a verdict. `status` still showed the same attempt, and resubmitting the identical answer returned
  `CORRECT` (2 ledger entries, 1 graded). A later ranked attempt (50 s window) was solved in 1 call in
  about 1.4 s total.
- **Mastery.** Five APKs, each decoded independently with its SHA-256 checked against the start
  response. All five decoded to the `ANS-` pattern and went in one call: `CORRECT`. `start` itself
  returned 503 three times before succeeding (nothing was consumed). A later ranked attempt (170 s
  window) was solved in 1 call in about 2.3 s total.

The ranked reruns used the per-challenge solver in `tools/ranked/solvers/`. It pairs each question to an
APK by package name from the manifest first, then by API hash and filename, so Mastery order does not
matter.

## Dead ends and gotchas

- **The decoy path.** The cleartext transport, deep link, `client_secret` and loopback endpoint are all
  present and plausible, but the answer never touches them. In the DEX the secret sits beside a
  `proof=` parameter, and the two are distinct request fields, so the secret is not the answer. The
  backend cannot be reached from outside the device anyway.
- **Tool gap on the first day.** The library's APK asset inspector reported `reverse` as unsupported at
  Single Solve time, so the value was read by hand. `reverse` has since been added as a named codec.
- **Transient 503s are not verdicts.** Both the Time Trial submit and the Mastery start hit them. Re-check
  `status`, then resubmit the same answer unchanged.

## Takeaways

- For `huntress.mobile.puzzle.v1` APKs, read `assets/huntress_puzzle.json` first: the codec name (`reverse`
  here, also used on Days 2 and 21) and the pattern in the answer format tell you what to compute. Theme
  text in the prompt and manifest is flavor.
- Validate any decoded value against the stated answer pattern; for a plain reversal that is nearly proof.
- Faster next time: skip the manifest and DEX string pass for this lane and go straight to the asset, then
  confirm the codec and pattern.

## Evidence

Leaf base: `challenges/day08/mobile-android/mobile-android-day-08-cleartext-session/`

- `single-solve/journal.md`, `single-solve/start-response.json`, `single-solve/submission-response.json`
  (`CORRECT`)
- `time-trial/journal.md`, `time-trial/start_response.json`, `time-trial/artifacts/answer_map.json`,
  `time-trial/ranked-1/result.json` (`CORRECT`, 1 call)
- `mastery/journal.md`, `mastery/start_response.json`, `mastery/artifacts/answer_map.json`,
  `mastery/ranked-1/result.json` (`CORRECT`, 1 call, 5 answers)
- Solver for the ranked reruns: `tools/ranked/solvers/mobile-android-day-08-cleartext-session/solve.py`
- Library: `asset_codecs.py` `reverse` codec (MOBILE-ANDROID-CODECS-001); `progress.md` rows for Day 8
