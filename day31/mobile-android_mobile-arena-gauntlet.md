# Mobile Arena Gauntlet

| | |
|---|---|
| Day / Lane | 31 / Mobile Android |
| Challenge ID | `mobile-android-day-31-mobile-arena-gauntlet` |
| Technique | run each APK's own `Release.code()` bytecode in a Dalvik interpreter (`puzzle.v2`/`analysis` codec), across every mode |
| Single Solve | Team solve (not ours) |
| Time Trial | Independent |
| Mastery | Independent |

## Summary

"Mobile Arena Gauntlet" was the Mobile Android lane's capstone for the event's final 24-hour day:
a "mass-generated multi-app campaign that composes every released mobile primitive under one
aggregate clock." Every delivery across all three modes used the `puzzle.v2`/`analysis` asset
schema, meaning the answer was never stored in the APK at all — it had to be computed by running
the app's own obfuscated bytecode. We solved Time Trial and Mastery by running each APK's
`Release.code()` through the lane's Dalvik mini-VM interpreter, unchanged from the wrapper used on
Days 28-30, and both went CORRECT on the first submission. The Single Solve was scored by a
teammate before our own attempt landed, so that mode's value was never graded for us.

## The challenge

Prompt (identical across modes): "Complete a mass-generated multi-app campaign that composes
every released mobile primitive under one aggregate clock." Each question names a per-case id and
Android package (e.g. `MOB-31-FB7D4573CC` in `com.huntress.mobile.d31.v...`) and asks for "the
exact backend answer code returned for this case." Answer format: `ANS-` followed by exactly 16
uppercase hex characters, exact-string normalization.

- **Single Solve:** one 12.7 KB APK (`mobile_arena_gauntlet.apk`), one answer id (`answer`).
- **Time Trial:** one APK per attempt (`mobile_arena_gauntlet_time_trial.apk`), one answer id,
  untimed on the first (regular) pass and a 50 s server window on the ranked rerun.
- **Mastery:** 15 APKs (`mobile_arena_gauntlet_mastery_01.apk` .. `_15.apk`), answer ids
  `set1.answer` .. `set15.answer`. Each set is not a Day 31 clone: the package names and case ids
  show the sets are rebuilds of Days 1, 4, 6, 8, 10, 11, 15, 16, 19, 20, 23, 25, 27, 30 and 31 —
  one app pulled from (almost) every stage of the lane's run, true to the "gauntlet" framing.
  Windows were untimed on the regular pass and 470 s on the ranked rerun.

## Solution

1. **Triage every delivery.** The standing lane fact (true since early in the event) is that the
   Java activity that talks to the backend only forwards a decoy `client_secret=SECRET-...`
   alongside the real `proof=` value; the actual answer lives in the packaged puzzle asset or, for
   `analysis` builds, nowhere on disk at all. The Single Solve asset read `huntress.mobile.puzzle.v2`,
   `day: 31`, `codec: "analysis"`, theme class `MobileArenaGauntlet` (`CONTROL = gauntlet`), with no
   bundled native library and no secondary `payload.dex` — so this build stores no answer data; it
   has to be computed.
2. **Run the bytecode instead of reverse-engineering it.** The solver (a thin wrapper unchanged in
   logic since Day 28) routes each delivery by its asset schema: a `v1` stored codec goes to the
   library's `asset_codecs.resolve_start`, and `v2` `analysis` runs `Release.code()` inside the
   Dalvik mini-VM and takes the returned string as the answer, after checking it against the
   pattern:

   ```python
   def vm_value(path):
       out = VM.run_code(str(path))
       val = out["answer"]
       if not isinstance(val, str) or not ANS.fullmatch(val):
           raise RuntimeError("VM result fails the answer format: %r" % (val,))
       return val
   ```

   All three modes resolved through this path — every delivery was `analysis`, with no `v1` codec
   on the whole day.
3. **Single Solve.** The VM produced `ANS-271111025BA7746E` in under half a second (15 bytecode
   calls, 0 native hops, 0 secondary-DEX hops, clean exit). Our one submission returned
   `already_solved`: a teammate had already scored this exact Single Solve. The value is recorded
   as our own candidate but was never run through the validator for us.
4. **Time Trial.** A fresh `retry` issued a new APK and case per attempt. The regular (untimed)
   attempt's VM run produced `ANS-35A3DF9611885CE0`, submitted and accepted on the first call.
5. **Mastery.** The solver ran all 15 APKs independently (one VM instance per delivery, matched to
   its question by the `artifact` field), assembled one 15-field map, and submitted it in a single
   call — accepted outright, no fix rounds.

## Time Trial and Mastery

