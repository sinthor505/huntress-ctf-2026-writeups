# Deep Link Drift

| | |
|---|---|
| Day / Lane | 2 / Mobile Android |
| Challenge ID | `mobile-android-day-02-deep-link-drift` |
| Technique | APK triage; static puzzle-asset decode (`reverse` codec) behind a decoy deep-link / `client_secret` surface |
| Single Solve | Solved (journal-recorded): CORRECT on 1 submission per the journal; no result file saved in the leaf |
| Time Trial | Independent (CORRECT, 1 submission; a second scripted ranked pass also CORRECT) |
| Mastery | Independent (CORRECT, 1 submission; a second scripted ranked pass also CORRECT) |

## Summary
The prompt dresses the challenge up as deep-link security ("tell verified routing apart from a scheme and host that wave everybody through"), but the answer never depended on routing. It sat in a bundled puzzle asset, stored reversed, and the manifest and DEX only supplied decoys. We decoded the asset offline and submitted once per mode.

## The challenge
Day 2 of the mobile lane. Each attempt hands you a small APK (12,704 bytes in the Single Solve and Mastery deliveries) and asks for "the exact backend answer code returned for this case". The case ID and package name (`com.huntress.mobile.d02.v<hash>`) are unique per attempt. The answer format is a pattern, `ANS-` followed by exactly 16 uppercase hex characters, with exact normalization.

- Single Solve: 1 APK, 1 answer (`answer`), untimed.
- Time Trial: 1 APK, 1 answer, server-enforced deadline.
- Mastery: 3 APK deliveries, 3 answers (`set1.answer` .. `set3.answer`), one per delivery.

## Solution
1. **Triage the APK.** The manifest declares an exported activity with a VIEW/BROWSABLE intent filter: scheme `huntress02`, host `deep-link-drift`, `pathPrefix=/unlock`. The journal records no validation code behind it. This is the "scheme and host that wave everybody through" the prompt talks about, and it is the theme, not the puzzle.
2. **Check the DEX.** It posts to a local `127.0.0.1` evidence endpoint that is unreachable from outside, and embeds a string of the form `client_secret=SECRET-...&proof=`. We treated that as app-side decoy material and did not use it.
3. **Find the real data.** The APK carries `assets/huntress_puzzle.json`. Its `codec` field is `reverse` and it holds one answer's worth of data. The `reverse` codec just reverses the Unicode string.
4. **Decode and validate offline.** A small mapper (`answer_map.py`) checks the APK's SHA-256 against the start response, runs the library's puzzle-asset inspector on it, requires exactly one supported asset, and checks that the decoded text matches `ANS-[0-9A-F]{16}` before emitting `{answer_id: value}` as JSON.
5. **Submit.** The Single Solve answer was `ANS-A7900B567A0EEA49` (strings only, as with every submission here). One call, CORRECT.

The mapper's guard is the part worth copying: it refuses to emit anything unless there is exactly one supported asset and the result fits the format regex.

```python
assets = json.loads(out)
if len(assets) != 1 or not assets[0].get("supported"):
    raise SystemExit("expected exactly one supported puzzle asset")
ans = assets[0]["decoded_text"]
if not re.fullmatch(r"ANS-[0-9A-F]{16}", ans):
    raise SystemExit("decoded asset does not match the answer format")
```

## Time Trial and Mastery
Nothing about the puzzle changed: same structure, same `reverse` codec, fresh case IDs and packages per attempt. The change was the clock and the bundle size.

- **Time Trial.** Our first attempt requested an untimed start, yet the server still returned a 50 second deadline. We downloaded, mapped and submitted in one scripted pass, about 5 seconds after the start. The single-APK mapper was reused unchanged.
- **Mastery.** The deadline was 110 seconds (30 s per set plus 20 s, consistent with the three-set bundle) and three APKs came back. The Game Master helper script downloads only the first artifact, so the other two were fetched from the capability URLs in `artifacts[]`. The Mastery mapper pairs delivery N with question `setN.answer` using the `delivery N: <file>` text in each question, verifies each APK's SHA-256, decodes each asset and submits all three in one call.
- **Ranked passes.** Both modes were later re-run as fully scripted ranked passes (per-challenge solver, all three or one answers in a single submission). Both returned CORRECT in one call, with a total pass time of 1.36 s (Time Trial) and 1.91 s (Mastery) on our side against 50 s and 110 s windows.

## Dead ends and gotchas
- **The prompt points at the wrong surface.** Everything about the wording and the manifest invites a deep-link exploit (crafting a `huntress02://deep-link-drift/unlock` intent, probing the local evidence endpoint, replaying `client_secret`/`proof`). None of it is needed; the answer is stored data in the asset. We ruled out the endpoint by reading the DEX rather than probing it.
- **A saved `bundle.zip` can differ from the API artifact.** The Single Solve `bundle.zip` copied from earlier handling had a different SHA-256 from the one in the start response. The cause was line-ending translation (LF to CRLF) of the binary, which grows a 12,704-byte APK by a few dozen bytes and breaks the ZIP central directory. We re-downloaded and compared against the API hash before analyzing. The hash check in the mapper exists for this reason.
- **"Untimed" is not guaranteed.** Requesting untimed play still returned 50 s (Time Trial) and 110 s (Mastery) windows on this day, so anything that touches the APK has to be scripted end to end.
- **An earlier "solved" mark was not a solve.** The challenge had been marked solved locally on a player's say-so before any graded result existed. Re-checking found no graded CORRECT, so the three modes were re-solved from freshly assigned, hash-verified bytes. The provenance above comes only from those re-solves.

## Takeaways
- When a mobile challenge's prompt is about an attack surface, still enumerate the APK's assets first. In this lane the answer was repeatedly in the declared puzzle JSON, with the manifest, DEX strings and `client_secret` acting as bait (see the "decoy `client_secret`" pattern).
- Treat the codec as fixed per release day (Day 2 is `reverse`); a generic asset-codec tool that reads the asset's own `codec` field handles it without a per-challenge script.
- For multi-delivery bundles, pair deliveries to answer IDs from the question text and the start response's per-artifact hashes, never by file order or name guesses.
- Do the whole download, decode and submit as one scripted pass; the window is under two minutes.

## Evidence
Leaf base path: `challenges/day02/mobile-android/mobile-android-day-02-deep-link-drift/`

- Single Solve: `single-solve/journal.md`, `single-solve/start_response.json`, `single-solve/artifacts/answer_map.py`, `single-solve/artifacts/candidate.json`
- Time Trial: `time-trial/journal.md`, `time-trial/start_response.json`, `time-trial/artifacts/answer_map.py`, `time-trial/ranked-1/result.json`, `time-trial/ranked-1/accepted_plan.json`
- Mastery: `mastery/journal.md`, `mastery/start_response.json`, `mastery/artifacts/answer_map.py`, `mastery/ranked-1/result.json`, `mastery/ranked-1/accepted_plan.json`
- Progress rows: `progress.md`, rows for Day 2 `mobile-android-day-02-deep-link-drift` (all three modes `Solved`, 2026-10-20, 1 submission each)
- Library techniques: asset codec tool (`reverse` codec), APK triage tool, mobile Android gotchas
