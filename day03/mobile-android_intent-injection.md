# Intent Injection

| | |
|---|---|
| Day / Lane | 3 / Mobile Android |
| Challenge ID | `mobile-android-day-03-intent-injection` |
| Technique | Static APK triage; puzzle-asset decode (`base64` codec) behind an Intent-injection decoy |
| Single Solve | Solved (journal-recorded): the journal records our own submission as `CORRECT`; no result file was saved |
| Time Trial | Independent (ranked rerun `ranked-1/result.json` is `CORRECT`; the original 2026-10-18 solve is also in the journal) |
| Mastery | Independent (ranked rerun `ranked-1/result.json` is `CORRECT`; the original 2026-10-18 solve is also in the journal) |

## Summary

The prompt asks you to trace attacker-controlled extras across an internal navigation boundary in an
Android app. The vulnerable component is real, but the answer does not depend on exploiting it: the
backend answer code is stored base64-encoded in `assets/huntress_puzzle.json`, so decoding that one
field from the downloaded APK solves every mode. Single Solve took one submission, and Time Trial and
Mastery reused the same ~30-line script.

## The challenge

- Prompt: "Trace attacker-controlled extras through an internal navigation boundary." Each question
  names a case id (`MOB-03-<hex>`) and a package (`com.huntress.mobile.d03.v<hex>`) and asks for
  "the exact backend answer code returned for this case".
- Answer format: pattern `ANS-` plus exactly 16 uppercase hex characters, exact match, one string.
- Single Solve: one APK (`intent_injection.apk`), answer id `answer`, untimed.
- Time Trial: one APK per attempt, answer id `answer`. Listing metadata gave a 50 s ranked window.
- Mastery: three APKs (`set1.answer` ... `set3.answer`), 110 s window.
- Time Trial was locked behind Single Solve, and Mastery behind Time Trial, in the original 2026-10-18
  run (`prerequisite_required` on the first `start` of each).

## Solution

1. **Triage the APK.** The ZIP holds `AndroidManifest.xml`, `classes.dex`, signing metadata and
   `assets/huntress_puzzle.json`. The launcher is `MainActivity`. `ChallengeActivity` is exported and
   accepts browsable `huntress03://intent-injection/unlock` links, which matches the prompt.
2. **Read the vulnerable path (Androguard disassembly).** In `ChallengeActivity.onCreate` the code reads
   `answer` and `case_id` with `getStringExtra`. If the Intent carries a URI, the URI's `answer` and
   `case_id` query parameters overwrite them. The values are then placed in an internal
   `/mobile/attempt/.../evidence?case_id=...&proof=...` URL that is loaded in a WebView. So the
   attacker-controlled values do cross an internal boundary unchecked. The class `IntentInjection.describe()`
   returns `DAY-03:intent_relay:attacker-controlled nested Intent`, which names the mechanic.
3. **Find where the answer lives.** `assets/huntress_puzzle.json` is a schema `huntress.mobile.puzzle.v1`
   document with `day: 3`, `codec: "base64"` and a `data` field:

   ```json
   {"schema":"huntress.mobile.puzzle.v1","day":3,"codec":"base64","data":"QU5TLUU0QzhFMzVBOEE5QjVENjc="}
   ```

   `classes.dex` only forwards this value as `proof=` in the WebView URL. Nothing needs to be executed.
4. **Decode it.** The solver reads the asset out of the APK, checks schema, day and codec, base64-decodes
   `data` and checks the `ANS-[0-9A-F]{16}` pattern:

   ```python
   with zipfile.ZipFile(apk_path) as apk:
       puzzle = json.loads(apk.read("assets/huntress_puzzle.json"))
   if puzzle.get("schema") != "huntress.mobile.puzzle.v1":
       raise ValueError(...)
   if puzzle.get("day") != 3 or puzzle.get("codec") != "base64":
       raise ValueError("APK does not contain the expected Day 3 base64 puzzle")
   answer = base64.b64decode(puzzle["data"], validate=True).decode("ascii")
   if not re.fullmatch(r"ANS-[0-9A-F]{16}", answer):
       raise ValueError("decoded value does not match the required answer format")
   ```

