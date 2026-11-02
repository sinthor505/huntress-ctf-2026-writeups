# One More Thing

| | |
|---|---|
| Day / Lane | 31 / Warmups |
| Challenge ID | `warmups-day-31` |
| Technique | layered text codec: base32 -> gzip -> base64 on the wire |
| Single Solve | Team solve (not ours) |
| Time Trial | Independent |
| Mastery | Independent |

## Summary

"One More Thing" is the finale day's Warmups challenge: the blob is plaintext wrapped in base32,
then gzip, then base64, in that order, and the final decoded line is the usual warmup sentence
naming the answer slug. No new decoding primitive was needed — base32, gzip and base64 were all
already in the codec-chain tool — but the Mastery bundle's record-31 marker was new and not yet in
the tool's per-record lookup table. The Single Solve was scored by a teammate before our
submission landed, so nothing was graded for us there; Time Trial and Mastery both went CORRECT on
the first submission call, which is what this page treats as the real evidence.

## The challenge

Prompt: "Thirty-two, a little pressure, then sixty-four. Unpack that thought." Every Warmups mode
asks the same question: decode the assigned blob completely and submit the slug named by its last
non-empty decoded line. The answer format is a `pattern` (three lowercase words joined by hyphens,
then a hyphen and a two-digit number), normalization `trim`, one answer id (`message`).

