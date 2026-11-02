# You Have a Point

| | |
|---|---|
| Day / Lane | 25 / Warmups |
| Challenge ID | `warmups-day-25` |
| Technique | `U+XXXX` Unicode code point list decoded to text (new codec in a layered-decoder warmup) |
| Single Solve | Independent |
| Time Trial | Independent |
| Mastery | Independent |

## Summary

The Day 25 warmup blob is a whitespace-separated list of `U+XXXX` Unicode code points. Decoding it is
plain `chr(int(hex, 16))`, and the answer slug is on the last non-empty line of the decoded text. The
only real work was that none of the existing decoder layers accepted the `U+` prefix, so a new layer had
to be added to the shared decoder before submitting. After that, Single Solve, Time Trial and the
123-set Mastery were each CORRECT on the first submission.

## The challenge

Prompt: "Every character has a point. A code point, anyway." The question, identical in every mode:
decode the assigned blob completely and submit the slug its final non-empty decoded line tells you to
submit. Answer format: `pattern`, three lowercase words joined by hyphens plus a hyphen and a
two-digit number, submit the slug only, `trim` normalization.

- **Single Solve:** one `blob.txt`, one answer id (`message`).
- **Time Trial:** the same shape against a fresh blob. Ranked windows were 30 s.
- **Mastery:** one ZIP, `warmups-day-25-mastery.zip`, of `blob-NN.txt` members and 123 answer ids
  (`set1.message` ... `set123.message`); each question names its member in its `artifact` field.
  Ranked window was 140 s.

## Solution

1. **Recognize the encoding.** The title is literal. The blob is tokens such as `U+004E U+0069 ...`,
   one token per character. The decoder layers the shared warmup tool already had (`decimal`, `hex`,
   `html-references`) all reject the `U+` prefix, so the blob did not auto-peel.
2. **Add the layer first.** We added a strict `unicode-codepoints` layer and a `codepoints` record
   codec (registered as record 25 in the per-record table) to the shared decoder. The strict part
   matters: every token needs a prefix, so a bare hex list still belongs to the `hex` layer and is not
   misread. The core of the solver's decoder is:

   ```python
   CP = re.compile(r"(?i)U\+([0-9a-f]{4,6})")

   def codepoints(text: str) -> str:
       toks = text.split()
       if not toks or not all(CP.fullmatch(t) for t in toks):
           raise ValueError("not a code point list")
       return "".join(chr(int(t[2:], 16)) for t in toks)
   ```

3. **Take the slug from the last line.** The solver takes the last non-empty decoded line, keeps what
   follows the last `:`, and matches `[a-z]+-[a-z]+-[a-z]+-\d{2}` (falling back to the whole line).
   Before submitting, it asserts the pattern matches and that the answer ids equal the question ids.
4. **Submit.** Single Solve answer for `message`: `brave-beacon-wizard-34`. One submission,
   CORRECT.

Both the decoding and the slug extraction were unambiguous, so there were no alternate readings to
submit. The answer groups were empty.

## Time Trial and Mastery

Same decoder, wider fixtures:

- **Time Trial:** a single blob again; CORRECT on the first call. The first ranked attempt had a 30 s
  window and finished in 2.75 s locally.
- **Mastery:** 123 sets, each a separate member of the ZIP. The solver maps each question to its member
  through the question's `artifact` field rather than by position, decodes each one with the same
  table-driven path, and submits all 123 slugs in one call, CORRECT on the first call. The first ranked
  attempt (140 s window) finished in 13.62 s locally, most of it decoding (about 11.8 s in the
  untimed rehearsal). The solver also accepts `NN<TAB>payload` record files (older warmup days used
  them); our notes do not say whether any Day 25 Mastery member used that form.

The ranked runs were repeated later (three more Time Trial and four more Mastery attempts, all CORRECT
on one call each). Those were speed reruns for bragging rights, not a different challenge.

## Dead ends and gotchas

- No rejected answers. Every submission across SS, TT and Mastery came back CORRECT on the first call
  (`progress.md` lists 2 ledger calls each for TT and Mastery, matching the one regular and one ranked
  submission in the lane log).
- The one trap was tooling, not validator behavior: a decoder that auto-detects layers fails silently on
  an unknown encoding, and the closest existing layers (`decimal`, `hex`, `html-references`) each reject
  the prefix. Check that auto-detect actually peels the blob before trusting a clean-looking run.
- Keep the new layer strict. Accepting prefix-less tokens would steal inputs from the `hex` layer.

## Takeaways

Technique to search for: **Unicode code point list (`U+XXXX`) decoding**, the decoder-per-day pattern in
the Warmups lane (a new codec each day, with Mastery bundling many blobs). When a new day appears, read
the title and prompt literally ("a point" -> code point), add the codec to the shared decoder before the
timed modes, and verify against the saved Single Solve blob. On the regular rehearsals, the same code
path was CORRECT on the first call in all three modes, so the work that mattered happened before the
clock started.

## Evidence

Leaf: `challenges/day25/warmups/warmups-day-25/`

- `single-solve/journal.md`, `single-solve/start_response.json`, `single-solve/accepted_plan.json`
- `single-solve/artifacts/cand-adapter/solve.py` (decoder and slug extraction shown above)
- `time-trial/regular/result.json`, `time-trial/ranked-1/result.json`
- `mastery/regular/result.json`, `mastery/ranked-1/result.json`

Day-level, under `challenges/day25/_run/`: `lane-warmups-day-25.log` (CORRECT on the first call in every
mode, regular and ranked), `librarian/warmups-day-25.md`. Shared library entries: the warmups
codec-chain tool (record 25 row) and the warmups gotchas file (Day 25 entry). `progress.md` (grep
`^| 25 .*warmups-day-25`).
