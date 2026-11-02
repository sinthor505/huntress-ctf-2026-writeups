# Encrypted Strings

| | |
|---|---|
| Day / Lane | 18 / Mobile Android |
| Challenge ID | `mobile-android-day-18-encrypted-strings` |
| Technique | static APK triage; puzzle-asset codec decode (XOR over Base64); title is a decoy |
| Single Solve | Solved (journal-recorded) |
| Time Trial | Independent (original attempt journal-recorded; two later ranked reruns have `result.json` CORRECT) |
| Mastery | Independent (original attempt journal-recorded; two later ranked reruns have `result.json` CORRECT) |

## Summary

Despite the title and the prompt ("recover only the strings that reach security-sensitive Android and
backend APIs"), nothing in the APK needed string decryption. The answer code sits in
`assets/huntress_puzzle.json`, wrapped in a trivial per-day codec (here XOR with a one-byte key over
Base64), and the tiny `classes.dex` only forwards it to a local backend as a decoy `proof=` parameter.
All three modes went CORRECT on the first submission.

## The challenge

Single Solve and Time Trial each delivered one small APK (about 12.7 KB, delivered as a `.bin`) and one
question, `answer`, formatted `ANS-` followed by exactly 16 uppercase hex characters, `exact`
normalization. The question text names a case ID (`MOB-18-...`) and a package
(`com.huntress.mobile.d18.v...`). Mastery delivered five APKs and five questions
(`set1.answer` .. `set5.answer`), one per delivery, with no deadline reported in the start response.

## Solution

1. **Recon.** Downloaded the APK and checked its SHA-256 against the value in the start response.
2. **Triage.** The APK holds a manifest, `assets/huntress_puzzle.json` and a 2.6 KB `classes.dex`. Strings in
   the DEX show an `EncryptedStrings` activity that builds a request to a local evidence URL with a
   `case_id`, an `answer` extra, and `client_secret=SECRET-...&proof=`. There is no decryptor for
   any DEX string, so the "encrypted strings" in the title do not exist in the Single Solve build.
3. **The asset is the answer.** The JSON names `codec: "xor"` with an integer `key` (20 in the Single Solve
   build) and a Base64 `data` field. Decoding is two lines, and the result is checked against the answer
   pattern before it is used:

   ```python
   code = bytes(b ^ int(p["key"]) for b in base64.b64decode(p["data"])).decode("ascii")
   assert re.fullmatch(r"ANS-[0-9A-F]{16}", code)
   ```

   Single Solve decoded to `ANS-2A3476CE18776C3C`.
4. **Submission.** The decoded value went in verbatim as `answer`. CORRECT on attempt 1 of 20. This is the
   same "asset forwarded as proof" pattern as the neighbouring mobile days.

## Time Trial and Mastery

- **Time Trial** was a fresh APK of the same shape (new case ID, package suffix and `data`/`key`). The Single
  Solve mapper ran unchanged and the answer was CORRECT on attempt 1. Newer builds from Day 16 on add a
  theme method that Single Solve lacks; the Time Trial/Mastery build carries `decryptByte(II)I`, which the
  deep review later confirmed has no caller (dead code), so the asset still decides the answer.
- **Mastery** delivered five APKs that were earlier-day specimens, not five Day 18 builds. The question text
  gives the case ID: `MOB-03-`, `MOB-07-`, `MOB-12-`, `MOB-13-` and `MOB-18-`. Each used a different codec:
  Base64, Caesar (subtract `key`, 5 here), hex, Base64, and XOR (key 233) over Base64, decoding to
  `ANS-` plus 16 hex in every case. The mapper dispatches on the asset's `codec`, and asserts that each
  question's artifact filename matches the delivery it pairs with. All five were accepted in one call
  (attempt 1 of 20).
- **Fetching the bundle.** The standard artifact helper downloaded only the first delivery. The other four
  came from the `artifacts[]` list in the start response, each with its own SHA-256 to verify.
- **Later ranked reruns.** After the original solves, two ranked Time Trial and two ranked Mastery reruns
  (50 s and 170 s windows) each went CORRECT in one submission call. The whole pass (issue, download,
  solve, submit) took roughly 1.2 to 3.3 s each, using a solver that reads the codec from each APK's own asset.
  These are speed-scoreboard attempts and are not what the Single Solve/Time Trial/Mastery provenance above rests on.

## Dead ends and gotchas

- **The title and prompt mislead.** "Encrypted Strings" and "security-sensitive APIs" suggest decompiling the
  DEX and undoing string encryption. In the Single Solve build the DEX contains no decryption at all, and in the
  timed builds the only decryption-looking method is dead code. The `client_secret` in the DEX is a decoy;
  the real value is only in the asset.
- **Per-delivery codec in Mastery.** A mapper that hard-codes the Single Solve XOR key fails four of five
  sets. The codec and key must be read from each asset, and the delivery order matched against the
  questions.
- No format surprises: the decoded `ANS-` value was submitted unmodified in upper case.

## Takeaways

Triage the APK for `assets/huntress_puzzle.json` before reverse engineering the DEX; on this lane the
asset-codec decode, not code analysis, was the intended path for the earlier days. Verify the decoded value
against the stated answer pattern, and check whether a theme method (`decryptByte`) has any caller before
trusting that it matters. In a Mastery bundle, read the whole `artifacts[]` list and dispatch per delivery.

## Evidence

Leaf base path: `challenges/day18/mobile-android/mobile-android-day-18-encrypted-strings/`

- `single-solve/journal.md`, `single-solve/start_response.json`, `single-solve/artifacts/answer_map.py`
- `time-trial/journal.md`, `time-trial/start_response.json`,
  `time-trial/ranked-1/result.json`, `time-trial/ranked-2/result.json`
- `mastery/journal.md`, `mastery/start_response.json`, `mastery/artifacts/answer_map.py`,
  `mastery/artifacts/download_deliveries.py`,
  `mastery/ranked-1/result.json`, `mastery/ranked-2/result.json`
- Library entry MOBILE-ANDROID-D18-ENCSTR-001 and the consolidated asset-codecs tool (MOBILE-ANDROID-CODECS-001)
- `progress.md` (grep `^| 18 .*encrypted-strings`)
