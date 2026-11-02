# Trust Everyone

| | |
|---|---|
| Day / Lane | 9 / Mobile Android |
| Challenge ID | `mobile-android-day-09-trust-everyone` |
| Technique | Static read of a declared Base64 asset codec; the TLS-trust theme is decoration |
| Single Solve | Independent (journal-recorded CORRECT; submission response saved) |
| Time Trial | Independent (original attempt: saved CORRECT response; three later ranked reruns: `ranked-N/result.json`) |
| Mastery | Independent (original attempt: saved CORRECT response; one ranked rerun: `ranked-1/result.json`) |

## Summary

The prompt talks about "a challenge CA accepted by a permissive trust implementation", which suggests
a TLS trust-manager bug. The answer never depended on it. Each APK carries its backend answer code in
`assets/huntress_puzzle.json`, encoded with a codec the file declares (Base64 on this day), and we
decoded it offline without installing or running the app. All three modes went CORRECT on the first
submission.

## The challenge

The prompt reads "Exploit a challenge CA accepted by a permissive trust implementation." The
question asks for "the exact backend answer code returned for this case", for a named case id and
Android package. The answer format is a pattern, `ANS-` plus exactly 16 uppercase hex characters,
with exact normalization.

- Single Solve: one APK (`trust_everyone.apk`, 12,704 bytes), one answer field `answer`.
- Time Trial: one different APK, one field `answer`. Our original timed window was 50 s.
- Mastery: five APKs (also 12,704 bytes each) with fields `set1.answer` to `set5.answer`. The original
  window was 170 s.

Every APK is a different build with its own case id and package name (`com.huntress.mobile.d09.v...`).

## Solution

1. **Recon.** We checked the APK's SHA-256 against the start response. Each APK has the same six
   members: binary manifest, `assets/huntress_puzzle.json`, `classes.dex` (about 2.5 KB) and three
   signing entries. The manifest declares `huntress.mechanic=permissive X509 trust manager`, an exported
   `ChallengeActivity` with a browsable `huntress09://trust-everyone/unlock` filter, and
   `INTERNET` permission. The DEX contains the strings `TrustEveryone` and `trust_manager`.
2. **Where the answer is.** The 101-byte asset is a JSON object with four keys: `schema`
   (`huntress.mobile.puzzle.v1`), `day` (9), `codec` (`base64`) and `data`. The answer's
   shape (`ANS-` plus 16 hex) is what a 20-byte decoded payload looks like, which fits the format in
   the question.
3. **Decode.** The library's offline asset inspector reads the declared codec and decodes `data`.
   Each APK decoded to exactly 20 UTF-8 bytes that matched `ANS-[0-9A-F]{16}`.
4. **Map and submit.** A tiny mapper (`answer_map.py`) asserts one supported Base64 asset, checks
   the pattern, and prints `{"answer": "..."}` as JSON. It has no network or submission code.
   Submission was a separate step, one call.

Single Solve answer: `ANS-3D3B80395240B8EE`.

## Time Trial and Mastery

Nothing about the technique changed. Each mode delivers fresh APKs with different case ids, so the
decoded values differ and nothing carries over from Single Solve (the journals say each answer was
mapped from that mode's own APKs).

- Time Trial: one APK, decoded and submitted well inside the 50 s window (the original attempt was
  accepted about 16 s after the start).
- Mastery: five APKs. The mapper read `..._0N-asset-facts.json` by question index, which was correct
  here but is fragile in general; later work pairs each question to its APK by package name or hash
  instead. All five fields were submitted in one request and accepted.

Later ranked reruns (the one-attempt ranked sweep of older days, plus further reruns) used a general
per-challenge solver. It reads the codec from each APK's own asset, so a different codec still decodes,
and it pairs questions to APKs by the `in package X` text in the question. Timed passes took about
1.4 to 2.1 s end to end, with three Time Trial reruns and one Mastery rerun all CORRECT on the first
call.

## Dead ends and gotchas

- There were no real dead ends. The `jadx`/`apktool` tools were not available in the original
  session, so the DEX was only searched for strings; it contains no `TrustManager` or
  `HostnameVerifier` code at all. The "permissive trust" theme is flavor text, and the lane's playbook
  notes the same for this fixture.
- Answer values must be JSON strings; the mapper emits them that way.
- The codec is fixed per day (asset `day` field). Day 9 is Base64, so this mapper would not have
  worked unchanged on a different day's codec.

## Takeaways

- On this lane the asset JSON (`assets/huntress_puzzle.json`) is the first and usually only place to
  look. Read its `codec` before reading any bytecode, and treat the title and prompt as theme.
- Search for the technique as "static asset codec decode", `huntress.mobile.puzzle.v1`. The library
  entries `MOBILE-ANDROID-STATIC-001` and the asset-codecs tool cover it.
- Pair Mastery questions to APKs by a stable key (package name or hash), not by position.

## Evidence

Leaf base path: `challenges/day09/mobile-android/mobile-android-day-09-trust-everyone/`

- `single-solve/journal.md`, `single-solve/artifacts/answer_map.py`, `single-solve/artifacts/asset-facts.json`,
  `single-solve/submission-01-response.json` (CORRECT)
- `time-trial/journal.md`, `time-trial/artifacts/answer_map.py`, `time-trial/submission-01-response.json`
  (CORRECT), `time-trial/ranked-1..3/result.json` and `time-trial/open-1/result.json` (all CORRECT)
- `mastery/journal.md`, `mastery/artifacts/answer_map.py`, `mastery/submission-01-response.json`
  (CORRECT), `mastery/ranked-1/result.json` (CORRECT)
- Ranked solver (outside the leaf): `tools/ranked/solvers/mobile-android-day-09-trust-everyone/solve.py`
- `progress.md` rows for day 9 record all three modes as Solved with one submission call each.