- Single Solve and Time Trial: one text blob (`blob.txt`), one answer.
- Mastery: a ZIP of 153 blobs (`blob-01.txt` ... `blob-153.txt`), 153 answers with ids
  `setN.message`, each pointing at its blob through the question's `artifact` field. Each blob is a
  list of numbered, tab-separated records, and the record number selects the codec for that record
  (Day N's bundle has records up to N, so Day 31 reaches record 31).

## Solution

1. **Read the prompt as an encoding recipe, in encode order.** "Thirty-two" names base32,
   "a little pressure" names compression (gzip), "sixty-four" names base64 — the order the
   plaintext was wrapped in before transit. Decoding is the reverse: base64-decode, gunzip, then
   base32-decode.
2. **Reuse the existing solver unchanged.** The Single Solve and Time Trial blobs are not
   record-tagged at all — just the three layers stacked directly — so the library's generic
   codec-chain decoder peeled them with no table lookup and no code change, the same thin wrapper
   used on prior Warmups days:

   ```python
   results = cc.process_all(list(cc.iter_inputs([Path(a.artifact)])), crib,
                            cc.DEFAULT_SLUG, dict(cc.DEFAULT_RECORD_MAP), 16, strong, [])
   ```

3. **De-risk the Mastery format before it arrived.** The Mastery bundle's blobs *are*
   record-tagged, and record 31 (the day-31-specific record) was not yet in
   `DEFAULT_RECORD_MAP`. Rather than wait and find out live, the day-31 solo worker built 148
   synthetic 31-record blobs by taking Day 30's saved Mastery sets, splitting record 30's fragment
   in two, and re-encoding the second half as a new record 31 wrapped the same way as today's
   Single Solve blob (`mk_synth.py`). Running the unchanged solver against all 148 matched Day
   30's own saved per-set answers 148/148 (`run_synth.py`), and the tool's auto-detect path picked
   a generic 3-step peel (`peel-3`) for the unmapped record and got it right.
4. **Submit.** Single Solve returned `already_solved` — a teammate had already scored it — so our
   offline answer map there was never validated by the Game Master. Release automation detected
   that the team's `CORRECT` readout was showing on `start` and moved straight to issuing fresh
   Time Trial and Mastery attempts via `retry` rather than treating the Single Solve result as a
   stopping point. Both went CORRECT on the first submission call:
   - Time Trial (regular attempt): `fancy-compass-willow-51`.
   - Mastery (regular attempt): 153 answers, e.g. `set1.message = brave-rocket-copper-77`.

## Time Trial and Mastery

Both Time Trial and Mastery used the same unchanged solver against the real record-31 blobs, with
the record still resolved by auto-detection at this point:

- **Time Trial:** one blob, one answer. Regular (untimed) attempt: CORRECT on call 1, `t_total`
  3.67 s. A later ranked, timed attempt (30 s window) was also CORRECT on call 1 in 3.62 s, with a
  different assigned slug (`tidy-tent-grove-58`) since the Game Master issues a fresh attempt per
  `retry`.
- **Mastery:** 153 sets in one ZIP (up from Day 30's 148), each decoded independently and mapped to
  its answer id through the question's `artifact` field. Regular attempt: CORRECT on call 1 for all
  153 answers, `t_total` 9.99 s. A ranked, timed attempt (170 s window) was also CORRECT on call 1
  in 7.23 s.
- **Promoting the new record.** After both CORRECT results, `31: "base64-gzip-base32"` was added
  to `DEFAULT_RECORD_MAP` and a matching `base64-gzip-base32` codec function registered in the
  library tool. Re-running the tool against the saved Time Trial blob and the saved 153-set Mastery
  ZIP and diffing the slug multiset against each leaf's `accepted_plan.json` gave 1/1 and 153/153
  matches with zero repairs and zero `auto_picked` records (versus 153/153 auto-picked as `peel-3`
  before the entry was added, with identical slugs either way). A later ranked rerun of both modes
  (post-event bragging-rights sweep) used the now-registered table entry directly and was CORRECT
  on call 1 in both modes.

## Dead ends and gotchas

- **Record 31 was unmapped at delivery time, same as Day 30's record 30.** The house rule is to
  add a new record number to the table only after a CORRECT that actually exercised it, so the
  live Mastery run depended on the generic auto-detect fallback (`peel-3`) rather than a table
  entry — the synthetic rehearsal in step 3 above is what justified trusting that fallback instead
  of guessing blind.
- **The prompt names layers in encode order, not decode order.** "Thirty-two ... then sixty-four"
  describes how the plaintext was wrapped, which is the opposite of the order to peel it back in.
- **Team-scored Single Solve.** `already_solved` on the submission, and the team's `CORRECT`
  showing up on `start`, are not evidence of our own solve; release automation's rule is to treat
  that as a signal to move straight to independent Time Trial/Mastery attempts rather than stop.

## Takeaways

- Technique name: **layered codec chain** (base32 -> gzip -> base64 on the wire, same family as
  every other Warmups day — only the layer combination changes). The per-record lookup table that
  drives Mastery decoding now covers records 1-31.
- When a new record number is unmapped, a synthetic rehearsal built from a prior day's real
  Mastery data (same split-and-re-encode trick as `mk_synth.py`) is a cheap way to validate the
  auto-detect fallback before the real timed attempt, rather than relying on it untested.
- Because the Single Solve was lost to a teammate within seconds, the independent proof that the
  workflow still works end to end came from Time Trial and Mastery.

## Evidence

Leaf base path: `challenges/day31/warmups/warmups-day-31/`

- `single-solve/journal.md`, `single-solve/start_response.json`, `single-solve/team_solved.json`
- `single-solve/artifacts/cand-solo/solve.py` (the wrapper used for all three modes)
- `single-solve/artifacts/scratch-solo/mk_synth.py`, `run_synth.py` (synthetic record-31 rehearsal)
- `time-trial/regular/result.json`, `time-trial/regular/accepted_plan.json`,
  `time-trial/ranked-1/result.json`, `time-trial/ranked-1/accepted_plan.json`
- `mastery/regular/result.json`, `mastery/regular/accepted_plan.json`,
  `mastery/regular/start_response.json`, `mastery/ranked-1/result.json`
- `challenges/day31/_run/librarian/warmups-day-31.md`, `challenges/day31/_run/lane-warmups-day-31.log`
- Library: `library/warmups/codec-chain/codec_chain.py` (record 31 entry), `WARMUP-CODEC-CHAIN-AUTO-001`,
  `WARMUP-PLAYBOOK-001`
- `progress.md` (grep `^| 31 .*warmups-day-31`)
