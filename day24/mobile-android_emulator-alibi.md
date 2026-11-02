# Emulator Alibi

| | |
|---|---|
| Day / Lane | 24 / Mobile Android |
| Challenge ID | `mobile-android-day-24-emulator-alibi` |
| Technique | static asset decode (`split` codec) for Single Solve; running the APK's own `Release.code()` bytecode in a Dalvik interpreter for Time Trial / Mastery |
| Single Solve | Independent |
| Time Trial | Independent |
| Mastery | Independent |

## Summary

The prompt says to "defeat app-owned environment checks on the same pinned emulator". No emulator
or environment check had to be defeated. The Single Solve APK carried its answer in a bundled
puzzle asset under a fixed codec, and the Time Trial and Mastery APKs computed it in ordinary
arithmetic bytecode that never calls a framework API. The Single Solve went CORRECT on the first
submission. Time Trial and Mastery needed the library's Dalvik interpreter. A ranked Mastery
bundle then included a second-stage `payload.dex` loaded through `InMemoryDexClassLoader`, which
the interpreter did not yet model. That attempt failed and expired, and we extended the interpreter.

## The challenge

Prompt: "Defeat app-owned environment checks on the same pinned emulator used by the solve
oracle." Each question named a case (`MOB-24-...`) and a package, and asked for "the exact backend
answer code returned for this case". The answer format was a pattern, `ANS-` plus exactly 16
uppercase hex characters, with `exact` normalization.

- **Single Solve:** one APK (`emulator_alibi.apk`), one answer id `answer`, no deadline.
- **Time Trial:** one APK, one answer id. Regular attempts were untimed. Ranked attempts had a
  50 s window.
- **Mastery:** 8 APKs, answer ids `set1.answer`..`set8.answer`. The bundles were earlier-day
  builds in ascending day order, with Day 24's own APK as set 8. Ranked windows were 260-290 s.

## Solution

### Single Solve (v1, `split` codec)

1. **Triage.** The APK's theme class (`EmulatorAlibi`) had static fields `CODEC=split`,
   `CONTROL=emulator_detection` and `DAY=24`. It had no `Release` class and no answer-shaped
   strings in the DEX. This matches the lane's known layout: the answer lives in
   `assets/huntress_puzzle.json`, and `classes.dex` only forwards a decoy `client_secret`.
2. **Decode.** For the `split` codec, `parts[i]` is placed in output slot `order[i]`. The library's
   `asset_codecs.py` did this directly, and the result fit the `ANS-[0-9A-F]{16}` pattern with no
   padding or ordering ambiguity: `ANS-70755D5A67781B05`.
3. **Submit.** The Single Solve was CORRECT on call 1, about 65 s after the attempt started.

### Time Trial and Mastery (v2, `analysis` codec)

The Time Trial APK was a different schema (`huntress.mobile.puzzle.v2`, codec `analysis`). It
stores no answer data. `ChallengeActivity` calls `Release.code()`, compares the supplied answer
with `equalsIgnoreCase`, and also calls `Release.proof()`, a decoy that hashes the same token
without the final fold. The real answer is built as follows.

`code()` calls `token()`, then a few themed fold helpers (`EmulatorAlibi.emulatorScore`,
`BoardCheck.finBoardXX`, `Util.scanEmu4a`, `SensorMap.mixSensorXX`, `Probe.s1/s3`), then `Mix.f1`.
It folds the result, XORs a 64-bit constant, and emits 16 nibbles MSB-first through
`"0123456789ABCDEF".charAt`, prefixed `ANS-`. That is 10-11 calls in the Time Trial build. The
`seal*` helpers (`sealEmu`, `sealQemu`, `sealSensor`) and `proof()` were never executed. No helper
reads `Build.*` or any sensor, so the emulator theme is naming only.

Rather than hand-port each build, the solver runs the APK's own `Release.code()` in the library's
Dalvik interpreter (`dalvik_mini_vm.py`, `run_code(apk)`) and falls back to the `asset_codecs`
path for v1 builds. Time Trial and Mastery then each took one graded call.

## Time Trial and Mastery

Mastery was eight APKs, all v2. Within one mode the fixtures differed from attempt to attempt:

| Attempt | Day mix (set 1..8) |
|---|---|
| Regular rehearsal | 3, 6, 9, 10, 15, 17, 23, 24 |
| Ranked 1 and 2 (same bundle) | 2, 6, 8, 11, 14, 18, 20, 24 |
| Untimed retry after the failures | 3, 4, 7, 10, 15, 17, 22, 24 |

The regular Mastery and the Time Trial rehearsal were CORRECT in one call each. A ranked Time
Trial attempt on release day was also CORRECT in one call (3.65 s of a 50 s window).