| Attempt | Result | Calls | Local total | Window |
|---|---|---|---|---|
| Time Trial, regular | CORRECT | 1 | 4.95 s | untimed |
| Time Trial, ranked | CORRECT | 1 | 3.62 s | 50 s |
| Mastery, regular (15 sets) | CORRECT | 1 | 5.23 s | untimed |
| Mastery, ranked (15 sets) | CORRECT | 1 | 8.18 s | 470 s |

Both regular attempts landed within about 100 seconds of the challenge unlocking (recon at
13:00:06 UTC, both CORRECT by 13:01:43); these are the attempts that count as our own solves. The
ranked reruns, which do not count toward the event's scoring, followed later from a fresh `retry`
on each mode and drew different fixtures (different case ids and `ANS-` values).

What had to generalize: two of the 15 Mastery sets carried the lane's harder hop mechanics rather
than a plain bytecode run — set 7 (rebuilding Day 15) and set 8 (rebuilding Day 16) each bundled a
native `lib/x86_64/libarena.so` and needed the VM's x86-64 hop, and set 10 (rebuilding Day 20)
carried a secondary `assets/payload.dex` loaded via `InMemoryDexClassLoader` and needed the VM's
cross-DEX loader-hop model (`Core.mixModuleb1` → `Guard.finLoaderfc` → `LoadedLater.moduleContract`,
3 secondary-DEX hop events). Both mechanics were already modeled in the library VM from earlier
days, so nothing needed to change; this Mastery just happened to be the first *graded* delivery to
actually exercise the payload.dex loader-hop path end to end (an earlier ranked attempt that
carried the same mechanic had expired before submission).

## Dead ends and gotchas

- **The Single Solve being team-scored meant its decode was never validator-confirmed.** The VM
  value fit the answer pattern and matched what we would have submitted, but `already_solved`
  graded nothing for us; only the Time Trial and Mastery results are real validator evidence that
  the Day 31 build runs correctly through the VM.
- **"Single Solve ships a new asset codec" was not a rule this time.** Days 28-30 had each
  introduced a new `v1` stored codec in their Single Solve while Time Trial/Mastery stayed `v2`.
  Day 31 broke that pattern: the Single Solve itself was `v2` `analysis` (the first time since Day
  23), so a solver that only knew how to decode a new `v1` codec would have had nothing to do here.
- **A `retry` reissues a different case and a different answer value each time**, as on every
  other dual-pool day — the regular and ranked Time Trial attempts produced different `ANS-...`
  strings for different `MOB-31-...` case ids, so a value from one attempt is not reusable on
  another.

## Takeaways

- The lane's running approach — identify the asset schema, then either decode a stored `v1` codec
  or run the app's own `Release.code()` in the Dalvik mini-VM for `v2` `analysis` — carried through
  unchanged to the event's last mobile challenge. No wrapper or VM change was needed on Day 31.
- A Mastery bundle that rebuilds many earlier days (15 sets spanning Days 1-31 here) is routine for
  this lane; always resolve each set from the delivery named by its own question's `artifact`
  field rather than assuming set order matches day order.
- The VM's native (x86-64/capstone) and secondary-DEX loader-hop features, both added for earlier
  days' challenges, were exercised again here without modification — evidence that generalizing
  the interpreter rather than hand-decoding each day's obfuscation paid off by the end of the event.

## Evidence

Base path: `challenges/day31/mobile-android/mobile-android-day-31-mobile-arena-gauntlet/`

- `single-solve/journal.md`, `single-solve/team_solved.json` (`already_solved`),
  `single-solve/start_response.json`, `single-solve/artifacts/cand-solo/solve.py`,
  `single-solve/artifacts/cand-solo/out.json`
- `time-trial/regular/result.json`, `time-trial/regular/accepted_plan.json`,
  `time-trial/ranked-1/result.json`, `time-trial/ranked-1/accepted_plan.json`
- `mastery/regular/result.json`, `mastery/regular/accepted_plan.json`,
  `mastery/regular/start_response.json` (15 questions with per-set `artifact`/case-id fields),
  `mastery/ranked-1/result.json`, `mastery/ranked-1/accepted_plan.json`
- `challenges/day31/_run/librarian/mobile-android-day-31-mobile-arena-gauntlet.md`
- `challenges/day31/_run/lane-mobile-android-day-31-mobile-arena-gauntlet.log`
- `challenges/day31/_run/champion-mobile-android-day-31-mobile-arena-gauntlet.json`
- `library/mobile-android/lane-evolution-and-playbook.md` (Day 31 lessons section: native-hop and
  payload.dex-hop set identification, verification counts)
- `progress.md` (grep `^| 31 .*mobile-arena-gauntlet`): Time Trial and Mastery rows, regular and
  ranked
