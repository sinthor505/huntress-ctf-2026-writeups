# Dot Com

| | |
|---|---|
| Day / Lane | 16 / Warmups |
| Challenge ID | `warmups-day-16` |
| Technique | Morse code whose symbols are hex digits (Single Solve / Time Trial); 16-record numbered codec chain (Mastery) |
| Single Solve | Independent (saved submission response reads CORRECT, attempt 1) |
| Time Trial | Independent (untimed attempt CORRECT on call 1; two later timed `ranked-N/result.json` files also CORRECT) |
| Mastery | Independent (untimed attempt CORRECT on call 1; later timed `ranked-1/result.json` CORRECT) |

## Summary

"Dot Com" hides a one-line instruction ("submit this exact answer to the Game Master: ...") under
Morse code. The Morse alphabet is used only to spell the 16 hexadecimal digits, so the blob is
Morse -> hex -> UTF-8 text, and the answer slug is the end of the decoded line. Mastery wrapped the
same idea in 78 blobs of 16 numbered records each, one codec per record number. All three modes were
CORRECT on the first submission.

## The challenge

Prompt: "Sixteen little symbols are calling. Dot dot dot." Each mode delivered a static blob (no
live target) and asked one question per blob: decode it completely and submit the slug its final
non-empty decoded line instructs. The answer format is a pattern: three lowercase words separated by
hyphens, then a hyphen and a two-digit number, "slug only".

- **Single Solve / Time Trial:** one blob, one answer id (`message`). The blob is a single line of
  Morse symbols separated by spaces (about 1.26 KB), starting `....- . -.... ----.`.
- **Mastery:** a ZIP of 78 blobs (`blob-01.txt` ... `blob-78.txt`), 78 answer ids
  (`set1.message` ... `set78.message`), each blob is 16 lines of `NN<TAB>payload`.

"Sixteen" in the prompt is literal: it is the number of records in each Mastery blob, and the
sixteenth record is the Morse-of-hex layer.

## Solution

1. **Recon.** The first tokens `....-` `.` (4, E) then `-....` `----.` (6, 9) are digits and
   letters A-F only, and no token had a space-separated word break. That points to Morse used as a
   hex alphabet, not Morse spelling English. Decoding it as text would give a long run of digits
   and letters, not a sentence.
2. **Morse -> hex.** A 36-symbol Morse table (the 16 hex symbols are enough, but the full table was
   used) maps each token to one hex character. The key detail is that `E` is a single dot.
3. **Hex -> text.** `bytes.fromhex(...)` then UTF-8 gave:
   `Nice work decoding this warmup! To solve, submit this exact answer to the Game Master: silly-banner-marble-88`.
4. **Answer.** The mapper takes the last non-empty line and extracts the last match of a
   `word-word-word-NN` regex, so a slug-shaped word earlier in the sentence could not be picked:

   ```python
   SLUG = re.compile(r"(?<![a-z0-9])([a-z]+-[a-z]+-[a-z]+-[0-9]{2})(?![a-z0-9])")
   text = binascii.unhexlify(morse(raw)).decode()
   line = [l for l in text.splitlines() if l.strip()][-1]
   print(json.dumps({"message": SLUG.findall(line)[-1]}))
   ```

   The accepted Single Solve answer was `silly-banner-marble-88` (the Game Master returned CORRECT on
   attempt 1 of the 20-submission budget).

## Time Trial and Mastery

**Time Trial** used the identical codec on a fresh blob; the same mapper ran unchanged and was
CORRECT on call 1 (untimed attempt, no deadline in the start response). The later timed attempts
(30 s window) were CORRECT on call 1, with local wall times of about 4.0 s and 1.8 s from issue to
CORRECT, dominated by the network calls rather than the 0.01-0.03 s solve.

**Mastery** is the part that changes shape. Each blob has exactly 16 numbered lines, and the record
number fixes the codec. Decoded fragments are concatenated in order, and the answer is the text after
the last `:` of the concatenation. The sixteen codecs, in record order:

