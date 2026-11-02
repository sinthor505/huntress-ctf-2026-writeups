# Exported Trust

| | |
|---|---|
| Day / Lane | 1 / Mobile Android |
| Challenge ID | `mobile-android-day-01-exported-trust` |
| Technique | Static APK triage: exported deep-link Activity, puzzle asset holding the evidence code in clear |
| Single Solve | Solved, graded result not preserved (local mark only) |
| Time Trial | Independent (ranked attempt, CORRECT on call 1) |
| Mastery | Independent (ranked attempt, CORRECT on call 1, 3 sets) |

## Summary

An APK exposes an exported, browsable Activity that builds a backend "evidence" URL out of a case reference, an
embedded client credential and a proof value. The credential is a decoy and the proof is the answer, which Day 1
stores as plain text in the app's puzzle asset. This is the lane's entry-level challenge. The Single Solve was
recorded by the player on 2026-10-18 without a saved journal or graded result, so the clearest evidence we hold is
the Time Trial and Mastery runs from the 2026-10-26 ranked sweep.

## The challenge

The prompt: identify an exported deep-link Activity that attaches an embedded client credential, invoke it with
the correct case reference, and return the backend evidence code. Each question names a case (`MOB-01-<10 hex>`)
and a package (`com.huntress.mobile.d01.v<hex>`) and asks for the exact backend evidence code. The answer ID was
`answer` in Single Solve and Time Trial, and `setN.answer` in Mastery.

Delivery was one small APK (the Single Solve APK is about 12.7 KB). Time Trial delivered one APK
(`exported_trust_time_trial.apk`) with a 50 s window. Mastery delivered three APKs
(`exported_trust_mastery_NN.apk`) with a 110 s window and three answers. Both timed modes were on the
"legacy" route, where a plain `retry` was already a ranked, Hall-of-Fame-eligible attempt.

## Solution

The leaf has no journal, so the steps below come from the lane's consolidated notes on the Day 1 build and from the
saved results, not from a first-hand log of the original solve.

1. **Recon.** The app is a two-component skeleton that every later mobile day reuses: a launcher `MainActivity`
   showing the case reference, and an exported `ChallengeActivity` with a browsable `huntress01://<slug>/unlock`
   pathPrefix intent filter. The manifest carries `huntress.day` / `huntress.mechanic` metadata ("exported Activity
   and caller trust"), and the Activity has no caller check at all, so any app or `adb` intent can reach it.
2. **Read the Activity.** `onCreate` reads `answer` / `case_id` extras, lets URI query parameters overwrite them,
   URL-encodes the pair and appends a `client_secret=SECRET-...` constant, then calls `WebView.loadUrl` on a
   loopback backend URL of the form `http://127.0.0.1:19090/mobile/attempt/d01-<hex>/evidence?...&proof=<answer>`.
   The backend is not reachable from the analysis host, so the code path is for understanding, not for running.
3. **Find the value.** The `SECRET-` constant is active (it is sent in the URL) but is a decoy, not the answer. The
   evidence code that gets sent as `proof=` lives in `assets/huntress_puzzle.json`. Day 1 is the only day whose
   asset codec is `plain`: the `data` field is the answer verbatim, in `ANS-<16 hex>` form. No answer-shaped string
   exists in `classes.dex`.
4. **Submit** the string exactly as stored, as a JSON string, under the question's answer ID.

The Single Solve answer value was not preserved in the leaf, so none is given here.

## Time Trial and Mastery

- **Same shape, new values.** The package suffix, case ID, endpoint slug, decoy secret and asset data change per
  build, but the manifest, components and code are identical across modes, so a mapper that works on one APK works
  on all of them.
- **Time Trial:** one question, one answer (`ANS-3630D83F6C96844C` in the ranked attempt). The APK was downloaded,
  decoded and submitted in a scripted pass: 0.53 s download, effectively 0 s solve, 1.36 s submit, 2.78 s total
  of a 50 s window.
- **Mastery:** three sets, answer IDs `set1.answer`..`set3.answer`, one per delivered APK. The decoded values were
  `ANS-9439C2870449265B`, `ANS-D542D77BC0FC502B` and `ANS-AED9CF6A43118847`, accepted in one submission, 3.36 s
  total of a 110 s window.
- **Which solver.** The ranked passes reused the Day 2 solver, which already read the Day 1 `plain` family. The
  windows were short enough that the solve had to be scripted: pair each answer ID with its APK by delivery,
  read the asset, check the format, submit.
- The ranked Time Trial and Mastery attempts are the only graded results saved for this challenge; both returned
  CORRECT with a single submission call (`calls: 1`).

## Dead ends and gotchas

- **Decoy credential.** The `SECRET-...` string is the one the Activity visibly attaches, and it looks like the
  "embedded client credential" from the prompt. It is not the answer.
- **Saved-copy damage.** The saved Day 1 Single Solve APK was one of the copies corrupted by LF to CRLF
  translation in the early days (library note: Days 1-5 saved ZIPs/APKs). Day 1 has no recorded hash to verify a
  repair against, so the library tools only accept a repair when every member passes its CRC. Re-download rather
  than trust an old copy.
- **Mastery pairing.** Pair each set's answer with its APK by SHA-256 / delivery name, not by question order.
- **`plain` is special.** Later days encode the asset (reverse, Base64, XOR, split and so on), so a Day 1 habit of
  reading `data` directly stops working from Day 2.

## Takeaways

- For this lane, read the asset before the bytecode: the manifest and Activity explain the mechanic, but the value
  is in `assets/huntress_puzzle.json`, and the code only forwards it as `proof=` next to a decoy secret.
- Static triage is enough. The "exported trust" flaw (an exported, browsable Activity with no caller check that
  attaches a secret and loads a URL in a WebView) is real, but the backend is unreachable, so nothing is gained by
  running the app.
- With 50 to 110 s windows, script download, decode and submit as one pass.

## Evidence

Leaf base path: `challenges/day01/mobile-android/mobile-android-day-01-exported-trust/`

- `single-solve/exported_trust.apk` is the only Single Solve file. There is no journal, start response or result
  for it; `progress.md` rows for all three modes read "Player-directed local mark", dated 2026-10-18.
- `time-trial/ranked-1/{start_response,result,accepted_plan}.json`, `pass.log` (CORRECT, `calls: 1`).
- `mastery/ranked-1/{start_response,result,accepted_plan}.json`, `pass.log` (CORRECT, `calls: 1`, 3 answers).
- Day 1 build description: lane notes "Mobile Android: how the lane evolves" (day table and invariants) and the
  asset-codecs codec reference (`plain`).
