# Layered Protections

| | |
|---|---|
| Day / Lane | 30 / Mobile Android |
| Challenge ID | `mobile-android-day-30-layered-protections` |
| Technique | recover a build-time answer from an APK: a new v1 asset codec (Single Solve), then running the APK's own `Release.code()` in a Dalvik interpreter (Time Trial, Mastery) |
| Single Solve | Team solve (not ours) |
| Time Trial | Independent |
| Mastery | Independent |

## Summary

"Layered Protections" is the Mobile Android lane's Day 30 challenge. Each delivery is an APK that
hides one `ANS-<16 hex>` code. The Single Solve used a new asset codec (`layered`) that we decoded
offline, but a teammate had already scored that mode seconds after unlock, so our one submission
returned `already_solved` and was never graded for us. The Time Trial (one APK) and the Mastery
(14 APKs) were both computed by running each APK's own `Release.code()` bytecode in the library's
Dalvik interpreter, and both went CORRECT on the first submission.

## The challenge

Prompt: "Unwind obfuscation, native indirection, runtime decryption, and integrity checks while
proving the same backend effect." Each question names a case id and package (for example
`MOB-30-... in package com.huntress.mobile.d30...`) and asks for "the exact backend answer code
returned for this case". Answer format: pattern `ANS-` plus exactly 16 uppercase hex characters,
`exact` normalization, one string per question.

- **Single Solve:** one APK, one answer id (`answer`).
- **Time Trial:** one APK (`layered_protections_time_trial.apk`), one answer id. A fresh APK per
  attempt; the ranked reruns had a 50 s window.
- **Mastery:** 14 APKs, answer ids `set1.answer` .. `set14.answer`. The sets are rebuilds of earlier
  days' challenges (cases tagged `MOB-02`, `04`, `06`, `07`, `10`, `12`, `15`, `17`, `19`, `21`,
  `24`, `25`, `28`, `30`), not 14 copies of the Day 30 build. One delivery in the ranked bundle was
  an `.apks` split bundle rather than a plain `.apk`.

## Solution

1. **Triage the APK.** The standing lane fact is that the answer never comes from the Java code that
   talks to the backend: `ChallengeActivity` only forwards a value as `proof=` next to a decoy
   `client_secret`. The real value is in the packaged puzzle asset, and which of two schemas the asset
   uses decides the method.
2. **Single Solve: a new v1 codec, `layered`.** The asset stored an integer `key` and Base64 `data`,
   with the theme constant naming the codec. We could not find the layering in the app code, so the
   decoder was inferred from the known `ANS-` prefix: a brute search over "reverse x xor/add/sub x
   offset/step" found exactly one family that produced a pattern-fitting string. That family is
   Base64-decode, `p[j] = data[j] ^ ((key + j) & 255)`, then reverse the whole result. The library
   decoder does the default reading first and, if a later build changes the layering, searches the
   other combinations and accepts a result only if exactly one output fits the answer pattern:

   ```python
   first = run("x", 1, "post")          # xor, step 1, reverse after
   if first is not None and pat.fullmatch(first):
       return first
   fits = set()
   for op in "xsa":                     # xor / sub / add
       for step in range(256):
           for rev in ("no", "pre", "post"):
               t = run(op, step, rev)
               if t is not None and pat.fullmatch(t):
                   fits.add(t)
   if len(fits) == 1:
       return fits.pop()
   ```

   The decoded value was `ANS-3F023201F46243CA`. Submitting it returned `already_solved` (see
   provenance below), so this decoder was never validated by a grade of our own.
3. **Time Trial and Mastery: v2 `analysis`.** Here the asset carried no stored answer data at all. The
   value is computed on the device by `Release.code()`, with sibling "protection" methods
   (`LayeredProtections.layeredGate`, `LayerKit`, `PeelStack`, `seal*`, `Release.proof`) that exist as
   decoys and never run on the real path. Rather than hand-decode the constants, the solver ran the
   APK's own `Release.code()` in the library's Dalvik mini VM. The solver wrapper routes by asset
   schema (v1 to the asset codecs, v2 to the VM) and checks every result against the answer pattern
   before it writes the answer map.
4. **Submit.** The Time Trial map had one field; the Mastery map was assembled per set from the
   delivery that matched each question's `artifact` field, then submitted as one 14-field call.

