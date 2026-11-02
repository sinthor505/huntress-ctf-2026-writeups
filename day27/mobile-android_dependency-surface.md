# Dependency Surface

| | |
|---|---|
| Day / Lane | 27 / Mobile Android |
| Challenge ID | `mobile-android-day-27-dependency-surface` |
| Technique | plain asset decode (v1 `base64` codec); the "dependency" theme was dead metadata |
| Single Solve | Team solve (not ours) |
| Time Trial | Independent |
| Mastery | Independent |

## Summary

"Dependency Surface" shipped an APK whose manifest talked about a "reachable challenge-owned
dependency", which invited a search for a flaw hidden in a bundled library. There was none. In every
mode the answer was the base64 decode of `assets/huntress_puzzle.json`, the same skeleton as the
previous few days. A teammate scored the Single Solve within seconds of unlock, so our submissions
there returned `already_solved` and nothing was graded for us. Time Trial and Mastery were solved
independently on fresh `retry` attempts, one submission call each.

## The challenge

Prompt: "Prove which generated dependency flaw is reachable from an exported mobile surface." Each
question named a case (`MOB-27-...`) and a package (`com.huntress.mobile.d27.v...`) and asked for
"the exact backend answer code returned for this case". There was one answer id per delivery and
the answer format was a `pattern`: `ANS-` plus 16 uppercase hex characters, `exact` normalization.

- **Single Solve:** one APK (`dependency_surface.apk`), answer id `answer`.
- **Time Trial:** one APK, answer id `answer`; the timed attempts had a 50 s window.
- **Mastery:** 11 deliveries (answer ids `set1.answer` .. `set11.answer`), ten `.apk` and one `.apks`
  split bundle, under one timer (350 s on the ranked attempt). Each question's case id and package
  carried an earlier day's number (`MOB-02-...`, `MOB-04-...`, ... `MOB-27-...`), so the bundle was a
  set of rebuilds of earlier-day challenges plus the Day 27 build.

## Solution

1. **Triage the APK.** The Single Solve build had the lane's usual skeleton: manifest meta-data
   `huntress.mechanic = reachable challenge-owned dependency`, theme statics `CODEC = base64` and
   `CONTROL = legacy_dependency`, and an exported `ChallengeActivity` with the deep link
   `huntress27://dependency-surface/unlock`. That activity forwards the intent's `answer` as
   `proof=` beside a decoy `client_secret` (`SECRET-...`), which the triage tool flags as a
   secret-like string in use.
2. **Check for code to run.** Earlier v2 days (`codec: "analysis"`) computed the answer on-device in
   a `Release.code()` method, so we looked for a `Release` class. This build had none: no second
   DEX, no nested jar or archive, nothing that could be a "dependency".
3. **Decode the asset.** `assets/huntress_puzzle.json` is a `huntress.mobile.puzzle.v1` document
   with `codec: base64` and the asset `day` field set to 27. The answer is its plain base64 decode;
   the decoded string fits `ANS-[0-9A-F]{16}` and no other string in the APK does. The decoy
   `client_secret` is not the answer, as on earlier days.
4. **Submit.** Our offline decode for the Single Solve was `ANS-802A91C1FCCB2F7E`. It was never
   graded for us (see below).

The whole solve is one library call, `asset_codecs.resolve_start` (start response in, answers out,
about 0.2 s), which pairs each question with its delivery, repairs CRLF-damaged downloads and
handles `.apks` splits.

## Time Trial and Mastery

Both were solved from fresh `retry` attempts, because the `start` for each came back as the team's
already-`CORRECT` readout with no `artifacts[]`, so there was nothing to solve in it.

- **Time Trial** was the same v1 `base64` skeleton as the Single Solve. CORRECT on the first call in
  the untimed attempt and in two timed (ranked) attempts; the ranked ones took about 3.5 s locally
  end to end.
- **Mastery** delivered 11 sets, all v1, with no `Release` class anywhere. The asset day per set
  followed the case ids: the untimed bundle was days 2, 3, 6, 9, 12, 15, 16, 19, 23, 26, 27 and the
  ranked bundle 2, 4, 6, 8, 12, 14, 16, 20, 21, 26, 27. The codecs seen across them were reverse,
  base64, hex, xor, caesar, native, reflection, dynamic and split, so the generic decoder had to
  cover every earlier-day codec, not only Day 27's. Set 10 was the Day 26 `.apks` split shape, the
  ranked bundle's set 8 carried a Day 20 `assets/payload.dex`, and two sets carried a native `.so`.
  Both bundles went CORRECT on the first call (11 answers in one submit); the ranked one in about
  6.7 s locally.

## Dead ends and gotchas

- **The theme text was a red herring.** "Reachable challenge-owned dependency" suggested that the
  answer would live in a bundled library. Before the Time Trial and Mastery arrived, we built a
  fallback that runs `Release.code()` from `classesN.dex`, an asset DEX or a bundled jar. It was
  never needed: every build we saw had no dependency at all. We tested it only on synthetic
  repackagings of an earlier day's APK.
- **A team-solved `start` has no artifact.** Time Trial and Mastery `start` returned the team's
  `CORRECT` readout without `artifacts[]`; the solvers failed on it until we issued a `retry` for a
  fresh attempt.
- **`already_solved` is not a rejection.** The four Single Solve submissions returned
  `already_solved` and graded nothing; our tooling logged them as rejected and ran fix rounds, which
  was a mistake in our automation (see the Day 27 overview).

## Takeaways

- Technique: **v1 asset decode**. The answer lives in `assets/huntress_puzzle.json`; `classes.dex`
  only forwards it beside a decoy secret.
- Before reaching for a Dalvik interpreter, check whether the build has a `Release` class (or any
  code that computes the value). A theme string in the manifest is not evidence of a mechanism.
- In a Mastery bundle, classify each set by its own asset (`codec`, `day`) and its container
  (`.apk` vs `.apks`), not by the day of the challenge.

## Evidence

Leaf base path: `challenges/day27/mobile-android/mobile-android-day-27-dependency-surface/`

- `single-solve/journal.md`, `single-solve/start_response.json`, `single-solve/team_solved.json`
  (team-scored; our submissions were `already_solved`)
- `single-solve/artifacts/cand-adapter/solve.py`, `single-solve/artifacts/cand-adapter/out.json`
  (offline Single Solve value, ungraded)
- `time-trial/regular/result.json`, `time-trial/ranked-1/result.json`, `time-trial/ranked-2/result.json`,
  `time-trial/*/accepted_plan.json`
- `mastery/regular/result.json`, `mastery/ranked-1/result.json`, `mastery/*/accepted_plan.json`
- `challenges/day27/_run/librarian/mobile-android-day-27-dependency-surface.md`
- `challenges/day27/_run/lane-mobile-android-day-27-dependency-surface.log`
- Library: MOBILE-ANDROID-CODECS-001 (asset codecs), MOBILE-ANDROID-DALVIKVM-001 (Day 27 section)
- `progress.md` (grep `^| 27 .*dependency-surface`)
