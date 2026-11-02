# Native Library Secrets

| | |
|---|---|
| Day / Lane | 15 / Mobile Android |
| Challenge ID | `mobile-android-day-15-native-library-secrets` |
| Technique | static JNI tracing: exported getter -> `.rodata` string in a bundled x86_64 `.so` |
| Single Solve | Solved (journal-recorded) |
| Time Trial | Independent (journal-recorded original attempt plus a ranked rerun with a saved `result.json`) |
| Mastery | Independent (journal-recorded original attempt plus a ranked rerun with a saved `result.json`) |

## Summary

The APK ships a challenge-owned x86_64 library, `libarena.so`, whose exported getter returns a
pointer to the answer string in `.rodata`. We recovered the value statically, never running the
APK, and cross-checked it against the `native` puzzle asset. The same approach solved all three
modes. Mastery bundled earlier-day APKs with other codecs, and only one of its five APKs actually
used the native library.

## The challenge

Prompt: "Recover split configuration from DEX and a challenge-owned native library", with the
instruction to "analyze the public APK and cross the challenge-owned JNI shared library boundary".
Each question asked for the exact backend answer code for one case (case id plus package name):
`ANS-` followed by exactly 16 uppercase hex characters, with exact normalization.

- **Single Solve:** one APK (about 13 KB of ZIP content: manifest, a 2.6 KB `classes.dex`,
  `assets/huntress_puzzle.json`, `lib/x86_64/libarena.so`), one answer field `answer`.
- **Time Trial:** one APK of the same shape (12,772 bytes), one answer field. The ranked window
  was 50 s.
- **Mastery:** five APKs, five fields `set1.answer` ... `set5.answer`, each bound to its own case id
  and package, under a window of about 170 s.

## Solution

All analysis was static. Nothing was installed or executed.

1. **Triage the APK as a ZIP.** `aapt dump badging` reported `native-code: x86_64`, and the
   manifest metadata named the day's mechanic as a "challenge-owned JNI shared library". The DEX
   string table contained a `NativeLibrarySecrets` class, a `jni_secret` string, and a decoy-looking
   client secret. None of these strings were treated as answer evidence on their own.
2. **Read the puzzle asset.** `assets/huntress_puzzle.json` declared
   `{"schema":"huntress.mobile.puzzle.v1","day":15,"codec":"native","data":"ANS-..."}`. The
   `native` codec means the stored `data` is not decoded, so the asset can only corroborate. The
   journal treated it as a lead and verified it against the library.
3. **Follow the getter.** The unstripped ELF exported two plain C symbols, not JNI-mangled names:
   `huntress_arena_secret` (17 bytes at `0x1000`) and `huntress_arena_day` (15 bytes at `0x1011`).
   `objdump -d -Mintel` showed the secret getter returning the address of `arena_value` at
   `0x2000`. The bytes there were a NUL-terminated 20-character string, `ANS-E28C9614AEF73B8F`.
   `huntress_arena_day` returned `0x0f` (15), matching the assigned day.
4. **Cross-check and submit.** The `.so` string equalled the asset's `data`, matched the
   `ANS-[0-9A-F]{16}` format, and the day constant matched the manifest. We submitted it under
   answer id `answer`. Single Solve went CORRECT on the first submission.

The extractor reduced the procedure to a single regex over the library bytes, with the asset as a
tie-breaker:

```python
CODE = re.compile(rb"ANS-[0-9A-F]{16}\x00")
...
values = CODE.findall(archive.read(libraries[0]))
if len(values) != 1:
    raise ValueError(f"expected one native code string in {apk.name}, got {len(values)}")
result = values[0][:-1].decode("ascii")
if data != result:
    raise ValueError(f"native getter and config disagree in {apk.name}")
```

Requiring exactly one answer-shaped string, and requiring it to equal the asset, is what made the
shortcut safe. If either check failed, the fallback was to trace the getter by hand.

## Time Trial and Mastery