| Record | Codec | Record | Codec |
|---|---|---|---|
| 1 | base64 | 9 | JSON string with `\uXXXX` escapes |
| 2 | ROT13 | 10 | base32 |
| 3 | hex | 11 | base64 |
| 4 | binary bytes | 12 | reversed text |
| 5 | decimal bytes | 13 | Atbash |
| 6 | octal bytes | 14 | Caesar, shift 7 |
| 7 | percent-encoding | 15 | ROT47 |
| 8 | HTML numeric references | 16 | Morse-of-hex (as above) |

The numbering is the same as the first 16 records of the Day 19 chain, so the mapper was the library's
numbered-codec-chain script with the expected record count changed (its docstring still says 19). Two
checks keep it honest: it verifies the archive SHA-256 against the start response and requires the
ZIP members to match the questions' `artifact` fields (each answer id is resolved from its own
blob name, not from order), and it rejects any blob whose result does not fully match the slug
pattern. All 78 slugs were submitted in one call and accepted. A later timed Mastery attempt (95 s
window, fresh 78-blob bundle) was also CORRECT on call 1 using the repo's generic warmups solver with
no changes; solve time was about 1.9 s of a 6.3 s total.

## Dead ends and gotchas

- **Morse output is not the message.** It is hex. A solver that stops at the Morse layer gets a
  digit-and-letter string; the second decode to bytes is what reveals the English sentence.
- **Use the full Morse table, and treat an unknown token as "wrong codec".** Substituting `?` for
  unknown symbols (as the plain-Morse helper in the leaf does) would hide that the blob is not this codec.
- **Unpadded base64.** Record 11 in the Mastery blobs is base64 without `=` padding
  (`dG8gdGhlIA`), so the mapper pads to a multiple of 4 before decoding.
- **Fragments are mid-word.** Each record carries a few characters (`Nice w`, `ork dec`, ...), so
  decoding a single record gives nothing readable; the slug is only visible after concatenation.
- **Record 16's UTF-8 hex** worked as expected here; a result starting `fffe`/`feff` would be UTF-16
  (Day 19 record 19), not UTF-8. It did not occur on Day 16.
- No format surprises: the plain lowercase slug (no braces, no `flag{}` wrapper) was accepted by
  all three modes.

## Takeaways

- **Technique:** "Morse of hex digits" is a recognizable shape. A Morse stream whose decoded
  characters are all `0-9A-F` is a hex string, not text; chain a second decode.
- For numbered-record warmups, the record number is the codec selector. Build (or reuse) a
  per-record table once and reuse it across days, because later days extend the same table: Day 16
  is the first 16 entries of Day 19's chain, so its mapper only needed a count change.
- Faster next time: run the repo's generic codec-chain tool on the blob first. Later timed
  attempts showed it decodes both the Single Solve blob and the Mastery bundle with no changes.

## Evidence

Leaf: `challenges/day16/warmups/warmups-day-16/`

- `single-solve/journal.md`, `single-solve/start_response.json`,
  `single-solve/artifacts/decode.py`, `single-solve/artifacts/answer_map.py`,
  `single-solve/artifacts/submission-attempt-01-response.txt` (CORRECT)
- `time-trial/journal.md`, `time-trial/artifacts/submission-attempt-01-response.txt` (CORRECT),
  `time-trial/ranked-1/result.json`, `time-trial/ranked-2/result.json`
- `mastery/journal.md`, `mastery/start_response.json` (78 questions),
  `mastery/artifacts/answer_map.py`, `mastery/artifacts/submission-attempt-01-response.txt`
  (CORRECT), `mastery/ranked-1/result.json`
- Library entries: WARMUP-MORSE-HEX-001, WARMUP-NUMBERED-CODEC-CHAIN-001
- `progress.md` (grep `^| 16 .*warmups-day-16`)
