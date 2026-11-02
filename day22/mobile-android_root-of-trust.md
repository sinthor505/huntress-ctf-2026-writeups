# Root of Trust

| | |
|---|---|
| Day / Lane | 22 / Mobile Android |
| Challenge ID | `mobile-android-day-22-root-of-trust` |
| Technique | synthetic root-detection theme; the answer is a base64-encoded asset, not a bypass |
| Single Solve | Independent |
| Time Trial | Independent |
| Mastery | Independent |

## Summary

The prompt asks you to "model and bypass multiple synthetic root signals without globally patching
success". In practice there was nothing to bypass: the backend answer code sat in
`assets/huntress_puzzle.json` as base64, and the theme code in the APK was either absent (Single
Solve) or dead code (Time Trial and Mastery). We decoded the asset with the lane's codec table and
submitted. All three modes went CORRECT on the first submission.

## The challenge

Prompt: "Model and bypass multiple synthetic root signals without globally patching success."
Each question reads "For case MOB-22-... in package com.huntress.mobile.d22....: ... Submit only
the exact backend answer code returned for this case."

- Single Solve: one APK (`root_of_trust.apk`), one answer id `answer`, format "`ANS-` followed by
  exactly 16 uppercase hex characters", `exact` normalization, untimed.
- Time Trial: one APK, same single answer, 50 s ranked window.
- Mastery: six APKs, answer ids `set1.answer` ... `set6.answer`, 200 s ranked window. Only one
  delivery per bundle was a Day 22 build; the rest were earlier-day builds (the case id `MOB-NN` and
  the package `...dNN...` in each question give the build's own day). Examples: one bundle held days
  1/7/12/13/17/22, a ranked one 1/6/12/13/17/22, another 1/7/12/16/21/22.

## Solution

1. **Recon.** Download the APK, check its SHA-256 against the start response, and open the ZIP.
   The fixtures in this lane are not real root-detection apps. The `ChallengeActivity` forwards a
   value as `proof=` beside a decoy `client_secret`, and `classes.dex` contains no usable `ANS-`
   answer. (This is the lane pattern recorded in our library as the mobile codec table; the Day 22
   journal confirms "no `ANS-` strings in DEX", answer lives in the asset, decoy `client_secret`
   ignored.)
2. **The key insight.** `assets/huntress_puzzle.json` carries a `codec` and `data`. For Day 22 the
   codec is `base64`, so the answer is `base64.b64decode(data)`. The decoded string matched the
   `ANS-[0-9A-F]{16}` pattern from the question, which is a cheap self-check.
3. **Extraction.** The solver is a short offline script that maps the asset's codec to a decoder
   (plain, base64, hex, reverse, caesar, xor, split, indexed, dynamic, native) and fails loudly if
   the result does not match the pattern:

   ```python
   if codec == "base64":
       return base64.b64decode(data).decode("utf-8")
   ...
   if not PAT.fullmatch(value):
       raise SystemExit("decoded value does not match ANS- format")
   ```

4. **Submission.** Single Solve answer: `ANS-725991794E5E611F`, accepted on the first call
   (CORRECT roughly one minute after unlock, per the lane log).

No specimen code was run; the whole solve is static.

## Time Trial and Mastery

Time Trial and Mastery APKs are richer than the Single Solve one: they add a theme method,
`rootSignals(ZZZ)I`, that Single Solve lacks. Our review (2026-10-24) found it has no caller
anywhere in the DEX, so it is dead code, and the asset value is still the answer; all 15 accepted
IDs we checked reproduced from the asset decode.

- **Time Trial:** same decode on a fresh APK. The regular (untimed) attempt and the ranked attempt
  were both CORRECT on call 1; the ranked one finished in 3.36 s locally inside a 50 s window.
- **Mastery:** six deliveries per attempt, so six decodes keyed by `setN.answer`. Because
  earlier-day builds are re-drawn per Mastery variant (Days 1, 6/7, 12, 13, 16/17/21 and 22
  appeared across attempts), the solver has to handle every asset codec the lane had introduced by
  then, not just base64. One submission carrying all six answers was CORRECT in the regular attempt
  and in ranked attempt 1 (3.96 s locally, 200 s window). The journal notes that regular and ranked
  Mastery differed in one set, so the question's own `MOB-NN` id has to be read per attempt rather
  than assumed.
- Both modes were later re-run in further ranked attempts (Time Trial: three more, one of them from
  a second machine; Mastery: two more); each was CORRECT on call 1. Those reruns were speed
  attempts and are not the prize timing.

## Dead ends and gotchas

- The prompt is pure theme. There is no root detection to model or patch; a reader who starts
  decompiling the (nonexistent or dead) root-signal code is on the wrong track. Triage the build for
  callers first: `rootSignals(ZZZ)I` is 3-18 code units of primitive-argument logic with no
  framework API and no caller.
- The DEX holds a decoy `client_secret` next to the real `proof=` value. The one self-contained
  solver variant emits an alternate-answer group only if a DEX contains other `ANS-` strings; none
  were needed.
- Single Solve's start response reported `deadline: null`; Time Trial and Mastery in the ranked
  route returned real server deadlines (50 s and 200 s here), so the solve had to be one scripted
  download-decode-submit pass.
- Mastery bundles contain other days' APKs, so a decoder written only for the release day's codec
  would fail on the older sets.

## Takeaways

Technique to search for: **asset-codec decode** in a synthetic-theme mobile lane: read
`assets/huntress_puzzle.json`, apply its declared `codec`, validate against the answer pattern, and
ignore the DEX `client_secret` decoy. Check for callers of any theme-named method before treating it
as the solution path. We would not do anything differently on this challenge; the offline decoder
solved all three modes without modification beyond covering the older codecs for Mastery.

## Evidence

Leaf base: `challenges/day22/mobile-android/mobile-android-day-22-root-of-trust/`

- `single-solve/journal.md`, `single-solve/start_response.json`, `single-solve/accepted_plan.json`
- `single-solve/artifacts/cand-rubric/solve.py` (self-contained asset decoder with the DEX decoy cross-check)
  and `single-solve/artifacts/cand-adapter/solve.py` (second decoder, same result)
- `time-trial/regular/result.json`, `time-trial/ranked-1/result.json` (also `ranked-2`, `ranked-3`, `ranked-vm-1`)
- `mastery/regular/result.json`, `mastery/ranked-1/result.json` (also `ranked-2`, `ranked-3`)
- `challenges/day22/_run/lane-mobile-android-day-22-root-of-trust.log` (submission calls, timings)
- `library/mobile-android/` codec table (MOBILE-ANDROID-CODECS-001) and lane playbook (MOBILE-PLAYBOOK-001)
- `progress.md` (grep `^| 22 .*root-of-trust`)
