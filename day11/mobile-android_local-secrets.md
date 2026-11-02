# Local Secrets

| | |
|---|---|
| Day / Lane | 11 / Mobile Android |
| Challenge ID | `mobile-android-day-11-local-secrets` |
| Technique | static APK analysis; declared Base64 + byte-XOR puzzle asset (`huntress.mobile.puzzle.v1`) |
| Single Solve | Solved (journal-recorded) |
| Time Trial | Independent |
| Mastery | Independent |

## Summary

"Local Secrets" ships a tiny APK full of credential-looking strings meant to distract. The backend
answer code is not any of them: it sits in `assets/huntress_puzzle.json`, encoded with the
day's declared codec (Base64, then XOR with an integer key). Decoding that asset offline, without
installing or running the app, gave the answer in all three modes. Each mode was accepted on the
first submission.

## The challenge

Prompt: "Separate active credentials from decoys across generated local stores." The question
named a case (`MOB-11-...`) and a package (`com.huntress.mobile.d11.v...`) and asked for "the exact
backend answer code returned for this case". Answer format: `pattern`, `ANS-` followed by exactly
16 uppercase hex characters, `exact` normalization.

- **Single Solve:** one 12,704-byte APK (`local_secrets.apk`), one answer id (`answer`), no deadline.
- **Time Trial:** one fresh APK, one answer id, a window of about 50 s.
- **Mastery:** three APK deliveries (`local_secrets_mastery_01..03.apk`) under one timer of about
  110 s, answer ids `set1.answer`, `set2.answer`, `set3.answer`.

## Solution

1. **Triage.** SHA-256 of the download matched the one in the start response. The APK is six ZIP
   members: `AndroidManifest.xml`, `classes.dex`, `assets/huntress_puzzle.json` and three
   `META-INF` signature files.
2. **The asset.** The JSON declares its own decoder:

   ```json
   {"schema":"huntress.mobile.puzzle.v1","day":11,"codec":"xor","data":"EB8CfGkXY2NkZmVmYBUXZWkVYGU=","key":81}
   ```

   Standard Base64 decode, then XOR every byte with `key` (81). That yields 20 bytes,
   `ANS-8F2257471DF48D14`, which matches the required `ANS-` + 16 uppercase hex pattern exactly.
   The match against a strict format is the main confirmation that the codec reading was right.
3. **The decoys.** Disassembling `classes.dex` with Androguard (static only) showed what the
   "local stores" framing refers to:
   - `ChallengeActivity` is exported and browsable under `huntress11://local-secrets/unlock...`.
     Its `onCreate` reads `answer` and `case_id` from the Intent or URI, URL-encodes them and loads
     a local evidence URL into a `WebView`, appending a hard-coded `client_secret=SECRET-...`
     and `proof=`.
   - `LocalSecrets.describe()` returns a marker string
     (`DAY-11:shared_preferences:locally stored synthetic credential`) and has no caller.

   So the `SECRET-` constant is active but only authenticates the app to its evidence endpoint, and
   `describe()` is dead code. Neither carries the answer. The app forwards whatever `proof` it is
   given; the real value lives only in the asset. No network request was made to the evidence
   endpoint.
4. **Submission.** `ANS-8F2257471DF48D14` under answer id `answer`, one call, `CORRECT`.

## Time Trial and Mastery

Nothing about the technique changed; the codec stayed `xor` (the asset declares it, and the day
fixes it), only the fixtures did: new package names, keys and data per APK.

- **Time Trial** windows of about 50 s meant scripting download, decode and submit in one pass. The
  asset codec library read the codec from each APK's own JSON, so no per-variant code was needed.
  The first attempt took one submission. A later ranked attempt (legacy route, where a plain
  `retry` is the ranked attempt) took 1.5 s end to end and was `CORRECT` on the first call.
- **Mastery** delivered three APKs. Each decoded independently with the same codec and all three
  answers matched the `ANS-` pattern; they went in one submit call, `CORRECT`. The ranked rerun
  took 3.7 s total (mostly the bundle download) and was also `CORRECT` on the first call. The solver
  pairs each question to its APK by the `in package X` text in the question, falling back to the
  delivery name or set ordinal.

## Dead ends and gotchas

- **Credential-looking strings are not the answer.** The `SECRET-` constant and the `client_secret`
  / `proof` parameters in the DEX are plumbing for the evidence URL. Trace a constant to its use
  sites before calling it active or decoy.
- **A transient HTTP 500 on a Mastery download** (a Cloudflare worker error on delivery 01). The
  same capability URL succeeded on a second request and its SHA-256 matched the start response's
  `artifacts[]`. It is not a verdict on anything; retry the download.
- **Format.** Nothing surprising: the decoded string was already in the required format, so there
  were no spelling variants to try.

## Takeaways

The technique is **declared-codec puzzle asset**: in this lane the answer comes from the APK's
own `assets/huntress_puzzle.json` (schema, `codec`, `data`, `key`), decoded offline, and the DEX
is a source of decoys. Check the asset first, confirm the decode against the question's answer
format, and only then spend time in the bytecode. With 50 s and 110 s windows, a decoder that
reads the codec from each APK rather than hard-coding it is what makes Time Trial and Mastery a
single scripted pass.

## Evidence

Leaf: `challenges/day11/mobile-android/mobile-android-day-11-local-secrets/`

- `single-solve/journal.md`, `single-solve/start_response.json`. No `result.json` was saved for
  the first Single Solve; the journal records the `CORRECT`, and `progress.md` row 11 lists it as
  Solved.
- `time-trial/journal.md`, `time-trial/ranked-1/result.json`, `time-trial/ranked-1/pass.log`
- `mastery/journal.md`, `mastery/ranked-1/result.json`, `mastery/ranked-1/pass.log`
- `tools/ranked/solvers/mobile-android-day-11-local-secrets/solve.py` (the ranked solver; the
  codec itself comes from the asset-codecs library entry, MOBILE-ANDROID-CODECS-001)
- `progress.md` (grep `^| 11 .*local-secrets`)
