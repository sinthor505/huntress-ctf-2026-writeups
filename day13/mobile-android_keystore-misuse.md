# Keystore Misuse

| | |
|---|---|
| Day / Lane | 13 / Mobile Android |
| Challenge ID | `mobile-android-day-13-keystore-misuse` |
| Technique | declared-codec asset decode (static APK read); the "Keystore" theme is flavor text |
| Single Solve | Solved (journal-recorded) |
| Time Trial | Independent (ranked reruns); original attempt journal-recorded |
| Mastery | Independent (ranked reruns); original attempt journal-recorded |

## Summary

The prompt promised an app that wraps data with an app-controlled key while nominally using the
Android Keystore. None of that was needed. Every build carried its answer as a Base64 string in
`assets/huntress_puzzle.json`, with the codec declared in the same file. Reading the APK as a ZIP
and decoding that string gave the `ANS-` code in one step, in all three modes.

## The challenge

Prompt: "Recover data protected by an app-controlled wrapping key despite nominal Keystore use."
Each question named a case (`MOB-13-<hex>`) and a package (`com.huntress.mobile.d13.v<hex>`) and
asked for "the exact backend answer code returned for this case". The answer field was a pattern
(`ANS-` plus exactly 16 uppercase hex characters) with `exact` normalization.

- Single Solve: one field, `answer`, one untimed artifact. The download was a ZIP-named file that
  was in fact the APK (saved as `keystore-evidence.zip`).
- Time Trial: one field, one fresh APK, a 50 s clock on the original attempt.
- Mastery: three fields (`set1.answer` to `set3.answer`), three separate APK deliveries, 110 s on the
  original attempt.

## Solution

1. **Recon.** The artifact SHA-256 matched the start response. `file` and `unzip -l` showed an
   ordinary APK holding `AndroidManifest.xml`, `classes.dex` and `assets/huntress_puzzle.json`.
   The manifest had an exported, browsable `ChallengeActivity` on a `huntress13://keystore-misuse/unlock`
   route, and the DEX strings carried a Day 13 mechanic marker and an evidence-request URL. We read
   these as observations only and did not install or run the app.
2. **The asset.** The JSON was tiny and named its own transform:

   ```json
   {"schema":"huntress.mobile.puzzle.v1","day":13,"codec":"base64","data":"QU5TLTc3Mzc2MkU4NTc5NzA2NEM="}
   ```

3. **Decode and check.** Base64-decoding `data` gave `ANS-773762E85797064C`, which fits the
   required pattern. The mapper pinned the APK hash, checked the schema, day and codec, decoded with
   `validate=True`, and rejected anything that did not match `ANS-[0-9A-F]{16}`:

   ```python
   answer = base64.b64decode(puzzle["data"], validate=True).decode("ascii")
   if not re.fullmatch(r"ANS-[0-9A-F]{16}", answer):
       raise SystemExit("decoded value does not match the required answer format")
   ```

4. **Submit.** One call with `answer = ANS-773762E85797064C` returned CORRECT.

## Time Trial and Mastery

Both modes issued fresh APKs with the same `puzzle.v1` / `base64` shape and different data, so the
Single Solve mapper's logic carried over unchanged. Only the pinned hash and answer ids changed. The
journals record the following values, each submitted in one call and graded CORRECT:

- Time Trial: `ANS-8D9E9BBB4DA4808E`.
- Mastery: `set1` `ANS-9ED753498DB0BCB3`, `set2` `ANS-6EB4390198502B30`, `set3` `ANS-0A00AA01BCA6F42A`.
  Each answer id had to be paired with its own delivery. The solver matched them by the package name in
  the question text against the APK manifest, falling back to delivery name and set order.

We later re-ran both timed modes as ranked attempts. The ranked Time Trial still had one field and a
50 s window. The ranked Mastery had **five** sets and a 170 s window instead of three sets in 110 s.
That was the only scale change. The general solver (reads the codec from each APK's own asset, pairs
questions to APKs by package) handled it with one submission call each, total 1.5 to 4.3 s. Three ranked
passes per mode all went CORRECT.

## Dead ends and gotchas

- The title and prompt point at Keystore, wrapping keys and cryptographic misuse. The Day 13 builds
  contained none of it. A later census of the saved Day 13 builds found only a `describe()` method in
  the theme class and no Keystore or Cipher calls. Reading the declared asset first avoided a long
  detour into the DEX.
- The manifest's exported `ChallengeActivity` and the DEX's evidence-request URL look like leads. The
  journal records them only as observations and says the answer was mapped from the asset alone.
- The Single Solve download was ZIP-named, not `.apk`. It is the APK.
- No rejected submissions in any mode; nothing about the validator surprised us (`exact`, upper-case
  hex, and the decoded text is already in that form).

## Takeaways

For this lane's early `puzzle.v1` days, open `assets/huntress_puzzle.json` before anything else,
treat the title as flavor, and decode whatever `codec` the file declares. Checking the result against
the answer pattern and the APK hash against the start response catches a stale or corrupted copy. For
Mastery bundles, pair each answer id to its APK by an identifier in the question (the package name
here) rather than by order. Searchable names: Android puzzle asset, declared-codec decode,
`huntress.mobile.puzzle.v1`.

## Evidence

Leaf: `challenges/day13/mobile-android/mobile-android-day-13-keystore-misuse/`

- `single-solve/journal.md`, `single-solve/start_response.json`, `single-solve/artifacts/answer_map.py`,
  `single-solve/artifacts/candidate_answers.json` (no saved `result.json`: the CORRECT is recorded in
  the journal and the `progress.md` row)
- `time-trial/journal.md`, `time-trial/start_response.json`, `time-trial/ranked-1/result.json`,
  `time-trial/ranked-2/result.json`, `time-trial/ranked-3/result.json`
- `mastery/journal.md`, `mastery/start_response.json`, `mastery/ranked-1/result.json`,
  `mastery/ranked-2/result.json`, `mastery/ranked-3/result.json`
- `tools/ranked/solvers/mobile-android-day-13-keystore-misuse/solve.py` (general ranked solver)
- `library/mobile-android/lane-evolution-and-playbook.md` (Days 13-23 table)
- `progress.md` (grep `^| 13 .*keystore-misuse`)
