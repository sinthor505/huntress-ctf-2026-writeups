# Backup Spill

| | |
|---|---|
| Day / Lane | 5 / Mobile Android |
| Challenge ID | `mobile-android-day-05-backup-spill` |
| Technique | Static APK asset extraction, `split` fragment codec (reassemble shuffled parts) |
| Single Solve | Solved, graded result not preserved (candidate and `submission.json` kept; no local `result.json`) |
| Time Trial | Independent (own CORRECT, call 1, plus a ranked re-run CORRECT) |
| Mastery | Independent (own CORRECT, call 1, plus a ranked re-run CORRECT) |

## Summary

An APK whose manifest advertises a "backup-admitted SharedPreferences" mechanic hides the whole answer in a
JSON asset, stored as three shuffled string fragments plus a permutation. Reordering the fragments gives
the required `ANS-` code. The same one-line decode worked unchanged for every Single Solve, Time Trial and
Mastery delivery, so the later modes were a matter of running the decoder inside a 50-110 s window.

## The challenge

The prompt was "Extract synthetic application state admitted by a flawed backup boundary." Single Solve delivered one
`backup_spill.apk` and one question (`answer`) for a named case and package (`com.huntress.mobile.d05.<hash>`).
The answer format was `ANS-` followed by exactly 16 uppercase hex characters, normalization `exact`. Single Solve
was untimed. Time Trial returned one fresh APK with the same single `answer` field and a 50 s window.
Mastery returned three APKs (`backup_spill_mastery_01..03.apk`), answer IDs `set1.answer`, `set2.answer` and
`set3.answer`, in a 110 s window.

## Solution

1. **Recon the manifest.** `aapt dump xmltree` showed `android:allowBackup=true`, `debuggable=false`, a launcher
   `MainActivity` and a `ChallengeActivity` reachable through a `huntress05://backup-spill/unlock` deep link. The
   metadata string names the mechanic as backup-admitted SharedPreferences. This matches the theme but is only a
   lead: nothing in the APK is actually a backup file.
2. **Look at what the APK carries.** It has only six entries: the manifest, `classes.dex`,
   `assets/huntress_puzzle.json` and three `META-INF/` signature files. There is no plaintext answer in the
   archive strings.
3. **Read the asset.** The puzzle asset in the Single Solve APK was:

   ```json
   {"schema":"huntress.mobile.puzzle.v1","day":5,"codec":"split","data":"","parts":["0A353BE7B","ANS-5","709E18"],"order":[2,0,1]}
   ```

4. **Check the app code does not transform the answer.** Decompiling `ChallengeActivity.onCreate` showed it takes
   `answer` and `case_id` from an Intent or URI, URL-encodes them and forwards them to a localhost evidence
   endpoint. `BackupSpill` only holds constants (`CODEC = "split"`, `CONTROL = "backup_store"`). The APK was never
   installed or run; the answer comes from the asset alone.
5. **Reassemble.** `order[i]` is the output slot of `parts[i]`. Sorting `(order, part)` pairs gives
   `ANS-5` + `709E18` + `0A353BE7B`, i.e. the Single Solve candidate `ANS-5709E180A353BE7B`. It satisfies the
   format (`ANS-` plus 16 uppercase hex) and the fragments form a complete permutation.

The core of the extractor (`single-solve/artifacts/solve_backup_spill.py`) is:

```python
# order[i] is the output position of parts[i]
if sorted(order) != list(range(len(parts))):
    raise ValueError("order must be a permutation")
answer = "".join(part for _, part in sorted(zip(order, parts)))
if not re.fullmatch(r"ANS-[0-9A-F]{16}", answer):
    raise ValueError("does not match the answer pattern")
```

## Time Trial and Mastery

Nothing about the technique changed. The fresh APKs carried the same schema and codec, with new fragments, so the
generic asset decoder (the library `asset_codecs.py`, which picks the codec from the asset's own `codec` field) was
run directly, with no per-day code:

- **Time Trial:** one APK, one answer, `matches_pattern: true`, one submission, CORRECT. The window was about 50 s
  and a plain retry was already a timed attempt on this day, so the solve was scripted.
- **Mastery:** three APKs. The first delivery is the only one the standard artifact download returns, so the other two
  were fetched from the capability URLs in the start response; all three SHA-256 values matched the start response
  before decoding. Each decoded to a pattern-valid `ANS-` value, all three IDs went in one submission, CORRECT.
- **Ranked re-runs (2026-10-26):** a dedicated per-challenge solver (same `split` logic, pairing each question to
  its APK by the `in package X` text in the question) re-ran both modes on fresh attempts. Time Trial finished in
  1.17 s total and Mastery in 1.85 s, each CORRECT on one call, including download and submit.

## Dead ends and gotchas

- **Fragment order direction.** The only real ambiguity is whether `order[i]` places `parts[i]` into output slot
  `order[i]`, or says which part goes in slot `i`. Every saved split asset used three parts with `order` `[2,0,1]`,
  and the inverse permutation yields a different string, so the choice matters. The forward reading
  (`parts[i]` goes to slot `order[i]`) was accepted by the validator in every mode.
- **`allowBackup=true` is a decoy theme, not a mechanism.** There is no backup artifact to pull; the manifest flag
  is the only backup-related element, and the value is entirely in the asset.
- **Saved APK copies were damaged.** The local copies of the Single Solve APK were corrupted by LF to CRLF
  translation (the ZIP central directory failed to parse). The library tools repair them in memory and the repaired
  hash equals the API hash. Always check the hash against the start response (see the day overview).
- **Short windows.** Time Trial (50 s) and Mastery (110 s) leave no room for manual analysis, so the decoder has
  to be ready before the retry.

## Takeaways

- On the Mobile Android lane, look in `assets/huntress_puzzle.json` first; the manifest theme (backup, deep link) is
  flavor and the DEX only forwards a decoy value.
- Name the technique: `huntress.mobile.puzzle.v1` `split` codec, a fragment shuffle undone by a position
  permutation. Validate the permutation and the answer pattern before submitting.
- A generic decoder keyed on the asset's own `codec` field turned Time Trial and Mastery into one-call solves.
  Mastery adds only bundle handling (several APKs, one question each).

## Evidence

Base path: `challenges/day05/mobile-android/mobile-android-day-05-backup-spill/`

- `single-solve/journal.md`, `single-solve/artifacts/solve_backup_spill.py`, `single-solve/submission.json`
  (candidate only; no graded `result.json` was saved for this mode, and no Single Solve submission appears in the
  ledger)
- `time-trial/journal.md`, `time-trial/artifacts/answer_map.json`, `time-trial/ranked-1/result.json` (CORRECT)
- `mastery/journal.md`, `mastery/artifacts/answer_map.json`, `mastery/ranked-1/result.json` (CORRECT)
- Per-challenge ranked solver: `tools/ranked/solvers/mobile-android-day-05-backup-spill/solve.py`
- `progress.md` rows for Day 5 mark all three modes Solved.
