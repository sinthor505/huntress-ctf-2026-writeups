# WebView Bridge

| | |
|---|---|
| Day / Lane | 7 / Mobile Android |
| Challenge ID | `mobile-android-day-07-webview-bridge` |
| Technique | Static APK triage; declared Caesar (ordinal shift) asset codec; verifying a claimed WebView bridge in DEX |
| Single Solve | Solved (journal-recorded): the journal records our own `CORRECT`; no result file was saved |
| Time Trial | Independent (`CORRECT` in the first-pass response and again in a ranked rerun) |
| Mastery | Independent (`CORRECT` in the first-pass response and again in a ranked rerun) |

## Summary

The prompt talks about crossing a WebView JavaScript bridge from an untrusted origin, but the APK
never registers a JavaScript interface. The answer is stored in a small JSON asset, shifted by a
Caesar key the asset declares itself. Subtracting the key from every character gave the
`ANS-<16 hex>` code in all three modes, with one submission each.

## The challenge

Prompt: "Cross a challenge-owned WebView bridge through an untrusted origin." Each question names a
case (`MOB-07-...`) and an Android package (`com.huntress.mobile.d07.v<hex>`) and asks for "the exact
backend answer code returned for this case". The answer format is a pattern: `ANS-` plus exactly 16
uppercase hex characters, normalization `exact`.

- Single Solve: one APK (`webview_bridge.apk`), answer ID `answer`.
- Time Trial: one fresh APK, answer ID `answer`.
- Mastery: three APKs delivered together, answer IDs `set1.answer`, `set2.answer`, `set3.answer`.

Each APK is tiny: `AndroidManifest.xml`, `assets/huntress_puzzle.json` (about 100 bytes) and a
`classes.dex` of about 2.6 KB.

## Solution

1. **Check the bytes.** We verified each APK's SHA-256 against the start response and confirmed the
   ZIP magic.
2. **Read the manifest.** `aapt dump xmltree` showed `.ChallengeActivity` exported and browsable for
   `huntress07://webview-bridge/unlock`, with manifest metadata that describes the mechanic as
   "JavaScript interface origin trust". That matches the prompt's wording and nothing else.
3. **Read the DEX.** Androguard (static only, nothing was installed or run) showed that
   `ChallengeActivity.onCreate` reads `answer` and `case_id` from the Intent extras, replaces them
   with the URI query parameters if a data URI is present, URL-encodes them into a local evidence URL
   as `proof` and `case_id`, and calls `WebView.loadUrl`. `WebViewBridge` has only a `describe()`
   method. The DEX contains no `addJavascriptInterface` and no `setJavaScriptEnabled` call, so the
   "bridge" in the title is flavor text. There is no live exploit to run.
4. **Decode the asset.** `assets/huntress_puzzle.json` carries schema `huntress.mobile.puzzle.v1`,
   `day: 7`, `codec: "caesar"`, a numeric `key` and `data`. The codec is an ordinal shift with no
   wraparound, applied to every character:

   ```python
   shift = int(puzzle["key"])
   decoded = "".join(chr(ord(ch) - shift) for ch in puzzle["data"])
   ```

   With key 10 the Single Solve asset decoded to `ANS-8AFEA54099F3106E`, which matches the required
   pattern. The mapper (`artifacts/answer_map.py`) also asserts that the start response's
   `answer_ids` is exactly `["answer"]` and that the asset schema and day are the expected ones, then
   prints `{"answer": "ANS-8AFEA54099F3106E"}`.
5. **Submit.** One call, `CORRECT`.

## Time Trial and Mastery

Same mechanism, different key and data per APK.

- Time Trial: key 18, decoded to `ANS-EF4F4D6A583828D4`.
- Mastery: three APKs, keys 8, 5 and 7, decoded to `ANS-16253A1B7808CAEB`, `ANS-6BBBC3AE4F0C841E` and
  `ANS-8A0B92D866C90B33`.

Mastery has no per-set question text beyond the case and package, so the mapping from APK to
`setN.answer` has to be right. The journal mapped each APK to its question by the package named in
the question text (`in package ...`) and the delivery number, not by guessing from file order. The
reusable ranked solver (`tools/ranked/solvers/mobile-android-day-07-webview-bridge/solve.py`) does the
same, preferring the manifest package, then the API hash, then file name, then `setN` ordinal. It
reads the codec from each APK's own asset instead of assuming Caesar, uses the shared
`asset_codecs.py` library, and handles a single APK, a bundle ZIP, or a directory.

The first-pass Time Trial and Mastery responses had `deadline: null` (untimed), so the first solves
were not time-pressured. Later ranked reruns went `CORRECT` in one submission each: Time Trial in a
50 s window with a total of 1.09 s, Mastery in a 110 s window with a total of 1.66 s (decode itself
took about 0.01 s; issue, download and submit dominate).

## Dead ends and gotchas

- **The title is bait.** The prompt, title and manifest metadata all point at a JavaScript-interface
  origin check. We confirmed in the DEX that nothing is exposed, so no origin-trust analysis is
  needed. Do not infer a bridge from wording or metadata; grep for the actual
  `addJavascriptInterface` call.
- **The key varies per APK.** Keys were 10, 18, 8, 5 and 7. Always read it from the asset.
- **No wraparound.** The Caesar shift here is a plain `ord(c) - key` over code points, not an
  alphabet rotation. The encoded strings contain punctuation (for example `[`, `>`, `?`) that a
  letters-only rotation would not produce.
- **The library helper did not yet support the codec.** At the time, the existing asset inspector
  recognized only Base64, XOR/Base64 and position-indexed fragments. The Caesar branch was added to it
  after the Single Solve.
- No wrong answers were submitted in any mode; there were no rejected readings to record.

## Takeaways

- For Android puzzle APKs, the answer lives in `assets/huntress_puzzle.json`; trust the declared
  `codec` and `key`, then check the result against the stated answer pattern.
- Verify a claimed WebView/JS bridge by searching the DEX for `addJavascriptInterface` and
  `setJavaScriptEnabled` before spending time on an origin-trust story.
- In Mastery, pair each APK with its question by package name, not by file order.
- Faster next time: skip the manifest and DEX reading entirely and decode the asset first. The
  format check on the output (`ANS-` plus 16 hex) is enough to confirm the codec choice.

## Evidence

Base path: `challenges/day07/mobile-android/mobile-android-day-07-webview-bridge/`

- `single-solve/journal.md`, `single-solve/artifacts/answer_map.py`, `single-solve/start-response.json`
  (no `result.json`; the journal records our own submission and `CORRECT`)
- `time-trial/journal.md`, `time-trial/submission-response.json`, `time-trial/ranked-1/result.json`
- `mastery/journal.md`, `mastery/submission-response.json`, `mastery/ranked-1/result.json`
- Ranked solver: `tools/ranked/solvers/mobile-android-day-07-webview-bridge/solve.py`
- Library entry: `MOBILE-ANDROID-STATIC-001` (puzzle-asset inspection and WebView bridge gotcha)