The ranked Mastery bundle was different. Set 7 was a Day 20 "Loaded Later" build rebuilt as v2, and
it shipped `assets/payload.dex`. Its `Release.code()` reads that asset through the app's own
`Assets.read` helper and wraps it in `new InMemoryDexClassLoader(...)`. It takes the class name
from the text after `:` in `LoadedLater.moduleContract()`, calls `loadClass`, and gets
`getDeclaredMethod("share", long)`. Finally it calls `Method.invoke(null, Long.valueOf(x))`. The
payload holds one class with one method of pure `long` arithmetic and no answer string, so a grep
of it finds nothing.

The interpreter raised `unsupported external invoke ...getAssets()` on set 7 in both ranked
attempts, so no submission was made and the Mastery expired. We then extended the interpreter to
model exactly that hop:

- `Assets.read` is served from the APK's real asset, but only after checking that the app's own
  reader makes nothing but getAssets/open/read/close/copyOf calls.
- `InMemoryDexClassLoader` builds a nested interpreter over the payload bytes.
- `loadClass`, `getDeclaredMethod` and `Method.invoke` (static targets only) resolve names inside
  that DEX.

Anything outside the allowlist still raises. A fresh untimed `retry` returned a different bundle
(table above, with no payload) and was CORRECT on call 1.

A week-later ranked Mastery (2026-10-31) drew the same day mix as the failed ranked bundle. It
went CORRECT in one call, in 50.5 s of a 260 s window, with set 7 now solved. The journals do not
say whether that bundle's set 7 carried `payload.dex`, so we cannot say it exercised the new path.
A ranked Time Trial the same day was also CORRECT in one call (6.4 s of 50 s).

## Dead ends and gotchas

- **The title and prompt describe the cover story, not the work.** Nothing here emulates a device,
  and the only emulator-detection logic is naming.
- **Single Solve and Time Trial/Mastery use different schemas.** The Single Solve is v1 `split`.
  Time Trial and all 8 Mastery sets are v2 `analysis`. A solver that only decodes the asset exits
  2 on the Time Trial.
- **Ranked bundles can contain builds a regular attempt never showed.** The regular rehearsal and
  untimed retry had no `payload.dex`; only ranked set 7 did. A regular-attempt CORRECT did not
  prove the ranked fixtures would decode, and the failure cost the first timed Mastery.
- **Do not count a differently-fixtured CORRECT as proof of the new code path.** The CORRECT
  retry contained no hop. The extension was checked by an offline regression and a negative test
  (emptying the allowlist makes the archived ranked set 7 raise again). It was later confirmed on
  a Day 25 ranked bundle, where one set contained the hop and was accepted.
- **The payload has no independent cross-check.** The hop was `observed` on Day 24 and only
  became validator-confirmed on Day 25.
- A wrapper calling the interpreter's CLI with the contract's `--artifact` flag failed first
  because the CLI lacked it (see the day overview).

## Takeaways

- Technique names to search for: **v2 `analysis` codec / execute the APK's own bytecode** and
  **secondary-DEX loading via `InMemoryDexClassLoader`**. Run the app's `Release.code()` in an
  interpreter instead of hand-decoding constants. The decoy `proof()` and unexecuted `seal*`
  helpers are noise.
- Make an interpreter refuse what it cannot model (the allowlist) rather than guess. A loud
  failure on set 7 was a diagnosis, not a wrong answer.
- Before a timed Mastery, run the solver over every delivery and check for new API use
  (`assets/payload.dex`, class loaders). Regular, ranked and retry bundles are different draws.

## Evidence

Leaf: `challenges/day24/mobile-android/mobile-android-day-24-emulator-alibi/`

- `single-solve/journal.md` (includes the post-solve library-promotion notes for all three modes),
  `single-solve/start_response.json`, `single-solve/accepted_plan.json`
- `single-solve/artifacts/cand-adapter/solve.py` (VM first, `asset_codecs` fallback)
- `time-trial/regular/result.json`, `time-trial/ranked-1/result.json`,
  `time-trial/ranked-2/result.json`
- `mastery/regular/result.json`, `mastery/ranked-3/result.json`, `mastery/reattempt-1/answers.json`
  (the untimed retry has no `result.json`; its CORRECT is recorded in `progress.md`),
  `mastery/ranked-1/start_response.json` (the failed ranked bundle; no result)
- `challenges/day24/_run/lane-mobile-android-day-24-emulator-alibi.log`
- `library/mobile-android/dalvik-mini-vm/README.md` (MOBILE-ANDROID-DALVIKVM-001: "Secondary DEX
  loading" and "Day 24 build anatomy"), `library/mobile-android/asset-codecs/README.md`
  (MOBILE-ANDROID-CODECS-001)
- `progress.md` (grep `^| 24 .*emulator-alibi`)
