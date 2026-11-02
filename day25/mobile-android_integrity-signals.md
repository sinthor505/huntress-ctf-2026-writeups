# Integrity Signals

| | |
|---|---|
| Day / Lane | 25 / Mobile Android |
| Challenge ID | `mobile-android-day-25-integrity-signals` |
| Technique | puzzle-asset decoding (Single Solve) and running the APK's own bytecode in a Dalvik interpreter (Time Trial / Mastery) |
| Single Solve | Independent |
| Time Trial | Independent |
| Mastery | Independent |

## Summary

The prompt talks about forging or replaying an "integrity statement" and finding the missing binding.
That is theme text only: every delivery is a puzzle APK whose real answer is an `ANS-` code that the
app computes or stores itself. The Single Solve was a stored hex asset that decoded in one call. Time
Trial and all nine Mastery sets were the newer `analysis` build, where nothing is stored and the code is
the output of the APK's own `Release.code()` method, so we ran that bytecode in a Dalvik interpreter.
Two Mastery builds hit constructs the interpreter did not yet model; after those were added, all three
modes went CORRECT.

## The challenge

Prompt: "Forge or replay a bounded challenge-owned integrity statement and explain the missing binding."
Each question names a case (for example `MOB-25-...`) and a package (`com.huntress.mobile.d25.v...`) and
asks for "the exact backend answer code returned for this case". Answer format: `ANS-` followed by exactly
16 uppercase hex characters, exact match, as a JSON string.

- **Single Solve:** one APK (`integrity_signals.apk`), one answer id (`answer`), untimed.
- **Time Trial:** one APK, one answer, a 50 s server-enforced window on the ranked attempt.
- **Mastery:** nine APKs delivered together, answer ids `set1.answer` ... `set9.answer`, a 290 s window
  on the ranked attempt. The sets are rebuilds of earlier days' challenges (the case ids read `MOB-01`,
  `MOB-02`, ... in order of the source day); the last set is the current day's.

Nothing in any APK implements an integrity check, a signature or an attestation. There is no
forge/replay step to perform; the "missing binding" in the prompt has no counterpart in the artifacts.
This lane's answer is always carried by the app's own puzzle logic rather than by any framework API
behind the theme.

## Solution

### Single Solve (v1 `hex` asset)

1. **Triage.** The APK carries `assets/huntress_puzzle.json` with schema `huntress.mobile.puzzle.v1`,
   asset `day` 25 and codec `hex`. A `client_secret` in `classes.dex` is a decoy; the dex only forwards
   the real value as a `proof=` parameter beside it.
2. **Decode.** The `hex` codec is the hex of the answer's UTF-8 bytes. The library codec tool decoded it
   and the result matched the stated pattern. The solver linted the value against `ANS-[0-9A-F]{16}`
   before submitting.
3. **Submit.** One call, CORRECT, about two minutes after the challenge unlocked. The accepted answer was
   `ANS-1333EAAB62D23DAB`. The pattern forces upper case, so there was no alternate reading to hold in
   reserve.

### Time Trial (v2 `analysis`)

The Time Trial APK was the second puzzle schema, `huntress.mobile.puzzle.v2` with codec `analysis`. Its JSON
holds no stored answer at all. `Release.code()` computes the code on-device: a chain of about ten calls
through themed helper methods, a final fold and a XOR with a 64-bit constant, printed as 16 upper-case
hex nibbles with the `ANS-` prefix. A sibling `proof()` method hashes the same token with a different
ending and is a decoy, as are several never-executed `seal*` helpers.

We did not decode by hand. The library's Dalvik mini-VM interpreter ran the app's own `Release.code()`
and produced the value, which was accepted on the first graded call.

### Mastery (nine v2 builds)

All nine deliveries were `analysis` builds from earlier days' challenges, so the same VM ran each one and
produced one flat `{answer_id: value}` map for the nine ids. The regular (rehearsal) bundle held builds
rebuilt from days 1, 4, 7, 10, 14, 16, 19, 22 and 25; the ranked bundle held days 2, 5, 7, 12, 14, 18,
20, 23 and 25. Bundles differ between attempts, so the regular CORRECT did not by itself prove the ranked
fixtures would decode.

## Time Trial and Mastery

Scale: 1 -> 1 APK for the Time Trial and 1 -> 9 for Mastery, with 50 s and 290 s ranked windows. The
ranked Time Trial decoded in 3.84 s and the ranked Mastery in 4.75 s, one graded call each. Two later
ranked reruns on 2026-10-31 and 2026-11-01 were CORRECT as well (Time Trial 4.26 s and 4.30 s, Mastery
16.19 s for the one Mastery rerun on file).

What had to be generalized came from two regular Mastery builds that the interpreter could not run
(it names an unsupported construct instead of guessing, which made these easy to find):

