# Food for Thought

| | |
|---|---|
| Day / Lane | 26 / Warmups |
| Challenge ID | `warmups-day-26` |
| Technique | Brainfuck-style accumulator stream (`+` `-` `.`) as a warmup codec; Mastery slug-extraction trap |
| Single Solve | Independent |
| Time Trial | Independent |
| Mastery | Independent (two ranked attempts were `incorrect` first, see below) |

## Summary

"Food for Thought" hides a sentence in a stream made only of `+`, `-` and `.`: one accumulator,
`+`/`-` change it by one, `.` emits it as a byte. The Single Solve and Time Trial are one blob each;
decoding it gives a line ending in the answer slug. The Mastery bundle was 128 sets, each a
26-record blob where record 26 is the new codec. All three modes ended CORRECT, but one Mastery
attempt taught us that the question asks for the literal tail of the last line, not the first
slug-shaped substring in it.

## The challenge

Prompt: "A very small language has quite a lot to say." Single Solve and Time Trial delivered one
`blob.txt` and one answer id (`message`); Mastery delivered a ZIP of `blob-NN.txt` files and 128
answer ids (`setN.message`). The question was the standard warmup one: decode the blob completely,
then submit the exact slug that its final non-empty decoded line tells you to submit. Answer format
was a pattern: three lowercase words joined by hyphens, then a hyphen and a two-digit number,
`trim` normalization. The final line reads `... answer to the game master: <slug>`.

## Solution

1. **Recognize the alphabet.** The Single Solve blob was only `+`, `-` and `.`. That is the
   accumulator subset of Brainfuck with no loops or pointer moves, so no interpreter was needed.
2. **Decode.** Start the accumulator at 0, add or subtract 1 per `+`/`-`, and on `.` emit
   `acc mod 256` as a byte. The decoder:

   ```python
   def plusminus(text, acc=0):
       t = "".join(text.split())
       out = []
       for c in t:
           if c == "+":   acc += 1
           elif c == "-": acc -= 1
           else:          out.append(chr(acc % 256))
       return "".join(out), acc
   ```

   The accumulator never resets inside one blob, so the stream climbs to the first letter and then
   moves by small deltas between letters.
3. **Take the slug.** The last non-empty line is the template sentence; the slug is the text after
   the last `:`. The Single Solve answer was `happy-comet-blossom-79`, CORRECT on the first and only
   submission, roughly two minutes after the challenge was detected as released.
4. **Time Trial.** Same decoder on a fresh blob, one call, CORRECT (regular attempt and four timed
   attempts all took one submission call each; the timed window was 30 s and the local solve was
   under a second).

## Time Trial and Mastery

Time Trial was identical in shape. Mastery was a 128-set ZIP (Mastery grows by five sets per day).
Each blob is `NN<TAB>payload` records, and Day N has N records, so Day 26 has 26. The record number
fixes the codec: records 1-25 are the previous days' codecs (base64, ROT13, hex, ... code points)
and record 26 is the `+`/`-`/`.` stream. Fragments are decoded and concatenated in record order to
rebuild the sentence. Two details mattered:

- **Accumulator scope.** We could not tell from the prompt whether the accumulator resets for each
  record or carries across records. The fresh-per-record reading decoded all 128 sets of three saved
  Mastery bundles with no repairs, and it is what the validator accepted.
- **Record 16 must not match the new codec.** Record 16 is Morse-for-hex, which contains only `.`,
  `-` and spaces and so passes a loose "is this a `+-.` stream" check. Tried before the codec table,
  the new decoder turned record 16 into garbage. Requiring at least one `+` rejects it, since
  starting from 0 a stream without `+` can only emit bytes >= 0x80.

The regular Mastery went CORRECT in one call of 128 answers.

## Dead ends and gotchas

- **Truncated slug in ranked Mastery (the real trap).** Our first two ranked Mastery passes (the same
  attempt, the second pass reattached to it) came back `incorrect` with 127 of 128 ids accepted;
  only `set6.message` was wrong. Nothing was wrong with the decoding: the decoded last line of that
  set ended `...answer to the game master: word-word-word-NN-NNNNN`, a slug plus an extra 5-digit
  suffix. Our extractor used a regex search for the first `word-word-word-NN` and submitted that
  prefix, which the validator rejected. The question asks for the exact slug the line tells you to
  submit, so the literal tail after the last colon was the answer. Evidence it was intended and not
  a decoding error: the checksums of the compressed layers passed, and every record's fragment length
  followed the generator's even split of the full message, as it did for all 256 other blobs.
  Fix: keep the literal tail as the primary answer and offer the pattern-shaped prefix only as a
  fallback. The first call of the repaired solver (a fresh ranked attempt) was CORRECT. That fresh
  bundle did not contain the odd tail, so the literal-tail path was checked offline against the
  rejected attempt's saved data, not by a live CORRECT.
- **The format line is not a regex to search with.** "Three words, a hyphen, two digits" describes
  the usual answer; a line that does not match it is a reason to read the tail by hand, not to cut
  it down to fit.
- **Incorrect is batch-level.** A rejected 128-answer submission reports only that the batch failed;
  we found the single bad id by halving the batch, which cost 15 calls on one attempt.
- **Attribution (one line).** One Mastery CORRECT was recorded under a solo account instead of the
  team, so it was re-run (a fresh `retry` went CORRECT with the same solver). See the day overview
  for the identity-attribution issue.
- A solver mistake that is not about the challenge: an early solver asserted on every Mastery set
  because it tried the new codec on record 16 first (above). See the day overview.

## Takeaways

Technique names to search for: **Brainfuck accumulator subset**, **warmup codec chain**,
**slug extraction (literal tail vs. pattern search)**. For a warmup day: decode by record number
rather than by sniffing, make the new codec strict enough that it cannot match older records, and
when a decoded line does not fit the advertised answer format, submit what the line literally says.
The Day 27 Single Solve is this same program re-spelled as Ook! tokens, so a strict `+-.` layer
carries over to it.

## Evidence

Leaf base: `challenges/day26/warmups/warmups-day-26/`

- `single-solve/journal.md`, `single-solve/accepted_plan.json`, `single-solve/start_response.json`
- `single-solve/artifacts/cand-adapter/solve.py` (the original decoder and slug extractor)
- `time-trial/regular/result.json`, `time-trial/ranked-1/result.json` ... `ranked-4/result.json`
- `mastery/regular/result.json`, `mastery/regular/accepted_plan.json`
- `mastery/ranked-1/result.json`, `mastery/ranked-2/result.json` (127/128, `set6.message` rejected)
- `mastery/artifacts/escalation-diagnosis.md` (root cause of the truncated slug)
- `mastery/ranked-3/result.json`, `mastery/ranked-4/result.json` (CORRECT, fixed solver)
- `challenges/day26/_run/lane-warmups-day-26.log` (submission rounds)
- `progress.md` (grep `^| 26 .*warmups-day-26`)
- No librarian note exists for this challenge under `challenges/day26/_run/librarian/`.