## Time Trial and Mastery

Both modes were answered on the first call:

| Attempt | Result | Calls | Local total |
|---|---|---|---|
| Time Trial, untimed regular | CORRECT | 1 | 3.9 s |
| Mastery, untimed regular (14 sets) | CORRECT | 1 | 5.9 s |
| Time Trial, ranked | CORRECT | 1 | 4.6 s in a 50 s window |
| Mastery, ranked (14 sets) | CORRECT | 1 | 8.5 s in a 440 s window |
| Time Trial, second ranked | CORRECT | 1 | 3.3 s in a 50 s window |

The regular attempts came first, on 2026-10-30, and are the ones that count as our own solves here;
ranked reruns followed on 2026-10-31 and 2026-11-01 (ranked attempts were bragging rights only on
Day 30, not the prize ranking).

What had to generalize: the Time Trial and Mastery were v2 on every APK, including the Mastery's
`libarena.so` build (set 7, a native hop) and the `.apks` delivery, which the existing loader and
VM already handled. No library change was needed for Day 30; the VM output equalled every accepted
field when re-run offline afterwards (15/15 for the Time Trial plus 14 Mastery APKs).

## Dead ends and gotchas

- **The Single Solve was un-gradable for us.** Our recon ran at 13:00:06 UTC on 2026-10-30, a
  teammate's solve is stamped 13:07:00, and our first submission at 13:09:24 returned
  `already_solved`. With a team-scored Single Solve a fresh attempt is not available, so the `layered`
  decode stays `observed`: it fits the answer pattern and equals the value we submitted, but nothing
  confirmed it.
- **The Single Solve's codec did not predict the other modes.** The Single Solve was v1 `layered`;
  the Time Trial and all 14 Mastery sets were v2 `analysis`, with no v1 set in the Mastery at all.
  A solver that handled only the new Single Solve codec would have solved none of the graded modes.
- **The solver's header docstring was a leftover copy from Day 28.** The logic is generic (route by
  schema), so this was harmless, but the header does not describe the Day 30 build.
- **Prefix-guided codec recovery is only as good as the prefix.** It works because the answer pattern
  is `ANS-` plus 16 uppercase hex characters; the decoder only trusts a result when exactly one
  alternative layering fits.

## Takeaways

- Route on the asset's schema, not on the day. The pattern across Days 28-30 was a new index-dependent
  v1 codec in the Single Solve (`packet`, `table`, `layered`), but the Time Trial and Mastery were v2,
  where running the app's own `Release.code()` in a Dalvik interpreter beats reimplementing a codec.
  Search the technique as "Dalvik interpreter for `Release.code()`".
- For an unknown stored-data codec, brute the small family of byte operations against a known answer
  prefix and require a unique fit; do not accept the first plausible decode.
- Mastery bundles are rebuilds of many earlier days, so a mixed 14-set bundle is the normal case;
  resolve each set from the delivery named by the question's `artifact` field.
- Decoys are deliberate: `seal*` helpers and `Release.proof` never run on the path that computes the
  real answer, so running the bytecode avoids them by construction.

## Evidence

Base path: `challenges/day30/mobile-android/mobile-android-day-30-layered-protections/`

- `single-solve/journal.md`, `single-solve/team_solved.json` (team-scored, `already_solved`),
  `single-solve/artifacts/cand-solo/solve.py` (the wrapper), `single-solve/artifacts/cand-solo/out.json`
- `time-trial/regular/result.json`, `time-trial/ranked-1/result.json`, `time-trial/ranked-2/result.json`
  and the matching `accepted_plan.json` files
- `mastery/regular/result.json`, `mastery/ranked-1/result.json` and the matching `accepted_plan.json`
  files; `mastery/regular/start_response.json` (14 questions, `artifact` per set)
- `challenges/day30/_run/librarian/mobile-android-day-30-layered-protections.md`
- `challenges/day30/_run/lane-mobile-android-day-30-layered-protections.log`
- Library entries MOBILE-ANDROID-CODECS-001 (the `layered` codec, marked observed) and
  MOBILE-ANDROID-DALVIKVM-001 (Day 30 validation paragraph)
- `progress.md` (grep `^| 30 .*layered-protections`): Time Trial and Mastery rows; no Single Solve row
