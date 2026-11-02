# Reflection Maze

| | |
|---|---|
| Day / Lane | 19 / Mobile Android |
| Challenge ID | `mobile-android-day-19-reflection-maze` |
| Technique | APK puzzle asset with a per-day codec; the DEX only forwards the answer (decoy `client_secret`) |
| Single Solve | Solved (journal-recorded) |
| Time Trial | Independent (untimed first solve journal-recorded; two later ranked reruns have `result.json` CORRECT) |
| Mastery | Independent (untimed first solve journal-recorded; one later ranked rerun has `result.json` CORRECT) |

## Summary

"Reflection Maze" promises a reflective-dispatch call graph to untangle. In every build we saw, no
reflection exists at runtime: the DEX is a thin WebView launcher and the answer code sits in the APK's
`assets/huntress_puzzle.json`, stored under a codec chosen per puzzle. Reading the asset and applying
its declared codec gave the answer; all three modes were accepted on the first submission.

## The challenge

Prompt: "Reconstruct reflective dispatch across generated equivalent call graphs." Each question names a
case (`MOB-19-...`) and a package (`com.huntress.mobile.d19.v...`) and says "Submit only the exact backend
answer code returned for this case." Answer format is `ANS-` plus 16 uppercase hex digits.

- Single Solve: one 12,704-byte APK, one answer (`answer`).
- Time Trial: a fresh APK of the same shape (new case id), one answer.
- Mastery: five APKs (`artifacts[]` in the start response, `set1.answer` ... `set5.answer`), each
  themed on an earlier day's puzzle (cases `MOB-03`, and others for days 5, 11, 18, 19). Mastery stayed
  locked (409 `prerequisite_required`) until the Time Trial was CORRECT.

## Solution

1. **Verify and unpack.** Each APK was checked against the SHA-256 in the start response. It contains
   a manifest, `classes.dex` and `assets/huntress_puzzle.json`.
2. **Read the DEX.** `ChallengeActivity` takes an `answer` and `case_id` from the intent or URI and
   loads `http://127.0.0.1:19090/mobile/attempt/<id>/evidence?case_id=...&client_secret=SECRET-...&proof=<answer>`
   in a WebView. `ReflectionMaze.describe()` only returns a descriptive string. There is no reflection
   API and no answer decoding in the DEX. The `SECRET-` value is a decoy credential, not an answer, and
   the localhost backend was not reachable or in scope, so we never contacted it.
3. **Key insight.** The answer is the asset's `data`, decoded by the codec the asset declares. For the
   Single Solve and Time Trial the codec is `reflection`, which is just the plain `ANS-` code:

   ```python
   a = json.loads(z.read('assets/huntress_puzzle.json'))
   assert a['codec'] == 'reflection'
   v = a['data']
   assert re.fullmatch(r'ANS-[0-9A-F]{16}', v)
   print(json.dumps({'answer': v}))
   ```

4. **Submit.** The decoded code was submitted as the single answer string; the Game Master returned
   CORRECT on attempt 1. The Single Solve answer was `ANS-DFCDF6E9F6ABE632`.

## Time Trial and Mastery

Time Trial was the same pipeline on a fresh APK (new case id and secret) and was CORRECT on the first
submission. The journal notes mode APKs differ, so answers are never reusable across modes.

Mastery's five APKs carried the earlier days' codecs, so the mapper had to be a small codec dispatcher
that keys each answer by its delivery number:

- `base64`: base64-decode `data`.
- `xor`: base64-decode, then XOR every byte with the one-byte `key` (seen on the Day 11 and Day 18 themed sets).
- `split`: join `parts`, where `order[i]` is the output position of `parts[i]`.
- `reflection`: identity (the Day 19 set).

```python
if c == 'split':
    out = [None] * len(a['parts'])
    for i, pos in enumerate(a['order']):
        out[pos] = a['parts'][i]
    return ''.join(out)
```

`gm.py artifact` only fetches the first delivery, so the other four were fetched by capability from
`artifacts[]` and each hash-checked. All five fields were accepted in one submission.

Later ranked reruns (timed, with a 50 s Time Trial window and a 170 s Mastery window) went CORRECT in one
call each using a generic APK solver that reads the codec from the asset; the Time Trial was rerun twice.
The Mastery rerun ran through the same solver.

## Dead ends and gotchas

- **The title is misleading.** Nothing reflective happens at runtime. A deep review on 2026-10-24
  confirmed that in the Time Trial and Mastery builds `dispatch(I)I` has no caller and no reflection API
  appears anywhere; all seven IDs reproduced from the asset alone.
- **`client_secret` is a decoy.** It sits in the WebView URL next to the real value, but only the asset
  `data` is graded.
- **Tooling gap.** The lane's APK inspector reported the `reflection` codec as unsupported and handles
  one APK at a time, so a separate mapper was written for this challenge (since folded into the lane's
  general codec decoder).
- No format surprises: `ANS-` plus 16 uppercase hex was accepted as decoded in all modes.

## Takeaways

Technique: **puzzle-asset codec decoding** in a decoy-heavy APK. When a mobile prompt talks about
dynamic behaviour but the DEX is tiny, check the assets first, validate the decoded value against the
stated answer regex, and treat any secret-looking string passed in a URL as a decoy. In a Mastery bundle,
dispatch on each APK's own `codec` field, key results by delivery number, and hash-check every delivery.
Next time we would go straight to a generic codec reader over all `artifacts[]` instead of per-day mappers.

## Evidence

Leaf base: `challenges/day19/mobile-android/mobile-android-day-19-reflection-maze/`

- `single-solve/journal.md`, `single-solve/start_response.json`, `single-solve/artifacts/answer_map.py`, `single-solve/artifacts/submission-attempt-log.json`
- `time-trial/journal.md`, `time-trial/artifacts/answer_map.py`, `time-trial/ranked-1/result.json`, `time-trial/ranked-2/result.json`
- `mastery/journal.md`, `mastery/start_response.json`, `mastery/artifacts/answer_map.py`, `mastery/ranked-1/result.json`
- `library/mobile-android/d19-reflection-maze.md` (MOBILE-ANDROID-REFLECTION-001)
- `tools/ranked/solvers/mobile-android-day-19-reflection-maze/solve.py`
- `progress.md` (grep `^| 19 .*reflection-maze`)
