# Provider Paths

| | |
|---|---|
| Day / Lane | 6 / Mobile Android |
| Challenge ID | `mobile-android-day-06-provider-paths` |
| Technique | Static APK triage; answer stored in a `hex`-encoded asset (`huntress.mobile.puzzle.v1`), manifest/provider story is a decoy |
| Single Solve | Solved (journal-recorded): CORRECT on the first submission per the journal; no response or result file was saved |
| Time Trial | Independent (first-day attempt CORRECT, 1 submission; a later ranked rerun has a local `result.json` CORRECT) |
| Mastery | Independent (3 sets, 1 submission, CORRECT; a later ranked rerun has a local `result.json` CORRECT) |

## Summary

The prompt describes abusing an over-broad exported ContentProvider and correlating its records with intent evidence. None of that was needed. The APK ships its answer as a hex string in `assets/huntress_puzzle.json`, and decoding it gave an `ANS-` code that the Game Master accepted on the first try in every mode.

## The challenge

Prompt: "Abuse an overbroad provider path and correlate its records with prior intent evidence. Submit only the exact backend answer code returned for this case." Each question names a case (`MOB-06-...`) and a package (`com.huntress.mobile.d06.v...`), and points at an APK.

- Single Solve: one small APK, no deadline, answer ID `answer`.
- Time Trial: one APK, answer ID `answer`, a 50 second server-enforced window.
- Mastery: three APKs (set 1 to 3, answer IDs `set1.answer` to `set3.answer`), 110 second window. The start response carried `download: null` and `sha256: null`, so the APKs were fetched by delivery index and could not be hash-checked.
- Answer format in all modes: `ANS-` followed by exactly 16 uppercase hex characters, exact match.

## Solution

1. Listed the APK (`AndroidManifest.xml`, `classes.dex`, `assets/huntress_puzzle.json` plus signing files) and checked the SHA-256 against the start response.
2. Read the manifest with `aapt`. `.ProviderPathsProvider` is exported, has `grantUriPermissions=true`, and declares no path restrictions, which matches the prompt's "overbroad provider path". `ChallengeActivity` is exported and handles `huntress06://provider-paths/unlock` URIs.
3. Parsed the 3.4 KB DEX statically (Androguard's pure-Python DEX parser, nothing was installed or run). `ProviderPathsProvider.query()` returns `null` immediately and `insert`/`update`/`delete` are stubs, so there are no provider records to abuse. `ChallengeActivity` only forwards `answer` and `case_id` extras to a WebView request for an evidence endpoint. `MainActivity` shows the case reference from the question.
4. The real answer is in the asset, a 110-byte JSON record:

   ```json
   {"schema":"huntress.mobile.puzzle.v1","day":6,"codec":"hex","data":"414e532d34364132344243444145354336453142"}
   ```

   Hex-decoding `data` gives ASCII `ANS-46A24BCDAE5C6E1B` (20 bytes: `ANS-` plus 16 uppercase hex characters, matching the format). Package name and case reference in the APK matched the question, so this was the assigned build's value.
5. Submitted `ANS-46A24BCDAE5C6E1B` as `answer`. CORRECT, attempt 1.

The whole decoder is a few lines (from the Time Trial script in the leaf):

```python
with zipfile.ZipFile(apk) as archive:
    data = archive.read("assets/huntress_puzzle.json")
record = json.loads(data)
if record.get("codec") == "hex":
    decoded = bytes.fromhex(record["data"])
elif record.get("codec") == "base64":
    decoded = base64.b64decode(record["data"], validate=True)
else:
    raise ValueError(...)
```

## Time Trial and Mastery

Both modes used the same asset schema and the same `hex` codec, so the only change was the clock. The Time Trial window was 50 seconds. The decoder was written and tested on the Single Solve APK before the timer started, then run on the Time Trial APK; the answer was submitted about 4 seconds after the start and was CORRECT on the first call.

For Mastery, the three APKs were decoded in one pass and submitted as one request with all three IDs, within the 110 second window. The per-APK `aapt` package name was matched against the package in each question to pair APK to answer ID. CORRECT on the first call.

Later, in a ranked rerun of Time Trial and Mastery (each its own fresh attempt with new APKs), a general solver read each APK's declared codec from its own asset, paired questions to APKs by package name, and went CORRECT in one call each (about 1.4 to 1.5 seconds end to end). Each mode's `ranked-1/` folder holds that result.

## Dead ends and gotchas

- The prompt invites provider exploitation (exported provider, path traversal, intent evidence). In the APK the provider does nothing. Reading the manifest and DEX was enough to confirm that, so no time was lost trying to "abuse" it.
- Mastery gave no download capability and no hash, so there was nothing to verify the APKs against; pairing by package name was the only integrity check available.

## Takeaways

- In this lane, the stored answer is in the puzzle asset and the app code only forwards it beside decoys. Check `assets/huntress_puzzle.json` first and read its `codec` field instead of assuming one.
- Treat exported-component findings (`exported`, `grantUriPermissions`, null-returning `query()`) as leads to confirm in the DEX, not as the path to the answer.
- For short timed windows, prepare and test the decoder on a previous APK before starting the clock.

## Evidence

Leaf base path: `challenges/day06/mobile-android/mobile-android-day-06-provider-paths/`

- `single-solve/journal.md`, `single-solve/artifacts/huntress_puzzle.json`
- `time-trial/journal.md`, `time-trial/decode_puzzle_asset.py`, `time-trial/ranked-1/result.json`, `time-trial/ranked-1/accepted_plan.json`
- `mastery/journal.md`, `mastery/start-response.json`, `mastery/artifacts/decode_mastery_bundle.py`, `mastery/artifacts/submission-answers.json`, `mastery/artifacts/submission-response.json`, `mastery/ranked-1/result.json`
- General solver: `tools/ranked/solvers/mobile-android-day-06-provider-paths/solve.py`
- `progress.md` rows for Day 6 Provider Paths (all three modes "Solved", 2026-10-19).