5. **Submit.** The Single Solve answer was `ANS-E4C8E35A8A9B5D67`, submitted once; the journal records
   `CORRECT` at 2026-10-18T09:37:49Z.

## Time Trial and Mastery

The only change in these modes was scale and clock.

- **Time Trial.** Same asset schema and the same script. The first attempt in the original run expired:
  the 50 s window ran out before the answer was submitted (the journal says only that; it records
  no cause). A `retry` issued a new case, package and APK, and the script's output for that new APK
  (`ANS-B3194A1A76066868`) was accepted. The earlier answer for the expired APK is not reusable.
- **Mastery.** Three APKs, each its own case and package, delivered via the artifact endpoint with
  delivery indices 0-2. The script was run once per APK and the three answers were submitted together
  under `set1.answer`, `set2.answer`, `set3.answer`; `CORRECT` came back on that single call.
- **Ranked reruns (2026-10-26).** Both modes were later re-run as scripted ranked attempts. Time Trial
  took 1.38 s end to end (about 0.7 s of that is the submit call) and Mastery 1.69 s, each `CORRECT` in
  one submission call, with the solve step itself under 0.02 s. These are the files that serve as the
  graded local evidence for those two modes.

## Dead ends and gotchas

- **The vulnerability is a decoy for the answer.** Following the prompt literally (building a malicious
  Intent, or running the app) is unnecessary; the nested-Intent/URI-overrides-extra path is analysis
  context only. The journal for the deep review notes the fixture contains URI-query-over-extra
  precedence and no actual nested-Intent code.
- **Short windows.** Days 1-5 Time Trial and Mastery windows were 50 s and 110 s, and a first Time
  Trial attempt expired during manual handling. Scripting download, decode and submit as one pass is
  what made later attempts trivial.
- **Saved APKs can be damaged.** The APKs saved locally in this leaf were corrupted by LF-to-CRLF
  translation (Python's `zipfile` reports a bad central directory). Checking the SHA-256 from the start
  response and re-downloading, or repairing in memory, fixes it. The decode itself is unaffected once
  the ZIP opens.
- **Answers are per case.** Each Time Trial retry and each Mastery set has a different case, package and
  answer, so values from another attempt never carry over.

## Takeaways

- For the early Mobile Android days, check `assets/huntress_puzzle.json` first. Its `schema`, `day` and
  `codec` fields identify the encoding (`base64` here), and the answer is the decoded `data`.
  Search terms: "huntress puzzle v1 codec base64".
- Still do the manifest and DEX read: it confirms the story (exported activity, deep link, URI
  parameters overriding extras) and shows `classes.dex` only echoing the value as `proof=`.
- Validate the decoded text against the stated answer pattern before submitting; with a 50 s window there
  is no room for a second try.
- Faster next time: skip the manual reading, run the script straight on the downloaded APK.

## Evidence

Leaf base path: `challenges/day03/mobile-android/mobile-android-day-03-intent-injection/`

- `single-solve/journal.md`, `single-solve/challenge.json` (start response, older wrapped shape),
  `single-solve/solve_intent_injection.py` (the solver, reused for every mode)
- `time-trial/journal.md`, `time-trial/ranked-1/result.json`, `time-trial/ranked-1/accepted_plan.json`,
  `time-trial/ranked-1/start_response.json`
- `mastery/journal.md`, `mastery/ranked-1/result.json`, `mastery/ranked-1/accepted_plan.json`,
  `mastery/ranked-1/start_response.json`
- `progress.md` rows for Day 3 (`Solved`, "Player-directed solve"): the Single Solve row has no
  saved graded result file, so that mode is recorded as journal-recorded only.
