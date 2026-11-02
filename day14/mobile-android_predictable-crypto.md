# Predictable Crypto

| | |
|---|---|
| Day / Lane | 14 / Mobile Android |
| Challenge ID | `mobile-android-day-14-predictable-crypto` |
| Technique | declared puzzle-asset codec (Caesar); the prompt's "seeded nonce" crypto is a decoy |
| Single Solve | Independent (journal plus saved CORRECT submission response) |
| Time Trial | Independent (saved CORRECT submission response; ranked reruns also CORRECT) |
| Mastery | Independent (saved CORRECT submission response; ranked reruns also CORRECT) |

## Summary

The prompt promises a seeded-nonce or weak-key-derivation attack on an Android app, and the manifest
even advertises a deep-link activity. None of that existed in the code. Each APK carried a small JSON
asset that declared a Caesar codec, and decoding it gave a string in exactly the required answer
format. The DEX only forwards a caller-supplied answer to a local evidence URL. All three modes went
CORRECT on the first submission.

## The challenge

Prompt: "Exploit seeded nonces or weak derivation without bypassing earlier storage gates." The one
question (`answer`) asked for "the exact backend answer code returned for this case", naming a case id
and package (e.g. `com.huntress.mobile.d14.v...`). The format was a pattern, `ANS-` followed by
exactly 16 uppercase hex characters, with `exact` normalization.

Delivered: a small APK (API level 23, no native code) with one `classes.dex`, an
`assets/huntress_puzzle.json` of about 118 bytes, and signing metadata. Single Solve and Time Trial
each had one APK and one answer id. Mastery's first (regular) start delivered three APKs under one
timer (`set1.answer` .. `set3.answer`), one per set.

## Solution

1. **Recon.** The APK hash was checked against the start response. The manifest shows
   `debuggable=false`, `allowBackup=false`, the INTERNET permission, and an exported, browsable
   `ChallengeActivity` on a custom URI scheme. That fits the prompt's "storage gate" framing, so the
   code needed reading before anything was assumed.
2. **The asset.** `assets/huntress_puzzle.json` has schema `huntress.mobile.puzzle.v1` and
   `codec: "caesar"` with a key (9 in the Single Solve build). Decoding is `chr(ord(c) - key)` per
   code point, with no alphabet wrap. The result matched `ANS-[0-9A-F]{16}`.
3. **Checking the code before trusting it.** The DEX has three classes: `MainActivity`,
   `ChallengeActivity` and `PredictableCrypto`. We parsed it statically with Androguard and never ran
   the app.
   - `PredictableCrypto.describe()` only returns a descriptive string. There is no nonce, key
     derivation, SharedPreferences, SQLite, Keystore or backup access anywhere.
   - `ChallengeActivity.onCreate` reads `answer` and `case_id` from Intent extras, overrides both
     with URI query parameters when URI data is present, URL-encodes them, and loads a local evidence
     route in a WebView with the answer as `proof=` next to a hard-coded `client_secret`. It derives
     nothing; it only forwards what the caller passes in.
   - So the only value in the APK in the required format is the decoded asset.
4. **Submission.** The decoded string was mapped to the answer id by a short script that checks the
   APK SHA-256 against the start response, runs the library's offline asset decoder, and asserts the
   exact pattern and answer-id set:

   ```python
   assets = json.loads(subprocess.run([sys.executable, "-I", inspector, apk_path],
                                      check=True, capture_output=True, text=True).stdout)
   if len(assets) != 1 or assets[0].get("codec") != "caesar" or not assets[0].get("supported"):
       raise SystemExit("expected exactly one supported Caesar puzzle asset")
   answer = assets[0]["decoded_text"]
   if not re.fullmatch(r"ANS-[0-9A-F]{16}", answer):
       raise SystemExit("decoded puzzle asset does not match the required answer format")
   json.dump({answer_ids[0]: answer}, sys.stdout)
   ```

   Single Solve answer: `ANS-C539C482FC5EE146`. One submission, CORRECT.

## Time Trial and Mastery

- **Time Trial.** A fresh APK for a new case (`MOB-14-79FEC4C950`), same structure, same Caesar asset.
  The start-to-deadline window was 50 s. Decoded answer `ANS-FFDA049C74988184`, CORRECT on the first
  submission about 24 s after the start timestamp.
- **Mastery.** Three APKs, one per set, with a 110 s window. Each delivery has its own case, package
  and DEX, and `answer_map.py` hash-checked and decoded each one. Candidates:
  `ANS-2C1197C6BA89DA27`, `ANS-4561FFEA182C4655`, `ANS-7DE42633A78D6CAA`. All three were accepted in
  one call about 46 s after the start timestamp. Mastery deliveries beyond the first one are fetched
  with the `delivery=N` parameter, because the plain artifact call returns only the first APK.
- **Ranked reruns.** In the later ranked sweep (Time Trial twice, Mastery twice) the same codec path
  was CORRECT on the first call each time, with total pass times of about 1.3 to 3.2 s. The ranked
  Mastery bundles had five sets, not three, so the solver pairs each question with its APK by package
  name, then by the API's sha256 by delivery name, then by delivery file name and set ordinal. It
  reads the codec from each APK's own asset rather than hard-coding Caesar. This was bragging-rights
  only and not part of the first solve.

## Dead ends and gotchas

- **The prompt describes a different challenge than the artifact.** "Seeded nonces", "weak
  derivation" and "earlier storage gates" imply crypto and storage-gate analysis. The library had no
  Android technique for them and the code has no such logic. The journals record this finding; we did
  not find a different intended path and did not look for one past the DEX, so the decoy reading
  rests on that static review only.
- **URI over Intent precedence.** `ChallengeActivity` prefers URI query parameters over Intent
  extras. This does not change the answer here, but it matters for tracing which source wins if you
  ever need to follow data flow in such a build.
- **Per-set pairing in Mastery.** Deliveries do not need to arrive in set order. The per-APK hash and
  package check keeps each answer with its own case.
- No format surprises: the decoded value was accepted verbatim, with no padding, case or prefix
  adjustment. No submissions were rejected.

## Takeaways

- Technique name: **declared puzzle-asset codec** (`huntress.mobile.puzzle.v1`, `caesar`). On this
  lane, check `assets/huntress_puzzle.json` before the DEX, and use the DEX to confirm that the
  decoded value is what the app forwards as `proof=`. On this day the title and prompt (crypto
  derivation) were flavor text.
- Verify that the pattern-matching candidate is the only one, by looking for any competing derivation
  in the code, before accepting it. Here the static review was cheap (three classes) and conclusive.
- Faster next time: skip straight to the library's asset decoder and the format check, then spend any
  remaining time on the DEX only if the decoded value fails the pattern.

## Evidence

Base path: `challenges/day14/mobile-android/mobile-android-day-14-predictable-crypto/`

- `single-solve/journal.md`, `single-solve/start_response.json`, `single-solve/submission-01.json`,
  `single-solve/artifacts/answer_map.py`
- `time-trial/journal.md`, `time-trial/start_response.json`, `time-trial/submission-01.json`,
  `time-trial/ranked-1/result.json`, `time-trial/ranked-2/result.json`
- `mastery/journal.md`, `mastery/start_response.json`, `mastery/submission-01.json`,
  `mastery/artifacts/answer_map.py`, `mastery/ranked-1/result.json`, `mastery/ranked-2/result.json`
- `tools/ranked/solvers/mobile-android-day-14-predictable-crypto/solve.py` (general ranked solver)
- Library: asset codecs (`caesar` entry) and the Android static puzzle-assets method
- `progress.md` (grep `^| 14 .*predictable-crypto`)
