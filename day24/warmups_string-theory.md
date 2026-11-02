# String Theory

| | |
|---|---|
| Day / Lane | 24 / Warmups |
| Challenge ID | `warmups-day-24` |
| Technique | new framing codec (concatenated netstrings `len:data,`) in a layered-codec warmup |
| Single Solve | Independent |
| Time Trial | Independent |
| Mastery | Independent |

## Summary

The Day 24 warmup hides a "submit this slug" instruction behind a text codec we had not seen before:
a run of concatenated netstrings (`len:data,`). The library's auto-detecting decoder misread it as a
column transposition and returned garbage with trailing commas, so we decoded the netstrings directly.
Every length in every blob we saw was 1, which matters for the one open ambiguity below. All three modes
went CORRECT on the first submission of the attempt that counted, after a Mastery solver that initially
crashed on the older codecs in its records.

## The challenge

Prompt: "Count carefully. The comma knows where to stop." The question for every blob is the same:
decode it completely and report the slug that the final non-empty decoded line tells you to submit.

- **Answer format:** one answer id (`message`), kind `pattern`: three lowercase words joined by hyphens,
  then a hyphen and a two-digit number, submit the slug only; `trim` normalization.
- **Single Solve and Time Trial:** one `blob.txt` each, no deadline on the untimed attempts. Ranked Time
  Trial windows were 30 s.
- **Mastery:** one ZIP (`warmups-day-24-mastery.zip`) with 118 blobs, answer ids `set1.message` through
  `set118.message`, each mapped to its blob by the question's `artifact` field. Ranked windows were 135 s.

## Solution

1. **Recon.** The blob was a long string of tokens shaped like `1:N,1:i,1:g,...`. The prompt's "comma
   knows where to stop" says each token carries its own length and ends at a comma.
2. **First reading was wrong.** Running the library's `codec_chain.py` on the blob peeled it as a
   `columns-inv-4` transposition and produced text with trailing commas that failed the slug pattern.
   The codec was new for Day 24, so no stored decoder matched.
3. **Decode netstrings.** A strict netstring walk (read the digits up to `:`, take that many bytes,
   require a `,`) turns the blob into plain text:

   ```python
   while i < n:
       j = data.index(b":", i)
       ln = int(data[i:j])
       end = j + 1 + ln
       if data[end:end + 1] != b",":
           raise ValueError("bad netstring terminator")
       out.append(data[j + 1:end])
       i = end + 1
   ```
4. **Extract the slug.** The question says the last non-empty decoded line carries the slug;
   our journal records the rule as "last line, text after its final colon", matched against the three-words-plus-two-digits pattern. Single Solve answer: `jaunty-compass-mosaic-76`.
5. **Time Trial** used the identical codec on a fresh blob and went CORRECT in one call.

## Time Trial and Mastery

Time Trial changed only the blob. Mastery is where scale mattered:

- **118 blobs, each in a different container.** Mastery blobs are `NN<TAB>payload` records, one record
  per earlier day's codec. The record number selects the codec (record 24 is this day's netstrings;
  records 1-23 are the older Warmups codecs). A solver that only knew the Day 24 codec crashed on
  records 1-23, so the first Mastery attempt had no usable candidate for any set. The fix was per-record
  dispatch through the library's record table, with record 24 decoded as netstrings. It then decoded
  118 of 118 sets to slug-shaped answers, and the regular Mastery attempt went CORRECT in one call.
- **Promoting the codec.** Afterwards the netstring decoder was added to `codec_chain.py` as a strict
  layer and as the codec for record 24, with a self-test, so later days no longer need a per-day decoder.
- **Ranked attempts.** All five ranked Time Trial passes were CORRECT in one call. Of five ranked
  Mastery passes, four were CORRECT in one call. The fourth was `incorrect`: 117 of 118 sets were
  accepted and a single set (`set45`) was rejected, after four submission calls. The next ranked pass,
  on a fresh bundle, was CORRECT in one call. We could not determine from the saved files why that one
  set failed.

## Dead ends and gotchas

- **Auto-detect misreads the new codec.** Anything that tries known codecs and keeps the first one that
  "looks like text" may accept the column-transposition reading of a netstring blob. The giveaway was
  trailing commas in the output and a slug that failed the pattern check.
- **Netstrings versus `COUNT:CHAR,` run-length is unresolved.** A second reading of the same tokens,
  run-length `COUNT:CHAR,`, produces byte-identical text whenever every count is 1, and every count in
  every saved blob was 1 (Single Solve, both Time Trials, both 118-set Mastery bundles). Only the
  netstring reading was ever submitted and it was accepted every time, so the validator never
  distinguished them. The prompt ("the comma knows where to stop") fits netstrings better.
- **Mastery records carry older codecs.** A Day-N Mastery blob has N records and the record number fixes
  the codec; do not assume every record uses the day's new codec.
- Our Mastery solver's first crash was a coverage bug in our own tooling; see the day overview.

## Takeaways

- Netstrings are a cheap framing to recognize: `digits:payload,` repeated, with the length telling you
  where the payload ends. Add them to the codec stack of any "decode all the layers" warmup.
- When a stored decoder returns text that fails the answer pattern (trailing junk, non-slug), distrust
  the auto-detected layer and read the prompt for a structural hint before forcing a different codec.
- Test a new warmup decoder on the Mastery bundle early: those blobs mix every earlier codec by record
  number, not just the day's own.

## Evidence

Leaf: `challenges/day24/warmups/warmups-day-24/`

- `single-solve/journal.md` (decoder notes and library promotion), `single-solve/start_response.json`,
  `single-solve/accepted_plan.json`
- `single-solve/artifacts/cand-adapter/solve.py` (netstring decoder with per-record dispatch; the
  submitted candidate)
- `time-trial/regular/result.json`, `time-trial/ranked-1/result.json`
- `mastery/regular/result.json`, `mastery/ranked-1/result.json`, `mastery/ranked-4/result.json`
  (the 117-of-118 pass), `mastery/ranked-5/result.json`
- `challenges/day24/_run/lane-warmups-day-24.log`
- Library: `library/warmups/codec-chain/` (record table and netstring layer), Warmups lane playbook
  and gotchas entries
- `progress.md` (grep `^| 24 .*warmups-day-24`)