- **Set 6, a Day 16 "JNI Boundary" build.** Its native export is `nativeShare(JI)J`: two arguments, and the
  machine code begins with a guard on the int argument (`cmp ecx,0xffff; jle` return 0;
  `movzx esi,cx; cmp esi,K; jne` return 0) before a 64-bit fold. The existing x86-64 model took one
  argument and had no branches. The model was rewritten to take the JNI argument registers, handle
  sub-registers, `movzx`/`movsx`/`lea`, `cmp`/`test` flags and conditional jumps. Tracing the set showed the
  Java side passes an int above 0xffff whose low 16 bits match the gate, so the result is the real
  value and not the gate-fail 0.
- **Set 7, a Day 19 "Reflection Maze" build.** `ReflectionMaze.dispatch(I)I` uses a `packed-switch` to pick
  a method number, the code builds `"m" + n`, then runs `Class.forName(...)` on a class of the same DEX
  followed by `getDeclaredMethod` and `Method.invoke`. The VM gained `packed-switch`/`sparse-switch` and
  `Class.forName` for classes in the same DEX. A class outside the DEX still raises.

Two independent implementations of these additions agreed on all nine sets. A regression over every
archived v2 APK from Days 23-25 gave 52/52 matches, and a later library check compared the VM's output
with the accepted answers field by field (20/20 on the Day 25 Time Trial and Mastery attempts).

One of the ranked Mastery sets (set 7, a Day 20 "Loaded Later" build) carries an `assets/payload.dex`
second stage that is loaded through an `InMemoryDexClassLoader`. This hop had been added to the VM after
the previous day's ranked Mastery failed on it; here it ran and the answer was accepted, which is the
first graded confirmation of that path.

## Dead ends and gotchas

- **Later schemas break the hex shortcut.** The Single Solve being v1 `hex` and everything else being v2
  `analysis` is the reason a solver that only reads the stored asset passes the Single Solve and
  fails everything after it. The codec tool correctly reports that a v2 asset has no stored answer.
- **Output shape of the interpreter.** The first Time Trial attempt failed offline because the solver
  expected an `{"answers": {...}}` wrapper while the VM writes a flat `{id: value}` map, so nothing was
  submitted. That is our own tooling's contract bug (see the Day 25 overview), not a challenge trap.
- **A wrong gate looks like a right answer in the JNI build.** The set 6 native returns 0 when its guard
  fails, which still yields a well-formed `ANS-` string through the fold. Checking that the guard
  passed, not just that the output matched the pattern, is the way to tell.
- **Decoys beside the real value.** `client_secret`, `proof()` and the unexecuted `seal*` methods all look
  plausible. Running the code and not trusting names is what separates them.
- The stronger model was used for the second fix round on regular Mastery; both of its changes, and an
  independent version of the same changes, are described above. Submissions to the validator were not
  the bottleneck: Single Solve 1 call, Time Trial 2 (regular, ranked), Mastery 2 (regular, ranked).

## Takeaways

- **Execute, do not decode.** When a mobile puzzle's JSON stores no answer (`codec: analysis`), run the
  app's own `Release.code()` bytecode in a Dalvik interpreter; static reading of the helper chain, the
  `proof()` method or the secret string misleads.
- **Prefer an interpreter that fails loudly.** An unsupported framework call or opcode raising by name
  is a work order for exactly what to add, and the tool's regression set (all earlier accepted
  answers) keeps the additions honest.
- **Mastery bundles reuse earlier days' builds.** Every earlier-day rebuild in the bundle can introduce a
  construct the current day's Single Solve never had (branching native exports, switches, reflection,
  a second-stage DEX).
- Keep the theme and the artifact separate: the prompt's "forge or replay an integrity statement"
  described nothing in the APK.

## Evidence

Leaf base: `challenges/day25/mobile-android/mobile-android-day-25-integrity-signals/`

- `single-solve/journal.md`, `single-solve/start_response.json`, `single-solve/accepted_plan.json`
- `single-solve/artifacts/cand-adapter/solve.py` (v1 asset codec first, VM fallback)
- `single-solve/artifacts/cand-rubric/solve.py`, `single-solve/artifacts/cand-rubric/dvm_r2.py` (leaf copy of the VM with the set 6/7 additions)
- `time-trial/ranked-1/result.json`, `time-trial/regular/result.json`, `time-trial/ranked-2/result.json`, `time-trial/ranked-3/result.json`
- `mastery/ranked-1/result.json`, `mastery/regular/result.json`, `mastery/ranked-2/result.json`, and each `accepted_plan.json`
- `challenges/day25/_run/librarian/mobile-android-day-25-integrity-signals.md`
- `challenges/day25/_run/lane-mobile-android-day-25-integrity-signals.log`
- Library entries: MOBILE-ANDROID-DALVIKVM-001 (Dalvik mini-VM, Day 25 additions), MOBILE-ANDROID-CODECS-001 (asset codecs), MOBILE-PLAYBOOK-001 (lane evolution, Day 25 lessons)
- `progress.md` (grep `^| 25 .*integrity-signals`)
