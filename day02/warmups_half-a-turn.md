# Half a Turn

| | |
|---|---|
| Day / Lane | 2 / Warmups |
| Challenge ID | `warmups-day-02` |
| Technique | ROT13 (Caesar shift 13) over a short text blob; numbered-record codec chain in Mastery |
| Single Solve | Solved, graded result not preserved (local mark only) |
| Time Trial | Independent (CORRECT on the 2026-10-20 audit attempt and on four later ranked attempts) |
| Mastery | Independent (CORRECT on the 2026-10-20 audit attempt and on two later ranked attempts) |

## Summary
A one-line ROT13 warmup. The prompt says "Thirteen steps forward. Funny how that works out.", which is the whole
hint: shift every letter by 13 and the blob tells you which slug to submit. Single Solve was one blob, Time Trial
the same shape on a ~20 s clock, and Mastery eight blobs (eight answers) on a ~25 s clock that needed a small
per-record decoder instead of a one-liner.

## The challenge
- Prompt: "Thirteen steps forward. Funny how that works out." Question: decode the assigned blob completely and
  submit the exact slug that its final non-empty decoded line instructs you to submit.
- Answer format: three lowercase words separated by hyphens, then a hyphen and a two-digit number
  (`word-word-word-NN`), slug only.
- Single Solve: one text file (`blob.txt`), one answer field (`message`).
- Time Trial: one blob, one answer field (`message`).
- Mastery: eight blobs (`set1.message` ... `set8.message`), one slug each.

## Solution
1. Look at the blob. The Single Solve text starts `Avpr jbex qrpbqvat guvf jnezhc!`, an obvious letters-only
   substitution with the word shapes of English ("Nice work decoding this warmup!"). Together with "thirteen
   steps" in the prompt, that is ROT13.
2. Apply ROT13 to the whole file. The last line reads, in clear text, "...submit this exact answer to the Game
   Master: plucky-clock-flint-94". The slug is the last whitespace-separated token of the last non-empty line.
3. Submit that token as-is under the answer ID from the start response (`message`). The answer ID always comes from
   the start response, not from the file name.

The extractor used for Time Trial is four lines of stdlib Python:

```python
import codecs, sys, json
t = open(sys.argv[1], encoding='utf8').read()
lines = [l for l in codecs.decode(t, 'rot13').splitlines() if l.strip()]
print(json.dumps({"message": lines[-1].rsplit(' ', 1)[-1].strip()}))
```

The Single Solve leaf holds only the blob and the saved start response; there is no solver or submission record
for it. Re-decoding the saved blob with the same four lines yields a slug matching the required pattern, but the
graded result for this mode was recorded as a local mark by the player, so it is not claimed as an independently
preserved CORRECT.

## Time Trial and Mastery
- **Short windows.** Neither mode was untimed in practice. A plain `retry` on this day is already a ranked
  attempt (`hall_of_fame_eligible: true`) with a server-set deadline: 20 s for Time Trial and 25 s for Mastery.
  Download, decode and submit therefore had to be scripted in one pass; the saved ranked-pass logs show a
  total of 1.3 to 3.3 s for Time Trial and about 4.3 to 4.8 s for Mastery (Mastery is dominated by the artifact
  download).
- **Time Trial** reused the Single Solve decoder unchanged. One submission each time: the first audit attempt on
  2026-10-20 and four later ranked attempts (2026-10-26) were all CORRECT on call 1.
- **Mastery** was a bundle of eight `blob-NN.txt` files, each line `NN<TAB>chunk`. Each blob had two numbered
  records, one base64 and one ROT13 (per the journal). The records are decoded individually, concatenated in
  record order, and the last token of the last non-empty line is the slug. The first version of the mapper guessed
  the codec per line (hex, then base64, then ROT13 fallback) and was CORRECT on 8/8 fields in one call. On the
  later ranked attempts the same eight blobs arrived as eight separate `blob.txt` artifacts rather than one ZIP;
  both ranked Mastery attempts were CORRECT on one call each.

```python
def dec(c):
    if re.fullmatch(r'(?:[0-9a-fA-F]{2})+', c):          # hex chunk
        try: return bytes.fromhex(c).decode('utf8')
        except Exception: pass
    if re.fullmatch(r'[A-Za-z0-9+/]+={0,2}', c) and len(c) % 4 == 0:   # base64 chunk
        try:
            s = base64.b64decode(c, validate=True).decode('utf8')
            if s.isprintable(): return s
        except Exception: pass
    return codecs.decode(c, 'rot13')                       # otherwise ROT13
```

## Dead ends and gotchas
- **The clock.** The start call without a timing flag still returned a short ranked window on Time Trial and
  Mastery, so a "rehearsal" attempt is a real attempt (see the day overview for the retry-budget consequences).
- **Per-line codec guessing is a heuristic.** Hex and base64 alphabets can overlap with plain text, so
  auto-detection can in principle misfire. It did not here, but the journal records that the generalized tool later
  switched to a record-number table as the more reliable selector.
- **Slug extraction.** The answer is the last token of the last non-empty line, not the first slug you see in the
  decoded text.

## Takeaways
- When the prompt names a number ("thirteen") and the text looks like letter-substituted English, try ROT13 before
  anything else. Python's `codecs.decode(t, 'rot13')` is enough.
- For multi-record warmups, decode per numbered record and concatenate in order instead of auto-detecting a
  single codec for the whole blob (technique: numbered codec chain; the library's `codec_chain.py` now covers it).
- When an untimed-looking mode returns a deadline, script start, download, decode and submit as one run.

## Evidence
Leaf base path: `challenges/day02/warmups/warmups-day-02/`

- `single-solve/journal.md`, `single-solve/blob.txt`, `single-solve/challenge.json` (start response)
- `time-trial/journal.md`, `time-trial/artifacts/answer_map.py`, `time-trial/artifacts/answers.json`,
  `time-trial/retry_response.json`; ranked results in `time-trial/untimed-{1,2,5,6}/result.json` (CORRECT; the
  directory name is a scripting mislabel, those were ranked attempts)
- `mastery/journal.md`, `mastery/artifacts/answer_map.py`, `mastery/artifacts/answers.json`,
  `mastery/start_response.json`; ranked results in `mastery/ranked-1/result.json` and `mastery/ranked-2/result.json`
  (both CORRECT, 8 answers, 1 call)
