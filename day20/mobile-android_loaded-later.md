# Loaded Later

| | |
|---|---|
| Day / Lane | 20 / Mobile Android |
| Challenge ID | `mobile-android-day-20-loaded-later` |
| Technique | answer constant in a shipped-but-never-loaded secondary DEX (`dynamic` puzzle codec) |
| Single Solve | Solved (journal-recorded) |
| Time Trial | Independent (original attempt journal-recorded; two later ranked reruns have `result.json` CORRECT) |
| Mastery | Independent (same evidence pattern as Time Trial) |

## Summary

"Loaded Later" suggests an app that fetches or loads code at runtime, and the prompt asks you to
"analyze a challenge-signed secondary DEX and recover the module contract". The APK does ship a
second DEX, but nothing in the app ever loads it. The answer is the single `ANS-` string sitting
as a static constant in that DEX file, so a regex over the bytes is enough, with no execution
needed. All three modes were CORRECT on the first submission.

## The challenge

Single Solve delivered one APK (`loaded_later.apk`), Time Trial one fresh APK, and Mastery five
APKs in `artifacts[]`. Every question asks for "the exact backend answer code returned for this
case", with answer format `pattern`: `ANS-` followed by exactly 16 uppercase hex characters, one
answer id (`answer`) for Single Solve and Time Trial, five (`set1.answer` to `set5.answer`) for
Mastery. The questions name a case id and an Android package (`com.huntress.mobile.d20.v...`).
Single Solve and the first attempts of Time Trial and Mastery were untimed (`deadline: null`).

## Solution

1. **Recon.** Every APK in this lane carries `assets/huntress_puzzle.json`. Here it read
   `schema: huntress.mobile.puzzle.v1`, `codec: dynamic`, `data: assets/payload.dex`. The
   codec value was new; earlier days used inline codecs (reverse, base64, xor, plain).
2. **Look at the main DEX.** `classes.dex` builds the evidence URL
   (`http://127.0.0.1:19090/mobile/attempt/.../evidence?case_id=...&proof=`) and carries a decoy
   `client_secret=SECRET-...`. This is the usual lane layout: the app forwards the answer to a
   local backend as `proof=` beside a decoy secret. The backend is loopback-only and was never
   contacted.
3. **Look at the secondary DEX.** `assets/payload.dex` holds a `Module` class with `ANSWER` and
   `CONTRACT` string constants. Running the regex `ANS-[0-9A-F]{16}` over its raw bytes gave
   exactly one match. The same regex over `classes.dex` gave none, which is the check that this
   is the real value and not a decoy.
4. **Extractor.** The solver is a few lines of standard library code (`answer_map.py`):

   ```python
   a = json.loads(z.read("assets/huntress_puzzle.json"))
   assert a["schema"] == "huntress.mobile.puzzle.v1" and a["codec"] == "dynamic", a
   dex = z.read(a["data"])
   assert dex[:4] == b"dex\n"
   codes = set(re.findall(rb"ANS-[0-9A-F]{16}", dex))
   assert len(codes) == 1, codes
   # the main classes.dex must not carry a competing ANS- code
   assert not re.search(rb"ANS-[0-9A-F]{16}", z.read("classes.dex"))
   ```

   It also checked the APK's SHA-256 against the start response before reading it.
5. **Submission.** The one match, `ANS-ED497119EFACC9FF`, went in as the `answer` field and
   was CORRECT on attempt 1.

The journal's later review confirmed that in Time Trial and Mastery builds `moduleContract()`
has no caller, so `payload.dex` is never loaded at runtime. The "dynamic" name in the title and
codec describes the shape of the APK, not any behavior you have to execute.

## Time Trial and Mastery

- **Time Trial** was a fresh APK with the same layout and different constants. The Single
  Solve mapper ran unchanged and was CORRECT on attempt 1 (journal and attempt log).
- **Mastery** delivered five APKs. They were themed on earlier days' codecs: delivery 1 `reverse`
  (a Day 2 case id), 2 `base64` (Day 9), 3 and 4 `xor` (Days 11 and 18, a one-byte key over
  base64), 5 `dynamic` (Day 20). Only delivery 5 matched the title. The Mastery `answer_map.py`
  dispatches on the asset's `codec` field, matches each delivery to its question by filename and
  SHA-256 (`delivery N` maps to `setN.answer`), asserts every value fits the `ANS-` pattern, and
  asserts the key set equals `answer_ids`. One submission of five fields was CORRECT.
- `gm.py artifact` fetches only the first Mastery delivery, so a small helper downloaded all five
  from `artifacts[]`.
- Later we reran both modes as timed ranked attempts (a 50 s Time Trial window and a 170 s
  Mastery window). Each `result.json` is CORRECT on the first submission, with about 1.3 s and
  1.7 s total in the latest pass. These are speed-board runs, not the original solves.

## Dead ends and gotchas

- The `client_secret=SECRET-...` string in `classes.dex` and the evidence URL are bait. The
  answer is not there (the secret is a decoy and the `127.0.0.1` backend was never contacted).
- Mastery mixes codec families from earlier days, so a decoder that only handles `dynamic`
  fails four of five sets. Dispatching on each asset's `codec` field is the fix.
- Reading questions against deliveries by position is fragile. A first codec script zipped
  questions with deliveries by index and was later marked superseded; the final mapper matches
  by filename and hash.
- Our attempt log shows one submission call per mode for the originals, and no rejected
  answers.
- Separate from this challenge's own traps: a later Day 24 Mastery drew a Day 20 build whose
  `payload.dex` was actually loaded by an `InMemoryDexClassLoader` (a different, v2 `analysis`
  codec). That needed a Dalvik VM, not a regex. Do not assume a `payload.dex` is dead without
  checking for a loader call.

## Takeaways

- Technique name: **static constant in an unloaded secondary DEX** (puzzle codec `dynamic`).
  Before reaching for a Dalvik interpreter or a debugger, regex the declared `data` asset and
  confirm no competing candidate exists in the main DEX.
- Check whether the main DEX references any class-loading API (`DexClassLoader`,
  `InMemoryDexClassLoader`). If it does not, the secondary DEX is data, not code.
- For Mastery in this lane, expect earlier-day codecs and verify each APK against its listed
  SHA-256 before decoding.
- Faster next time: a single consolidated codec tool (which the workspace later built) covers
  every codec value in one pass.

## Evidence

Leaf base path: `challenges/day20/mobile-android/mobile-android-day-20-loaded-later/`

- `single-solve/journal.md`, `single-solve/start_response.json`, `single-solve/artifacts/answer_map.py`,
  `single-solve/artifacts/submission-attempt-log.json` (one call, CORRECT)
- `time-trial/journal.md`, `time-trial/artifacts/submission-attempt-log.json`,
  `time-trial/ranked-1/result.json`, `time-trial/ranked-2/result.json`
- `mastery/journal.md`, `mastery/start_response.json`, `mastery/artifacts/answer_map.py`,
  `mastery/artifacts/fetch_deliveries.py`, `mastery/artifacts/submission-attempt-log.json`,
  `mastery/ranked-1/result.json`, `mastery/ranked-2/result.json`
- Library entry MOBILE-ANDROID-DYNAMIC-DEX-001 (`library/mobile-android/d20-loaded-later-dynamic.md`)
- `progress.md` (grep `^| 20 .*loaded-later`); Day 20 has no librarian note or lane log
  (`challenges/day20/_run/` only holds `release_list.json` and `watch.log`)