**Time Trial.** The first `start` was accidentally an untimed assignment (the helper had been
called without the timed flag). We solved that variant (CORRECT on submission 1). A `retry --timed`
issued while it was still open returned HTTP 409 `attempt_active`, which consumed nothing. After
completion, `retry --timed` issued a fresh 50 s ranked assignment with a different package and
case id. The same extractor, run on the new APK bytes, went CORRECT on submission 1.

**Mastery.** The five deliveries were not five copies of the day's native specimen. The case ids
were `MOB-03`, `MOB-07`, `MOB-09`, `MOB-13` and `MOB-15`: earlier-day APKs in ascending order, with
the Day 15 native one last. The journal records the codecs as follows:

- Sets 1, 3 and 4 declared `base64`. Sets 3 and 4 decoded cleanly to the answer pattern; set 1 is
  recorded the same way.
- Set 2 declared `caesar` with an integer key. Subtracting the declared shift from each code point
  gave the answer.
- Set 5 declared `native` and used the `libarena.so` path above.

The mastery `answer_map.py` read each APK's own asset, applied only the declared codec, validated the
pattern, and paired question `i` with delivery `i`. All five fields were accepted together in one
call. A later ranked rerun with a saved `result.json` also went CORRECT in one call, in 1.74 s total
against a 170 s window.

## Dead ends and gotchas

- **Do not assume every Mastery APK uses the current day's codec.** From Day 15 on, the bundle reuses
  builds of earlier days. A solver hard-wired to the native path would have failed on four of five
  fields, and one hard-wired to base64 would have failed on two (Caesar, native). The fix is to
  dispatch on each APK's own declared `codec`.
- **The DEX strings are not the answer source.** The DEX carries a client-secret-looking value and a
  local evidence URL, which the journal deliberately did not use. The native getter is the authority,
  with the asset as corroboration.
- **Same file name overwrote evidence.** The Time Trial's untimed and ranked downloads were saved
  under the same name, so the untimed APK was lost and its map could not be re-verified later (noted in
  the journal's deep review). Save downloads per attempt.
- No format traps: the answer was accepted as the bare `ANS-` + 16 uppercase hex string.

## Takeaways

Name the technique: **static JNI-getter-to-rodata tracing** (library entry MOBILE-ANDROID-JNI-001).
When a puzzle asset declares `codec: native`, list the ELF exports, disassemble the small getter, and
follow its returned pointer into `.rodata`. Use the asset only to confirm. A regex for a single
answer-shaped NUL-terminated string is a fast path; keep the disassembly as the fallback for builds
where the counts disagree. Later days (Day 16 onward) changed the pattern: some `.so` files export a
real `Java_..._nativeEvidence` function, and by Day 23 the library holds no answer string at all, only
one arithmetic hop of a computed chain, so do not extend this shortcut blindly.

## Evidence

Leaf: `challenges/day15/mobile-android/mobile-android-day-15-native-library-secrets/`

- `single-solve/journal.md`, `single-solve/start_response.json`, `single-solve/artifacts/answer_map.py`
- `time-trial/journal.md`, `time-trial/start_response.json`, `time-trial/artifacts/answer_map.py`,
  `time-trial/ranked-1/result.json`
- `mastery/journal.md`, `mastery/start_response.json`, `mastery/artifacts/answer_map.py`,
  `mastery/ranked-1/result.json`
- `library/mobile-android/jni-native-library/README.md` (MOBILE-ANDROID-JNI-001)
- `tools/ranked/solvers/mobile-android-day-15-native-library-secrets/solve.py` (general solver used for
  the ranked reruns)
- `progress.md` (grep `^| 15 .*native-library-secrets`)

The original 2026-10-20 attempts for all three modes are recorded as CORRECT in the leaf journals and
`progress.md`, but no result file was saved for them. The ranked reruns of 2026-10-26 for Time Trial and
Mastery are the saved graded results. Single Solve has only the journal and `progress.md` record.
