# Protocol Reconstruction

| | |
|---|---|
| Day / Lane | 28 / Mobile Android |
| Challenge ID | `mobile-android-day-28-protocol-reconstruction` |
| Technique | APK puzzle asset with a new framed-hex codec (`packet`); v2 builds computed by running the app's own bytecode |
| Single Solve | Team solve (not ours) |
| Time Trial | Independent |
| Mastery | Independent |

## Summary

Each Mobile Android challenge hides an `ANS-<16 hex>` code in the APK. On Day 28 the Single Solve
asset used a codec we had not seen, a hex string of checksummed `HA` frames. We decoded it offline,
but a teammate had already scored that Single Solve, so our submission returned `already_solved`
and our decode was never graded. Time Trial and Mastery were fresh attempts issued with `retry`.
Both went CORRECT on the first submission with the same solver.

## The challenge

Prompt: "Reconstruct the app-to-backend protocol and preserve nonce and ordering semantics." Each
question names a case (`MOB-28-...`) and an Android package and asks for "the exact backend answer
code returned for this case". The answer is a `pattern` field, `ANS-` followed by exactly 16
uppercase hex characters, with `exact` normalization.

- **Single Solve:** one APK (`protocol_reconstruction.apk`), one answer id `answer`.
- **Time Trial:** one APK, one answer id. The ranked attempt window was 50 s.
- **Mastery:** 12 APKs, one answer per set (`set1.answer` ... `set12.answer`). The packages are
  rebuilds of earlier days' mobile challenges (Days 1, 4, 6, 9, 11, 14, 17, 18, 22, 23, 25) plus a
  Day 28 build as set 12. The ranked window was 380 s.

Neither the title nor the prompt mattered for the solve. "Nonce and ordering semantics" is theme
text. The backend call in the APK only forwards the stored value as `proof=` next to a decoy
`client_secret`.

## Solution

1. **Recon.** The Single Solve APK was downloaded and its SHA-256 checked against the start response
   before analysis. The puzzle asset in these APKs is a JSON file (`huntress.mobile.puzzle.v1`)
   with a `codec` name and the encoded data.
2. **The new codec.** Day 28's v1 asset used a codec called `packet`. The data is hex. Decoded, it is
   a frame: the bytes `HA`, one length byte, the payload, then one XOR checksum over all the
   preceding frame bytes. The payload is the `ANS-` text. The checksum passed on this build, which
   confirmed the framing.
3. **Library change.** `decode_packet` was added to the shared asset-codec library
   (`asset_codecs.py`, entry MOBILE-ANDROID-CODECS-001). It raises on a bad magic, length or
   checksum, or if the result does not fit the answer pattern. If a stream held several frames, it
   joins the payloads in stream order, or takes the single frame that matches the pattern.
   The journal calls this multi-frame behaviour a guess, because the Single Solve held only one
   frame.

   ```python
   if raw[i:i + 2] != b"HA": raise CodecError("packet: bad frame magic ...")
   n = raw[i + 2]; end = i + 3 + n
   x = 0
   for b in raw[i:end]:
       x ^= b
   if x != raw[end]: raise CodecError("packet: checksum mismatch ...")
   frames.append(raw[i + 3:end].decode("utf-8"))
   ```
4. **Single Solve submission.** The one submission came back `already_solved` (the team's attempt
   scored it), and the lane log records "nothing graded for us". The decoded value matched the
   `ANS-` pattern, but it is unvalidated. We do not claim this mode.

## Time Trial and Mastery

`start` returned the team's CORRECT readout for both modes. That is not our result, so we issued a
`retry` for each, which gave a fresh attempt and fresh APKs. We solved a regular untimed attempt and
a ranked timed attempt of each. All four submissions were CORRECT on call 1, with local totals
of about 5 s (Time Trial 4.97 s regular and 4.78 s ranked, Mastery 7.71 s regular and 5.92 s ranked).

The solver was a thin wrapper over two library tools. APKs with a v1 asset go through
`asset_codecs.resolve_start`. If the APK declares a `Release` class (v2 builds, `codec: "analysis"`),
the wrapper runs `Release.code()` in the Dalvik mini VM instead. For Time Trial and Mastery
the chosen candidate came from that wrapper. Which path each timed APK took is not recorded in
any file we were allowed to read. The adapter's expectation was v2, the library playbook says
Mastery set 12 is a Day 28 v2 rebuild, and a v2 build has no stored answer data at all, so the codec
work above only mattered to the Single Solve. We could not determine from the evidence whether the
Time Trial APK was v1 or v2.

Mastery needed no per-set special cases. Each of the 12 deliveries was solved by the same wrapper
and submitted in one call.

## Dead ends and gotchas

- **The Single Solve cannot be re-graded.** Once a teammate scores it, `already_solved` is the only
  response. The offline `packet` decoder was never checked by the validator, and the library marks
  it `observed`, not confirmed. The pipeline saw the same thing on later days.
- **Multi-frame handling is untested against a real multi-frame asset.** The nonce/ordering wording in
  the prompt hinted at more than one frame, but the only v1 asset we saw had one.
- **The second candidate (rubric) never resolved a timed delivery.** On every Time Trial and Mastery
  set it exited with `unresolved ids: ['answer']`, so the wrapper candidate was submitted. We did
  not determine why. It did not matter here.

## Takeaways

- Technique names to search for: **puzzle-asset codec recovery from the known `ANS-` prefix**,
  **framed hex with an XOR checksum**, and **running a v2 APK's own `Release.code()` in a Dalvik VM**.
- The checksum is the best validator for a guessed framing: a wrong guess does not pass, so a
  pass means the framing is right.
- Do not assume a Mastery bundle is the day's own codec. It was 12 rebuilds of 12 different days.
- When a teammate scores the Single Solve, solve Time Trial and Mastery with `retry` anyway. They are
  separate attempts and the grader scores them independently.

## Evidence

Leaf base: `challenges/day28/mobile-android/mobile-android-day-28-protocol-reconstruction/`

- `single-solve/journal.md`, `single-solve/start_response.json`, `single-solve/team_solved.json`
- `single-solve/artifacts/cand-adapter/solve.py` (wrapper that was used for all three modes)
- `time-trial/ranked-1/result.json`, `time-trial/ranked-1/accepted_plan.json`,
  `time-trial/regular/result.json`
- `mastery/ranked-1/result.json`, `mastery/ranked-1/accepted_plan.json`,
  `mastery/regular/result.json`
- `challenges/day28/_run/lane-mobile-android-day-28-protocol-reconstruction.log`,
  `challenges/day28/_run/workers/mobile-android-day-28-protocol-reconstruction-adapter-r0.log`
- `library/mobile-android/asset-codecs/asset_codecs.py` (`decode_packet`),
  `library/mobile-android/lane-evolution-and-playbook.md`
- `progress.md` (grep `^| 28 .*protocol-reconstruction`)
