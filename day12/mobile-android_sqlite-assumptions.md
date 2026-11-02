# SQLite Assumptions

| | |
|---|---|
| Day / Lane | 12 / Mobile Android |
| Challenge ID | `mobile-android-day-12-sqlite-assumptions` |
| Technique | static APK triage; declared `hex` asset codec (`huntress.mobile.puzzle.v1`) |
| Single Solve | Solved (journal-recorded) |
| Time Trial | Solved (journal-recorded) |
| Mastery | Solved (journal-recorded) |

## Summary

The prompt talks about reconstructing authorization from a SQLite cache and proving server-side
impact. The APK contains no SQLite code or database. The answer sits in
`assets/huntress_puzzle.json` as a hex string, and the app only forwards it as a `proof=` parameter.
Hex-decoding the asset gave the answer in every mode. Each mode was submitted once (Mastery in two
calls), and the journals record `CORRECT` for all three.

## The challenge

Prompt: "Reconstruct authorization from a synthetic SQLite cache and prove server-side impact."
Each question names a case and an Android package, for example `MOB-12-4B4FA1A323` in
`com.huntress.mobile.d12.v4b4fa1a3235d`. It asks for "the exact backend answer code returned for
this case". The format was `pattern`, `ANS-` plus exactly 16 uppercase hex characters, with `exact`
normalization.

- Single Solve: one 12,704-byte APK, answer id `answer`, untimed.
- Time Trial: one fresh APK of the same size, answer id `answer`, 50-second server clock.
- Mastery: three APKs delivered separately, answer ids `set1.answer` to `set3.answer`, 110-second
  clock.

## Solution

1. **Recon.** The APK hash matched the one in the start response. The APK holds six members: the
   manifest, `classes.dex`, `assets/huntress_puzzle.json` and three `META-INF` signature files.
2. **Manifest.** `.ChallengeActivity` is exported and browsable for `huntress12://sqlite-assumptions/unlock`.
   The app is not debuggable and has backup disabled.
3. **DEX.** Reading `ChallengeActivity.onCreate` statically (Androguard, no install or execution), the
   activity takes `answer` and `case_id` from the Intent extras. If a URI is present, the URI query
   parameters override them. It URL-encodes both and loads
   `http://127.0.0.1:19090/mobile/attempt/<id>/evidence?case_id=...&client_secret=...&proof=<answer>`
   in a WebView. A `SQLiteAssumptions.describe()` method returns only a descriptive string,
   `DAY-12:sqlite_cache:client-side SQLite authorization state`, and no DEX code touches a database.
   The SQLite theme in the title is flavor text.
4. **The asset.** `assets/huntress_puzzle.json` is a small JSON file. In the Single Solve it read
   `{"schema":"huntress.mobile.puzzle.v1","day":12,"codec":"hex","data":"414e532d..."}`.
5. **Decode.** `data` hex-decodes to UTF-8 text that matches the required pattern. The whole mapper
   is a few lines:

   ```python
   asset = json.loads(archive.read("assets/huntress_puzzle.json"))
   assert asset["schema"] == "huntress.mobile.puzzle.v1" and asset["codec"] == "hex"
   candidate = binascii.unhexlify(asset["data"]).decode("utf-8")
   assert re.fullmatch(r"ANS-[0-9A-F]{16}", candidate)
   print(json.dumps({"answer": candidate}))
   ```

   Single Solve answer: `ANS-039F0D9071111050`.

The decoded value is the code the app would send as `proof` to the evidence endpoint. That
endpoint is a local loopback address, so there was no server to query offline. The decoded value
matched the required pattern, and the validator accepted it.

## Time Trial and Mastery

- **Time Trial.** Same APK structure and codec, with a fresh package and case. The asset decoded to
  `ANS-668B39E20D9A458F`, which was submitted inside the 50-second window.
- **Mastery.** Three APKs, each with its own `hex` asset (`ANS-CF8DBD28FC3C61A7`,
  `ANS-43D00DCBA9762EF1`, `ANS-A08AC2B1A2B70E61`). They were mapped to `set1`-`set3` in delivery
  order. `set1.answer` was submitted alone first and came back `accepted` with `remaining: 2`.
  Then `set2` and `set3` went in together and returned `CORRECT`. The journal's own review notes
  that one call would have sufficed, since every value already passed the format check.
- Time Trial and Mastery both reported `unique_attempts_remaining: 0` and
  `hall_of_fame_eligible: false` but still came with short windows.

## Dead ends and gotchas

- The title and prompt point at SQLite and authorization reconstruction. Nothing in the fixture
  implements that. `describe()` and the evidence URL are decoys for anyone expecting database work.
- The library's existing asset inspector did not support the `hex` codec at the time, and reported
  `supported: false` rather than guessing. The decode was done separately, and the codec was added
  to the library afterwards.
- The client secret in the DEX (`client_secret=...`) is a separate decoy from the answer. The answer
  is the asset-derived `proof`.
- The Mastery journal also records that the assigned APKs were not installed or run.

## Takeaways

For Mobile Android, check the declared codec in `assets/huntress_puzzle.json` before reading any
bytecode. In this lane the answer comes from the asset, and `classes.dex` only forwards it.
Cross-check the decode against the question's `pattern` format, and trace one call site in the DEX to
confirm the value is used as the `proof`. Don't take the title's theme literally: later days
used the same asset schema with other codecs and an `analysis` codec that needs the APK's own
bytecode run.

## Evidence

Leaf base: `challenges/day12/mobile-android/mobile-android-day-12-sqlite-assumptions/`

- `single-solve/journal.md`, `single-solve/start_response.json`, `single-solve/artifacts/answer_map.py`,
  `single-solve/artifacts/huntress_puzzle.json`
- `time-trial/journal.md`, `time-trial/start_response.json`, `time-trial/artifacts/answer_map.py`
- `mastery/journal.md`, `mastery/start_response.json`, `mastery/artifacts/answer_map.py`,
  `mastery/artifacts/first_attempt.json`
- Per-mode `ranked-1/result.json` (Time Trial, Mastery) read `practice_only`: a pool with no timed
  attempts left, so no graded ranked solve exists. Provenance rests on the journals' recorded
  `CORRECT` responses; no `result.json` with `CORRECT` was saved.
- `tools/ranked/solvers/mobile-android-day-12-sqlite-assumptions/solve.py` (later general solver)
- `progress.md` (grep `^| 12 .*sqlite-assumptions`)
