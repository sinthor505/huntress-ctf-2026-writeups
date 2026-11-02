# Deflated Ego

| | |
|---|---|
| Day / Lane | 22 / Warmups |
| Challenge ID | `warmups-day-22` |
| Technique | layered text decoding, new codec: base64 wrapping zlib (DEFLATE) |
| Single Solve | Independent |
| Time Trial | Independent |
| Mastery | Independent |

## Summary

"Deflated Ego" is the Day 22 warmup: a text blob whose last decoded line names an answer slug. This
day's new layer was base64 over zlib, hinted at by the prompt ("a little inflation"). Single Solve and
Time Trial decode with one fixed chain; Mastery repeats the same idea across 108 blobs of 22
records each, where record number N selects codec N. All three modes went CORRECT on the first
submission. The only friction was a codec missing from our table (the solver failed closed and cost no
submissions) and one later ranked Mastery rerun that came back `incorrect` on a single set.

## The challenge

Prompt: "Another familiar wrapper. This one needs a little inflation." No live target; the Game Master
delivered a text artifact.

- **Single Solve / Time Trial:** one `blob.txt`, one answer id `message`. The question: decode the
  blob completely and give the exact slug its final non-empty line tells you to submit.
- **Answer format:** `pattern`, three lowercase words joined by hyphens, then a hyphen and a two-digit
  number (for example `tidy-cloud-tulip-70`, the Single Solve answer); submit the slug only; normalization `trim`.
- **Mastery:** a ZIP of 108 members (`blob-01.txt` ... `blob-108.txt`), answer ids `set1.message` ...
  `set108.message`, one slug per member. The 108 matches `5N-2` for N = 22, the pattern the earlier
  warmup Mastery days followed. Each member is 22 lines of the form `NN<TAB>payload`.
- **Clocks:** the untimed (regular) attempts had no deadline. Ranked attempts had a 30 s window for Time
  Trial and 125 s for Mastery, consistent with the earlier days' `5N+15` rule for Mastery.

## Solution

1. **Recon.** The prompt names the technique by riddle. "Wrapper" is the family used all week
   ("familiar") and "inflation" points at DEFLATE, the same family as Day 21's compression layer.
2. **Peel the layers.** The single blob decodes as base64, then zlib inflate, giving plaintext in the
   fixed warmup form: `Nice work decoding this warmup! To solve, submit this exact answer to the Game
   Master: <slug>`.
3. **Take the last non-empty line, then the slug after the final colon.** The journal records the
   rule as "last-line slug after colon"; the slug is validated against the answer pattern before
   submitting.
4. **Submit once.** The plan was `{"message": "tidy-cloud-tulip-70"}` and the first (only) submission was
   CORRECT. The accepted plan is saved in the leaf.

The decoder is our own `codec_chain.py` library tool (named in our library as
WARMUP-CODEC-CHAIN-AUTO-001): it expands the artifact, tries a pool of decoders, and scores the result
against the known sentence template and the slug pattern. The solver wrapper around it is short; the
interesting part is the pattern check before it returns anything:

```python
good = [r for r in results if r["slug_matches_pattern"]]
if not good:
    print("no slug decoded", file=sys.stderr)
    return 1
...
for q in questions:
    if "pattern" in q["answer_format"].get("kind", ""):
        assert re.fullmatch(cc.DEFAULT_SLUG, answers[q["id"]])
```

The solver refuses to emit an answer unless a decode ends in a slug that fits the format.

## Time Trial and Mastery

**Time Trial** is the Single Solve again on a fresh blob. Both the untimed regular attempt (3.63 s end
to end) and the first ranked attempt (3.2 s, 30 s window) were CORRECT in one submission.

