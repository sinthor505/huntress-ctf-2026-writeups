# Network Config Split

| | |
|---|---|
| Day / Lane | 10 / Mobile Android |
| Challenge ID | `mobile-android-day-10-network-config-split` |
| Technique | Static APK asset decoding (`split` fragment codec); unreferenced Network Security Config as a decoy |
| Single Solve | Solved (journal-recorded) |
| Time Trial | Independent (ranked rerun `result.json` CORRECT; original attempt journal-recorded CORRECT) |
| Mastery | Independent (ranked rerun `result.json` CORRECT; original attempt journal-recorded CORRECT) |

## Summary

The prompt talks about diffing the manifest against the Network Security Config to find "the true transport exception", but the answer never depends on that. The code is stored in `assets/huntress_puzzle.json` as three shuffled fragments with an ordering array. Reassembling them gave the answer in all three modes, one submission each.

## The challenge

Prompt: "Diff manifest and Network Security Config variants to locate the true transport exception." Each question asks for "the exact backend answer code returned for this case" for a named case ID and package (for example `com.huntress.mobile.d10.va4b83c45132b`). The format is `ANS-` plus exactly 16 uppercase hex characters, with exact normalization.

- Single Solve: one APK (`network_config_split.apk`), one answer field `answer`, no deadline.
- Time Trial: one fresh APK, one answer, a window of about 50 s.
- Mastery: three APK deliveries (`network_config_split_mastery_01.apk` to `_03.apk`), answers `set1.answer` to `set3.answer`, a window of about 110 s.

## Solution

1. Verified the APK's SHA-256 against the start response. The APK contains `AndroidManifest.xml`, `classes.dex`, `resources.arsc`, `res/xml/network_security_config.xml` and `assets/huntress_puzzle.json`.
2. Read the manifest and the config. The manifest has `usesCleartextTraffic=true` and no `android:networkSecurityConfig` attribute. The XML file permits cleartext for `127.0.0.1` and subdomains, but nothing references it, so it has no effect and the manifest flag governs. This is the "transport exception" the prompt alludes to, and it is a decoy for the answer.
3. Read the DEX with Androguard. `ChallengeActivity` takes `answer` and `case_id` from the intent or deep-link URI (`huntress10://network-config-split/unlock...`) and loads a local evidence URL in a `WebView`, passing the answer as `proof=` next to an embedded client secret. It computes nothing and does not contain the answer.
4. Decoded the puzzle asset. It has schema `huntress.mobile.puzzle.v1`, codec `split`, three `parts` and an `order` array. In the Single Solve APK the parts were `["56C4B5B0D", "ANS-4", "9D8953"]` and the order was `[2, 0, 1]`. The key detail is direction: `order[i]` is the output slot of `parts[i]`, not "the i-th piece to emit". The library codec does this:

```python
order = _permutation(a["order"], len(parts))
out = [""] * len(parts)
for i, slot in enumerate(order):
    out[slot] = parts[i]          # order[i] = output slot of parts[i]
return "".join(out)
```

   That gives `ANS-4` + `9D8953` + `56C4B5B0D` = `ANS-49D895356C4B5B0D`, which matches the required format.
5. Submitted that value under `answer` and the Game Master returned CORRECT on the first submission (1 of 10 authorized). The journal says the backend endpoint was never contacted, since the decoded value was accepted directly.

## Time Trial and Mastery

Nothing new in the mechanism. Both used the same `split` codec, read from each APK's own asset, with fresh fragments. The 50 s and 110 s windows meant the solve was scripted as download, verify hash, decode and submit in one pass.

- Time Trial: one APK, decoded directly, pattern matched, one submission, CORRECT.
- Mastery: the three deliveries come from `artifacts[]`, not a single download link. Each was hash-verified and decoded independently, and all three answers went in one call, CORRECT.
- Both modes were later re-run as ranked attempts. The ranked-pass solver (`tools/ranked/solvers/mobile-android-day-10-network-config-split/solve.py`) pairs each question to its APK by the `in package X` text in the question, with API sha256, file name, set ordinal and input order as fallbacks. Both passes were CORRECT in one call each (about 1.3 s and 2.2 s total).

## Dead ends and gotchas

- The prompt's "manifest vs Network Security Config diff" is a decoy. The XML config is not referenced by the manifest. We did not find any puzzle content in either file.
- The `DEX` forwards the answer to a backend as `proof=` beside a decoy client secret. Nothing in the static APK shows what the backend returns, and the locally decoded string was accepted without it.
- Fragment order is the only real trap: the direction of the `order` array matters. This leaf did not test the inverse reading; the library notes that for the sibling `indexed` codec the inverse failed the answer format.
- Mastery delivers several APKs, and a fetch of only the first download link would miss the other two.

## Takeaways

- For Android puzzle APKs, check `assets/huntress_puzzle.json` before the manifest analysis. The schema `huntress.mobile.puzzle.v1` and its `codec` field name the decoder, and the codec is fixed per day (Day 5 and Day 10 both use `split`).
- `order[i]` is the output slot of `parts[i]`. The same direction applies to `indexed`.
- A Network Security Config in `res/xml` does nothing unless `<application android:networkSecurityConfig>` points at it. On a real app a referenced config would override `usesCleartextTraffic` on API 24+. Here it is unreferenced, so the manifest flag governs.

## Evidence

Leaf base: `challenges/day10/mobile-android/mobile-android-day-10-network-config-split/`

- `single-solve/journal.md`, `single-solve/start_response.json` (the graded CORRECT is recorded in the journal and `progress.md`; no result file was saved)
- `time-trial/journal.md`, `time-trial/artifacts/answer_map.json`, `time-trial/ranked-1/result.json`
- `mastery/journal.md`, `mastery/artifacts/answer_map.json`, `mastery/ranked-1/result.json`
- Solver: `tools/ranked/solvers/mobile-android-day-10-network-config-split/solve.py`; codec: `library/mobile-android/asset-codecs/asset_codecs.py`
- `challenges/day10/submission-attempts.md` (Single Solve 1 submission); `progress.md` Day 10 rows
