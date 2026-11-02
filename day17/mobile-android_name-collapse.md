# Name Collapse

| | |
|---|---|
| Day / Lane | 17 / Mobile Android |
| Challenge ID | `mobile-android-day-17-name-collapse` |
| Technique | static asset-codec decode (new `indexed` permutation codec); obfuscated DEX is a decoy |
| Single Solve | Solved (journal-recorded; CORRECT on attempt 1, no result file saved) |
| Time Trial | Independent (journal-recorded CORRECT, attempt 1; later ranked reruns CORRECT) |
| Mastery | Independent (journal-recorded CORRECT, attempt 1; later ranked reruns CORRECT) |

## Summary

"Name Collapse" is dressed as a de-obfuscation task: the APK's DEX was run through identifier
shrinking (single-letter class and method names) and padded with decoy code. The answer never
depended on any of it. It sat in `assets/huntress_puzzle.json`, encoded with a codec this lane had
not used before, `indexed`, a plain permutation of a 20-character string. All three modes went
CORRECT on the first submission.

## The challenge

Prompt: "Restore behavior after deterministic identifier shrinking and decoy insertion." Each
question named a case (`MOB-17-...`) and a package (`com.huntress.mobile.d17.v...`) and asked to
"submit only the exact backend answer code returned for this case". The answer format was a
pattern: `ANS-` followed by exactly 16 uppercase hex characters.

Delivery was a 12,704-byte APK per answer (the download had a `.bin` name; it is a ZIP/APK, and
the SHA-256 in the start response matched). Single Solve and Time Trial had one question and one
APK. Mastery had five questions (`set1.answer` ... `set5.answer`) and five APKs.

## Solution

1. **Recon.** Unzipped the APK and checked the SHA-256 against the start response. The DEX was
   tiny and R8-style (a class named `a`, one-letter members). In the Time Trial and Mastery builds
   `a.a`/`a.b`/`a.c` replace the app's `describe()`, and nothing calls `a.b`: dead decoy code. The
   journal's reading of the real code path is that the DEX only forwards a value to a local backend
   as `proof=`, next to an unrelated `client_secret`.
2. **Find the data.** `assets/huntress_puzzle.json` held a codec descriptor instead of a stored
   answer:

   ```json
   {"codec": "indexed", "data": "<20 chars>", "order": [<20 ints>], ...}
   ```

3. **The one real decision: direction.** `order[i]` is the output slot of `data[i]`, the same
   convention as the lane's earlier `split`/`order` codec. The inverse direction was the obvious
   alternative, so we tested both against the answer pattern. Only one produced a string matching
   `ANS-[0-9A-F]{16}`:

   ```python
   out = [None] * len(a["data"])
   for i, pos in enumerate(a["order"]):
       out[pos] = a["data"][i]
   value = "".join(out)
   assert re.fullmatch(r"ANS-[0-9A-F]{16}", value)
   ```

4. **Submit.** The mapper is offline (stdlib only); submission was a separate step. The Single
   Solve answer was `ANS-E26A5C21D3110FA1`, accepted on call 1.

## Time Trial and Mastery

- **Time Trial:** a fresh APK of the same size with the same `indexed` codec; the same mapper was
  run unchanged and went CORRECT on call 1.
- **Mastery:** five APKs, one per earlier day's codec, in question order. The question case IDs
  gave the day: `MOB-04` (xor), `MOB-08` (reverse), `MOB-12` (hex), `MOB-14` (caesar) and `MOB-17`
  (indexed). So the mapper needed every codec the lane had used up to that point, selected from
  each APK's own descriptor, not from the challenge day. The five answers went in one call, CORRECT.
- `gm.py artifact` only fetches the first delivery; the other four had to be downloaded from the
  start response's `artifacts[]` list and hash-checked individually.
- Later ranked reruns (50 s Time Trial window, 170 s Mastery window) used a generic per-APK solver
  that reads the codec from the asset; each went CORRECT on its first submission (Time Trial
  total 1.5 s in the second ranked run, Mastery 1.7 s). Those were speed-run attempts and are not
  what this page's solution timing refers to.

## Dead ends and gotchas

- **The title is a decoy, not a method.** "Identifier shrinking and decoy insertion" describes the
  DEX, which has no bearing on the answer. We did not decompile for the value. The journal's note
  from a later review: nothing calls the decoy method, and all seven IDs we checked (1 + 1 + 5)
  reproduced from the asset alone.
- **Direction ambiguity in the permutation.** Only the pattern check distinguishes `out[order[i]] =
  data[i]` from its inverse; without the `ANS-[0-9A-F]{16}` format you could not tell them apart
  from the data alone.
- **The locked Mastery response was not the real one.** Before the Time Trial unlocked it, the
  pre-unlock Mastery response listed different case IDs than the unlocked one, so the Mastery
  answers had to be derived from a fresh `start` after unlock.
- Local APK copies from the lane can be corrupted by CRLF translation; check hash and `PK` magic
  first (a general note for this event, not something that hit this challenge).

## Takeaways

Technique to search for: **APK asset puzzle codec** (`assets/huntress_puzzle.json`, codec
`indexed`). In this lane's puzzle-asset family, read the asset's `codec` and data first; the DEX
(however mangled) is just the forwarder. When a codec has an index/order array, try both
directions and let the answer's declared format arbitrate. In a Mastery bundle, pick the decoder
per APK from its descriptor and fetch every delivery, not only the first.

## Evidence

Leaf: `challenges/day17/mobile-android/mobile-android-day-17-name-collapse/`

- `single-solve/journal.md`, `single-solve/start_response.json`, `single-solve/artifacts/answer_map.py`
  (the decoder used for all three modes), `single-solve/artifacts/submission-attempt-log.json`
- `time-trial/journal.md`, `time-trial/ranked-1/result.json`, `time-trial/ranked-2/result.json`
- `mastery/journal.md`, `mastery/start_response.json`, `mastery/artifacts/fetch_deliveries.py`,
  `mastery/ranked-1/result.json`, `mastery/ranked-recover/result.json`
- `library/mobile-android/d17-name-collapse-indexed.md` (MOBILE-ANDROID-D17-INDEXED-001)
- `progress.md` (grep `^| 17 .*name-collapse`)

Provenance note: the original untimed Single Solve, Time Trial and Mastery solves are recorded by
the journals and `progress.md` rows (CORRECT on call 1 of the submission log); no `result.json`
from those first attempts was kept. The Time Trial and Mastery `ranked-N/result.json` files are
our own CORRECT results from the later ranked reruns.
