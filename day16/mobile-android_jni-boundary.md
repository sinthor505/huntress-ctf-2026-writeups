# JNI Boundary

| | |
|---|---|
| Day / Lane | 16 / Mobile Android |
| Challenge ID | `mobile-android-day-16-jni-boundary` |
| Technique | static recovery of an answer from an APK asset cross-checked against a native library's `.rodata` (JNI theme, decoy client secret) |
| Single Solve | Independent |
| Time Trial | Independent |
| Mastery | Independent |

## Summary

The prompt talks about a Java-to-native type or length assumption that changes backend authorization,
and the APK does ship a native library. The answer, though, never needed that logic traced. The value
sits in `assets/huntress_puzzle.json`, and the one answer-shaped string in `libarena.so`'s `.rodata`
equals it. The DEX only forwards it as `proof=` next to a decoy `SECRET-` client secret. We solved all
three modes statically, one submission each.

## The challenge

Prompt: "Identify a Java-to-native type or length assumption that changes backend authorization."
Each mode delivered APKs, not live targets.

- **Single Solve:** one APK, one answer id (`answer`), format `ANS-` followed by exactly 16 uppercase
  hex characters.
- **Time Trial:** one fresh APK, same answer format.
- **Mastery:** five APKs (`jni_boundary_mastery_01..05.apk`), answer ids `set1.answer` .. `set5.answer`,
  same format.

The APK carries `assets/huntress_puzzle.json` with a `codec` field. For this day's own build the codec
is `native` and `lib/x86_64/libarena.so` is present.

## Solution

1. **Triage the APK.** The asset JSON had `codec: "native"` and the package included
   `lib/x86_64/libarena.so`. The DEX held a `SECRET-` `client_secret` that the app sends beside the
   proof, which is the decoy pattern the lane uses.
2. **Check what the DEX does with the library.** In the Single Solve build the DEX has no `native`
   method and no `System.loadLibrary`; the library is shipped but never used. The Time Trial build (and
   Mastery set 5) declares `nativeEvidence(int)` and loads the library in `<clinit>`, but a static caller
   scan found no DEX method that calls it.
3. **Scan the library for answer-shaped strings.** A regex for `ANS-[0-9A-F]{16}` over every `lib/*.so`
   found exactly one string, and it was byte-identical to the asset's `data` value. That equality is the
   whole proof that the asset is the real value and not another decoy.
4. **Submit the asset value.** The extractor asserts all three conditions (one native string, codec
   `native`, asset value in the native set) before printing the answer JSON:

   ```python
   pat = re.compile(rb"ANS-[0-9A-F]{16}")
   native = set()
   for n in z.namelist():
       if n.startswith("lib/") and n.endswith(".so"):
           native |= set(pat.findall(z.read(n)))
   assert len(native) == 1 and asset["codec"] == "native" and asset["data"].encode() in native
   print(json.dumps({start["answer_ids"][0]: asset["data"]}))
   ```

   Single Solve answer: `ANS-901682D7DCA93404`. CORRECT on attempt 1, in one call.

The prompt's "length assumption" is thematic. In the timed builds the native export calls a
length-check export (`huntress_arena_length_accepts`) through the PLT and returns the address of the
same rodata string when the check accepts, otherwise NULL (capstone disassembly of both timed `.so`
copies, done later in a review pass). So it gates the one answer-shaped string rather than computing a
new one, and the asset cross-check was sufficient without tracing it.

## Time Trial and Mastery

- **Time Trial** repeated the same layout with a fresh case id and answer; the same mapper went CORRECT
  on attempt 1 (an untimed attempt, no deadline in the start response).
- **Mastery** delivered five APKs, themed on Days 1, 4, 11, 13 and 16, with codecs `plain`, `xor`, `xor`,
  `base64`, `native`. Only the native one ships a `.so`. The mapper became a per-delivery dispatcher over
  the asset's `codec` field: `base64` decodes the data, `xor` decodes base64 then XORs with the asset's
  `key`, `plain` and `native` take `data` as-is. Each decoded value is asserted to match
  `ANS-[0-9A-F]{16}`, and the key set is asserted equal to the start response's `answer_ids`.
  Set-to-APK pairing came from zipping `artifacts[]` with `questions[]`. CORRECT on attempt 1 with all
  five answers in one call.
- `gm.py artifact` fetches only the first Mastery artifact, so all five were downloaded from
  `artifacts[]` directly and SHA-256 checked.
- Later, in a ranked sweep on 2026-11-01 (bragging rights only, not the prize ranking), each of Time
  Trial and Mastery got two ranked attempts, all CORRECT in one call (Time Trial 3.95 s and 1.53 s in a
  50 s window; Mastery 5.01 s and 1.64 s in a 170 s window). No dedicated solver existed for this
  challenge; a thin delegate to the Day 15 generic APK solver handled it, because the codec is read from
  each APK's own asset JSON.

## Dead ends and gotchas

- **The theme is not the mechanism.** Nothing in the DEX calls `nativeEvidence`, and no authorization
  logic needed reversing. A solver who goes straight to disassembling the JNI export spends time on
  logic that only gates a string that is already in the asset.
- **Do not submit the DEX `client_secret`.** The `SECRET-` value is a decoy beside the real `proof=`
  value; only the asset/native `ANS-` value fits the answer pattern.
- **The fixtures are not uniform across modes.** The Single Solve build has no loader at all, while the
  timed builds declare `nativeEvidence(int)` and call `loadLibrary`. We could not tell from our files
  whether this was deliberate or an artifact of how the builds were generated. Re-check on a new day:
  a build whose native method is actually called, or whose `.so` holds no answer string, needs the
  export's own logic (Day 23 did exactly that).
- The three mode journals were written retroactively (the Single Solve journal says they were written
  after Day 19), so the narrative is reconstructed from the extractor scripts and submission responses
  rather than from live notes.

## Takeaways

Technique names: **asset-versus-rodata equality check** and **decoy client secret**. When an APK ships a
native library and the puzzle asset declares `codec: native`, regex the `.so` for the answer shape; if
exactly one candidate exists and it matches the asset, the answer is settled without running or
disassembling anything. Confirm first whether any DEX method actually calls the native entry point,
since a static caller scan is cheaper than tracing the export. For Mastery bundles, dispatch on each
APK's own `codec` field rather than assuming one codec per bundle.

## Evidence

Base: `challenges/day16/mobile-android/mobile-android-day-16-jni-boundary/`

- `single-solve/journal.md`, `single-solve/start_response.json`,
  `single-solve/artifacts/answer_map.py`, `single-solve/artifacts/submission-attempt-01-response.txt`
  (CORRECT)
- `time-trial/journal.md`, `time-trial/artifacts/submission-attempt-01-response.txt` (CORRECT),
  `time-trial/ranked-1/result.json`, `time-trial/ranked-2/result.json`
- `mastery/journal.md`, `mastery/artifacts/answer_map.py`, `mastery/artifacts/fetch_deliveries.py`,
  `mastery/artifacts/submission-attempt-01-response.txt` (CORRECT),
  `mastery/ranked-1/result.json`, `mastery/ranked-2/result.json`
- `tools/ranked/solvers/mobile-android-day-16-jni-boundary/solve.py` (delegate to the Day 15 generic solver)
- `library/mobile-android/jni-native-library/README.md` (MOBILE-ANDROID-JNI-001)
- `progress.md` (grep `^| 16 .*jni-boundary`)
