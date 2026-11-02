# Safe Travels

| | |
|---|---|
| Day / Lane | 11 / Warmups |
| Challenge ID | `warmups-day-11` |
| Technique | URL-safe Base64 with the terminal padding stripped; numbered-record codec chain in Mastery |
| Single Solve | Solved (journal-recorded) |
| Time Trial | Independent |
| Mastery | Independent |

## Summary

"Safe Travels" is the Day 11 warmup: a short text blob that is URL-safe Base64 with the `=` padding
removed. Restoring the padding that the length requires and decoding with the URL-safe alphabet gave a
sentence whose last line named the answer slug. Time Trial was the same decode on a fresh blob;
Mastery wrapped 53 sets of 11-record chains in one ZIP, and the same decoder handled all of them.

## The challenge

Prompt: "Our usual base went on a URL-friendly vacation." The riddle points at Base64 (the "usual
base") in its URL-safe form.

- **Single Solve / Time Trial:** one question, `message`: "Decode the assigned blob completely. What
  exact answer slug does its final non-empty decoded line instruct you to submit?" The artifact was
  `blob.txt`, 147 bytes of ASCII with no trailing newline in Single Solve. The answer format was a
  `pattern`: three lowercase words separated by hyphens, then a hyphen and a two-digit number, slug
  only, `trim` normalization.
- **Mastery:** one ZIP of 53 members (`blob-01.txt` ... `blob-53.txt`) and 53 answer ids
  (`set1.message` ... `set53.message`). Each question's `artifact` field names its member.

## Solution

1. **Recon.** The blob matched the delivered SHA-256 and was a pure Base64-alphabet string. Its
   length left three characters in the last Base64 quantum, so exactly one `=` was missing. That
   fit the "URL-friendly" hint: padding is the usual casualty when Base64 goes into a URL.
2. **Decode.** Add `(-len) % 4` padding bytes, then decode URL-safe:

   ```python
   import base64
   raw = open("blob.txt", "rb").read()
   print(base64.urlsafe_b64decode(raw + b"=" * (-len(raw) % 4)).decode())
   ```

   Output:

   ```text
   Nice work decoding this warmup! To solve, submit this exact answer to the Game Master: crisp-starfish-petal-89
   ```
3. **Answer.** The final non-empty line carried the slug, which matched the required pattern:
   `crisp-starfish-petal-89`, submitted as `message`. The journal records `CORRECT` on the first
   submission; no result file was saved for this attempt.

## Time Trial and Mastery

- **Time Trial.** The same unpadded URL-safe Base64 on a fresh blob, with a 20 s window. It was
  scripted as one fetch, decode, submit pass using the shared codec-chain tool, whose strict
  `base64` layer restores padding and accepts the URL-safe alphabet. `CORRECT` on submission 1. A
  later ranked rerun was also `CORRECT` on call 1 (about 1.8 s end to end).
- **Mastery.** The 53 members are not 53 single-codec blobs. The journal first recorded them that
  way, then corrected itself on review: each member is an 11-record chain of `NN<TAB>payload`
  lines, where record N uses the Nth codec of the lane's cumulative table (Day 11's new record was
  `base64`; records 1-10 are the earlier days' codecs: base64, rot13, hex, binary, decimal, octal,
  percent, HTML references, JSON string, base32). The codec-chain tool detected both shapes and
  reported 11 records for every member, so the mapper needed no change. All 53 slugs matched the
  slug pattern and the standard warmup message template with zero repairs. The answer map keyed
  each slug by the question's `artifact` field (member name), not by position. The timed window was
  about 62-70 s; the solve was one scripted fetch, expand, decode, submit pass, `CORRECT` on
  attempt 1. Four later ranked Mastery reruns were each `CORRECT` on one call (about 1.7-4.8 s
  total).

## Dead ends and gotchas

- **Stripped padding.** A strict Base64 decoder rejects the blob as-is; the fix is to add only the
  padding the length demands. A length of 1 mod 4 cannot be Base64 at all, so the tool refuses
  that case rather than guessing.
- **Mixed alphabets.** The shared tool also refuses input that mixes `+/` with `-_`, since that
  would hide a wrong reading. The Single Solve journal notes the specimen needed no character
  substitutions beyond the padding.
- **Percent decoding was considered and ruled out**: the bytes contained no `%XX` escapes.
- **Mastery is not one codec per file.** The journal's first description (single-codec blobs) was
  wrong. It only mattered for understanding the lane; the detection in the tool made the result
  correct either way.
- Timer: Time Trial and Mastery windows were short (20 s and about 62-70 s), which is why these
  modes were scripted end to end rather than decoded by hand.

## Takeaways

Technique names: **unpadded / URL-safe Base64** (restore `(-len) % 4` `=` padding, `-_` for `+/`)
and the **numbered record chain** shape of warmup Mastery bundles (record N = Nth codec in the
cumulative table; map each ZIP member to its question by the `artifact` field). Do not hand-retype
bytes; decode the delivered file. For the lane in general, read the riddle for the codec, confirm
with the decoded text's template, and use one shared decoder across all three modes so Mastery is
mostly a loop.

## Evidence

Leaf base: `challenges/day11/warmups/warmups-day-11/`

- `single-solve/journal.md`, `single-solve/blob.txt` (no result file; `progress.md` row marks it
  Solved, journal records `CORRECT`)
- `time-trial/journal.md`, `time-trial/start_response.json`, `time-trial/artifacts/answer_map.py`,
  `time-trial/ranked-1/result.json`
- `mastery/journal.md`, `mastery/start_response.json`, `mastery/artifacts/answer_map.py`,
  `mastery/ranked-1/result.json` through `mastery/ranked-4/result.json`
- `library/warmups/codec-chain/codec_chain.py` (strict `base64` layer, `DEFAULT_RECORD_MAP`) and
  library entries `WARMUP-TEXT-CODECS-001`, `WARMUP-PLAYBOOK-001`
- `progress.md` (grep `^| 11 .*warmups-day-11`)
