# Debug Me If You Can

| | |
|---|---|
| Day / Lane | 21 / Mobile Android |
| Challenge ID | `mobile-android-day-21-debug-me-if-you-can` |
| Technique | read the answer from the APK's `huntress_puzzle.json` asset (`reverse` codec); the anti-debug theme is decoration |
| Single Solve | Independent |
| Time Trial | Independent |
| Mastery | Independent |

## Summary

The prompt asks you to "neutralize a clean-room anti-debug state machine", which suggests patching
or hooking an APK. No debugger, patching or bytecode work was needed. Every build in this lane
carries its answer in `assets/huntress_puzzle.json`, encoded with one fixed codec per day. For Day 21
that codec is `reverse` (the string reversed). Decoding the asset gave the `ANS-` code, and all three
modes went CORRECT on the first submission.

## The challenge

Prompt: "Neutralize a clean-room anti-debug state machine while retaining the intended app flow."
The question for each build names a case (`MOB-21-<10 hex>`) and a package
(`com.huntress.mobile.d21.v<hex>`) and asks for "the exact backend answer code returned for this
case". Answer format: `ANS-` plus 16 uppercase hex characters, `exact` normalization.

- **Single Solve:** one APK (`debug_me_if_you_can.apk`), one answer id, `answer`.
- **Time Trial:** one APK, one answer id. The ranked window was 50 s.
- **Mastery:** five APKs (`set1.answer` .. `set5.answer`), ranked window 170 s. The sets are not all
  Day 21 builds: the case ids carry the build's own day, and in the five Mastery deliveries we saw
  they were four earlier days plus one Day 21 build (for example days 3, 10, 13, 19 and 21 in the
  regular attempt, and 2, 6, 15, 18 and 21 in the first ranked one).

## Solution

1. **Recon.** The APK's SHA-256 matched the start response. We triaged the build with our APK
   triage helper. It showed no debugger-detection API behind the theme, and the only secret-looking
   string in `classes.dex` was a `SECRET-` constant, which is a decoy.
2. **The key insight.** The lane's puzzle asset, `assets/huntress_puzzle.json`, carries a `day` field
   and a `codec` field, and the codec is fixed per day. Day 21's is `reverse`. The decoded value
   matched `ANS-[0-9A-F]{16}`, and the DEX contains no string of that shape, so the asset value was
   the answer. The DEX only forwards a value as `proof=` next to the decoy `client_secret`.
3. **Decode.** The solver opens the APK as a ZIP and finds the single `huntress_puzzle.json`. It
   applies the codec named in the asset and checks the result against the answer pattern before
   writing it out. The codec table is small and direction conventions are the usual way to get it
   wrong:

   ```python
   if c == "reverse":
       return a["data"][::-1]
   if c == "split":
       return permute(a["order"], a["parts"])   # order[i] = output slot of parts[i]
   if c == "indexed":
       return permute(a["order"], a["data"])
   ```

4. **Submit.** The Single Solve answer, accepted on the first call, was `ANS-B40FD52516FEF25A`.

The solver also repairs LF-to-CRLF damage in a saved ZIP in memory, a standing precaution in this
workspace.

## Time Trial and Mastery

- **Time Trial** was the same decode on a fresh build (new case id and package suffix). It went
  CORRECT in one call, both as an untimed regular attempt and as ranked attempts under the 50 s
  window. Our local total (issue, download, solve, submit) ranged from about 1.7 s to 7.3 s across
  the ranked runs.
- **Mastery** had to handle the mixed days. The solver does not hard-code Day 21's codec: it reads
  each delivery's own asset and dispatches on its `codec` (`plain`, `reverse`, `base64`, `hex`,
  `caesar`, `xor`, `split`, `indexed`, `reflection`, `dynamic`). The bundles we saw
  mixed `base64`, `split`, `reflection`, `reverse`, `hex`, `xor`, `caesar`, `native` and `dynamic`
  builds (the codec per day is listed in the library's lane playbook), and the same solver covered
  all of them. Answer ids are
  `setN.answer`, with deliveries mapped to ids in order. One call submitted all five answers each
  time, CORRECT in the regular attempt and in each ranked attempt we ran (local totals 3.4 s, 3.6 s
  and one slower 68 s run whose cause the files do not record; all inside the 170 s window).
- **The theme method is dead code.** From Day 16 the Time Trial and Mastery builds of the current
  day add a theme-named method that the Single Solve build lacks; for Day 21 it is
  `debuggerGate(ZI)I`. We checked its callers: there are none in the DEX. The asset value was still
  the answer, as every accepted map showed.

## Dead ends and gotchas

- **The title and prompt are not literal here.** There is nothing to debug and no state machine that
  has to be neutralized for the answer; the build's behavior does not matter. Read the asset before
  reaching for a patching workflow.
- **`classes.dex` has a decoy.** The only `SECRET-` constant is next to the `proof=` forwarding code
  and is not accepted as the answer; the DEX has no `ANS-` pattern.
- **Mastery bundles are re-drawn.** The regular and ranked Mastery attempts shared only their Day 21
  set; the other four were different earlier-day builds. Each question's case id carries the build's
  own day, so resolve the codec per delivery rather than assuming one for the bundle.
- **Direction conventions for `split`/`indexed`.** `order[i]` is the output slot of item `i` of
  `parts`/`data`, not the other way around. The Day 21 build did not use them, but the Mastery
  mixes did, so the solver had to match them.
- No answer-format surprises: the plain decoded `ANS-` string was accepted as is.

## Takeaways

Technique name: **puzzle-asset decode** (asset codec per day). For this lane, check
`assets/huntress_puzzle.json` and triage the APK for callers of any theme method before building any
dynamic-analysis tooling. A day's codec is stable across its Single Solve, Time Trial and Mastery
builds, so a decoder proven on one set works on the others; the part that changes in Mastery is
which day each set belongs to. Only when the asset stops carrying a stored value (a later schema
whose answer is computed by the app's own bytecode) is code execution needed, and that did not
apply on Day 21. We could not determine from the files why the challenge was themed around
debugger detection when nothing in the build uses it.

## Evidence

Leaf: `challenges/day21/mobile-android/mobile-android-day-21-debug-me-if-you-can/`

- `single-solve/journal.md`, `single-solve/start_response.json`, `single-solve/accepted_plan.json`
- `single-solve/artifacts/cand-adapter/solve.py` (the decoder shown above; `cand-rubric/solve.py` is
  an equivalent second implementation)
- `time-trial/regular/result.json`, `time-trial/ranked-1/result.json` (ranked-2, ranked-3 and
  ranked-vm-1 hold further CORRECT results)
- `mastery/regular/result.json` and `mastery/regular/accepted_plan.json`, `mastery/ranked-1/result.json`
  (ranked-2 and ranked-3 also CORRECT)
- `challenges/day21/_run/lane-mobile-android-day-21-debug-me-if-you-can.log`
- `library/mobile-android/` (asset codec reference and lane playbook, Day 21 row)
- `progress.md` (grep `^| 21 .*debug-me-if-you-can`)