**Mastery** is where the codec table matters. In each of the 108 members, line `NN` holds a chunk of the
same sentence encoded with codec `NN`: the first lines are base64 (`Tmljw...`, "Nice"), rot13, hex,
8-bit binary, decimal, octal, percent-encoding and so on, one new codec per line, up to the Day 22
codec on line 22. The solver decodes every record with the codec its number selects, joins the chunks,
and takes the slug from the last line. Each of the 108 members gets its own slug, so answers must be mapped by
the question's `artifact` name (`blob-NN.txt`), not by list position.

The first regular Mastery attempt had no record 22 in the table. Every set failed with "no slug decoded",
so nothing was submitted and no calls were spent. After adding `22: base64-zlib` to the table
(`DEFAULT_RECORD_MAP`), the regular Mastery went CORRECT in one call with all 108 answers (about
5 minutes after the first failure). The ranked Mastery attempt on release day was also CORRECT on one
call (11.64 s total, 125 s window).

## Dead ends and gotchas

- **A missing record codec shows up as a solver failure, not as wrong answers.** The decode pattern
  check (above) made the unknown record 22 fail closed. A solver that guessed would have submitted
  108 wrong slugs; here it cost nothing. See the library notes WARMUP-GOTCHAS-001 and
  WARMUP-PLAYBOOK-001 for the Day 21 contrast.
- **The single-blob solve hides the missing record.** Single Solve and Time Trial were a plain peel of
  base64 then zlib and passed without any table entry, while Mastery needed `22: base64-zlib` registered.
  Test the Mastery path, not just the Single Solve blob, before the clock starts.
- **One ranked Mastery rerun came back `incorrect` (107 of 108 accepted).** On a later ranked rerun,
  `set25.message` was the only rejected answer, found by bisection over 13 submission calls. In that
  run one of our two solver variants reported "no slug decoded" for set 25 while the other produced a
  value. The files do not say why set 25 decoded differently in that run, and we could not determine
  it. Three later ranked Mastery attempts (108 answers each) were CORRECT in one call, so it was not a
  general decode problem. See the Day 22 overview for automation issues.
- Two earlier problems were automation bugs, not challenge lessons: the second solver variant called a
  nonexistent helper and returned the same slug for every set, and it was rewritten to decode per
  artifact. Both were fixed before any wrong submission.

## Takeaways

- Technique name: **layered text decoding with per-record codec numbering** (warmup "codec chain").
  Identify each layer by its signature: a base64 string starting `eJ` decodes to zlib data (`78 9c`), so
  base64>zlib is recognizable without guessing.
- Validate every decoded line against the known template (`Nice work decoding this warmup!`) and the
  slug regex, and fail closed on mismatch. That caught the new codec for free.
- Add the new daily codec to the Mastery record table before submitting. Mastery record N is the day's
  codec for N, so the new record is always the one the previous table lacks.
- Mastery maps by artifact name, never by sort order or position.

## Evidence

Leaf base: `challenges/day22/warmups/warmups-day-22/`

- `single-solve/journal.md`, `single-solve/start_response.json`, `single-solve/accepted_plan.json`
- `single-solve/artifacts/cand-adapter/solve.py` (the solver wrapper around the library decoder)
- `time-trial/regular/result.json`, `time-trial/ranked-1/result.json` (CORRECT, 1 call each; later
  ranked reruns `ranked-2` to `ranked-4` also CORRECT)
- `mastery/regular/result.json` and `accepted_plan.json` (CORRECT, 108 answers, 1 call);
  `mastery/ranked-1/result.json` (CORRECT, 1 call); `mastery/ranked-2/result.json` (`incorrect`, 107
  accepted, `set25.message` rejected); `mastery/ranked-3` to `ranked-5` (CORRECT, 1 call each)
- `challenges/day22/_run/lane-warmups-day-22.log` (submission rounds, fail-closed, rerun records)
- Library: WARMUP-CODEC-CHAIN-AUTO-001, WARMUP-GOTCHAS-001, WARMUP-PLAYBOOK-001
- `progress.md` (grep `^| 22 .*warmups-day-22`)
