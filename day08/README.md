# Day 8 overview

**At a glance:** 5 challenges, 15 modes (Single Solve, Time Trial and Mastery for each). Provenance by mode: Independent 15, Journal-recorded 0, Local mark only 0, Team 0, n/a 0. Theme: a recap day. Four of the five lanes re-asked earlier days' techniques (HTML entities, ZIP/JPEG/PNG container fields, a reversed-string APK asset, an ELF/PE configuration record) at larger scale, and the real work was in the validator's rules and in Mastery fixtures that did not match the Single Solve ones.

Day 8 predates this repository's version control (history starts 2026-10-22) and was played before release automation existed. The Single Solves were solved on 2026-10-20 (UTC timestamps in our saved graded responses). Script Analysis Time Trial and Mastery were solved the same day. The other Time Trial and Mastery modes are dated 2026-10-23 in our progress rows.

## Challenges

| Lane | Challenge | Technique | Single Solve | Time Trial | Mastery |
|---|---|---|---|---|---|
| Warmups | [Entity Relations](warmups_entity-relations.md) | HTML numeric character references (`&#NN;`); Mastery members are 8-record codec chains | Independent | Independent | Independent |
| Metadata Forensics | [What's in the Box?](metadata-forensics_whats-in-the-box.md) | Byte-level container fields: hashes, JPEG SOF, PNG IHDR, PDF header, ZIP EOCD and central-directory paths | Independent | Independent | Independent |
| Script Analysis | [Lost in Translation](script-analysis_lost-in-translation.md) | Static decode of encoded descriptors in 75 inert scripts (about ten languages), then validator-specific role / ATT&CK / control-wrapper rules | Independent | Independent | Independent |
| Mobile Android | [Cleartext Session](mobile-android_cleartext-session.md) | `reverse` codec in `assets/huntress_puzzle.json`; the cleartext-traffic story is a decoy | Independent | Independent | Independent |
| Binary RE | [Swift Justice](binary-reverse-engineering_swift-justice.md) | Recap bundle (d01-d08); CRC-checked HCFG record carved from a Base64 run; ELF/PE mix | Independent | Independent | Independent |

What backs each label (details are in each page's Evidence section):

- Every Single Solve has our own saved graded response reading `CORRECT` (warmups `submission-01.json`, the metadata, mobile and BRE submission responses, and for Script Analysis `submission-60-response.json`).
- Script Analysis Time Trial and Mastery have saved per-request responses for the original untimed attempts, plus ranked-N `result.json` files.
- For the other Time Trial and Mastery modes, the original 2026-10-23 attempt is recorded only in the leaf journal. The label is Independent because each of those modes also holds our own `ranked-N/result.json` reading `CORRECT`, from the later ranked sweep. The page rows say which evidence backs which attempt.

## What the day had in common

- **The prompt describes one thing, the data asks another.** Mobile's cleartext replay story never mattered (the answer is a reversed asset string). Binary's title and prompt describe only the Swift d08 specimen, but the 80 questions cover d01-d08. Metadata's "impossible provenance claim" had no matching question in any mode. Script's "find the one that doesn't match" had no single odd-one-out answer. Read the answer IDs and formats before the prompt.
- **Mastery is not N copies of Single Solve.** Warmups Mastery members were 8-record chains, not bare HTML entities. Binary Mastery mixed PE and ELF within the same slots (sets 1, 3, 5 had a PE32+ d02 and d04; sets 2, 4 had ELF64). Metadata and mobile Mastery key everything on the member or package named by each question, not on ZIP order. Every extractor had to be driven by the delivered bytes and by the question's own artifact field.
- **Decoys sit next to the real value.** The CRC-invalid literal `HCFG` in the ELF, the mobile `client_secret` beside `proof=`, and (in Script Analysis) rubric text that the validator did not follow.
- **Recap bundles reuse earlier fixtures only partly.** Binary Single Solve's d01-d06 members were byte-identical to Day 6's, but Time Trial drew different bytes for most days. Carry an earlier value over only after comparing SHA-256.

### Validator and format traps seen on this day

- **String-only answers.** The native-typed first submission was rejected with `invalid_answers` in Metadata Single Solve and again in the Binary Single Solve 70-field call. Nothing was graded but both calls counted.
- **`incorrect` is batch-level only.** Script Analysis needed 60 Single Solve requests, mostly single-field retries, because rejections carry no field list. Binary found the PE bug by bisecting on `dNN_` prefixes, which was cheap because the IDs are grouped.
- **Script rubric versus prompt.** Batch one-item `FOR ... CALL` controls were accepted as `call_subroutine` where the prompt text says `for_once`. For host/PHP/HTA specimens the descriptor's own `attack` value won over the graph reading, and Python launcher/relay/chain attacks were accepted as the parent `T1059`. The Script page documents the individual fields.
- **Per-delivery spellings.** Binary d04 `architecture` is `x86-64` for an ELF and `AMD64` for a PE. Other accepted spellings from the Binary page: `ELF`, `little endian`, `PE32_CLR`.
- **Transient 503.** Mobile Time Trial's first submit and Mastery's `start` returned HTTP 503. It is not a verdict, and resubmitting the same answer worked.
- **Deadlines on dual routes.** The original Time Trial attempts were untimed for most lanes, but Binary Time Trial enforced a window of about 80 s even without a timed request, and two Binary attempts expired before a submit landed.
- **ZIP member-name order.** The central-directory-order answer was accepted on Day 8, but a later review found every nested ZIP from Day 8 to Day 22 had central-directory order equal to sorted order, so the acceptance could not tell the two rules apart. Day 23 did discriminate and accepted sorted order. This did not change Day 8's result.

## Mistakes in our own tooling and process

Collected here from the pages that deferred them.

| Challenge | Mistake | Cost and fix |
|---|---|---|
| Swift Justice (Time Trial) | The mapper read C++ typeinfo and vtable predicates only from the ELF `symtab` key. In one delivery d02 was a native PE, whose names live in the COFF symbol list. Both predicates came out `false`. | Full batch `incorrect`; bisecting by day prefix isolated `d02_`. Two attempts expired (one before the first submit, one during bisection), so the mode took 12 submission calls. Fixed by falling back to COFF symbols. The BRE page says the same mistake was independently made on Day 9 Mastery. |
| Swift Justice (Single Solve) | The 70-field call sent non-string values. | `invalid_answers`, nothing graded, one call spent. |
| What's in the Box? (Single Solve) | Answers were sent as native JSON types (ints, a bool, a list). | `invalid_answers` on submission 1; submission 2 (strings) was CORRECT. |
| Lost in Translation (Single Solve) | The first move was submitting all 615 fields at once. Request 26 was posted with an empty body (the proxy answered `challenge_unavailable`) and still counted against the cap. | The Single Solve took 60 requests. The journal shows the submission cap was extended in steps (20, then 40, 50 and finally 60) by the player. The ledger rows are in the journal. |
| Lost in Translation (Time Trial) | The map had the confirmed overrides pre-applied, but was still sent in coarse groups. | 5 of 18 requests came back `incorrect`. |
| Lost in Translation (Time Trial, Mastery) | No contemporaneous journals were kept for these two modes. | The 2026-10-20 audit reconstructed both journals from saved request/response files and added the progress rows. |
| Cleartext Session (Single Solve) | The library's APK asset inspector reported `reverse` as an unsupported codec, so the value was read by hand. | No attempts wasted. `reverse` has since been added as a named codec (`library/mobile-android/asset-codecs/asset_codecs.py` lists it, with Day 8 mapped to it). |
| Entity Relations (Mastery) | The first Mastery journal described the members as single-codec HTML blobs. The members are 8-record chains. | The mapper was right only because the shared codec tool detects both shapes; the journal was corrected later. |
| What's in the Box? | The library rule for ZIP member-name order was written from acceptances that could not discriminate. | Revised after Day 23; no effect on Day 8. |
| Swift Justice (ranked sweep) | The first ranked Time Trial attempt (2026-10-26) was lost: the wrapper crashed mid-download of the 31 MB archive and the attempt expired unscored (`ranked-1` holds only a start response). | One ranked attempt lost. `tools/ranked/PROCESS.md` records the fix (keep-alive sockets closed after 2 s idle, a POST retried only on a pre-response disconnect, `download_failed` reported as JSON). The mode was rerun with the player's OK. |
| Swift Justice (ranked sweep) | Two 2026-11-01 attempts (Time Trial ranked-6, Mastery ranked-5) failed before submitting. The solver reported that AES-encrypted evidence needed `pyzipper`. | Nothing was submitted. The same solver succeeded about two minutes later (ranked-7, ranked-6). The logs say pyzipper was not importable in that run, but we could not determine why from the files. |

## Tooling and process changes around this day

Day 8 predates version control, so nothing before 2026-10-22 has commits. These are the changes the repository and the journals support:

- **2026-10-22 (`265e9a63`, "Add Day 8 challenge work"):** the first commit of the Day 8 leaves. It adds challenge material only, not tooling.
- **2026-10-23 (`1d82b9d5`, Days 5, 7, 8, 10, 11 Time Trial/Mastery backlog):** the commit message names two relevant changes. `tools/gm.py artifact` gained `--capability`/`--name` so a Mastery bundle delivered as several same-named top-level artifacts can be fetched delivery by delivery; previously only the first was reachable. The commit message does not say which day prompted it, but Day 8 BRE and mobile Mastery are of that shape (the BRE leaf holds `set1.zip` to `set5.zip`, the mobile leaf five APKs). The same commit records the Day 8 Swift Justice PE/COFF mapper fix described above.
- **Library entries named in the journals:** Script Analysis Single Solve's journal says its decoders and rubric were generalized into `material_codec_decode.py` and `behavior_rubric.py`; both files exist in `library/script-analysis/`. The Warmups journal wrote up HTML character references as a new technique entry (`WARMUP-HTML-ENTITIES-001`), and the Metadata journal revised the ZIP member-path rule (`META-ZIP-MEMBER-PATHS-001`).
- **Ranked sweep of Days 1-10 (2026-10-25/26, `1df4ec2c`):** added `tools/ranked/solvers/` for this day's Metadata, Mobile, Script and BRE challenges (Warmups has no per-challenge solver here; it goes through the shared codec-chain tool), plus case lists (`tools/ranked/cases/bre-d06-10.json`, `metadata-d06-10.json`, `mobile-d06-10.json`, `scriptA-d06-08.json`) and `tools/ranked_pass.py` route auto-detection. `PROCESS.md` lists Day 8 warmups, metadata, mobile and script as dual-pool (`run_for_time:true` is the ranked call) and Day 8 BRE Mastery as legacy (a plain `retry` is the ranked attempt). The result files in each leaf show `route: dual` for the first four lanes and `route: legacy` for BRE Time Trial and Mastery. Each non-BRE lane went CORRECT in one call, 1.4 to 8.3 s end to end.
- **Later ranked reruns (2026-10-31 and 2026-11-01, `021814b6`, `6af31628`, `38668151`, `f0e20aa1`):** commit messages describe these as gap-closure sweeps toward the speed scoreboard. The leaves hold the extra `ranked-N` folders: Script Analysis Mastery ranked-2/-3, Warmups Mastery ranked-2 to -4, Script Time Trial ranked-2, BRE ranked-3 to -7 and Mastery ranked-2 to -6. These add evidence but do not change what the original attempts were.

## Effort

Submission-call counts for the original attempts, from the journals and progress rows. These are not times.

| Challenge | Single Solve | Time Trial | Mastery |
|---|---|---|---|
| Entity Relations | 1 | 1 | 1 |
| What's in the Box? | 2 (first `invalid_answers`) | 1 | 1 |
| Cleartext Session | 1 | 2 (first a 503) | 1 |
| Swift Justice | 3 (first `accepted` for d08 only, second `invalid_answers`) | 12 (three attempts, two expired) | 1 |
| Lost in Translation | 60 | 18 | 5 (one per set) |

The Script Analysis solves have a minute-level record, because the saved graded responses carry UTC timestamps (all 2026-10-20). Single Solve `CORRECT` at 05:41, the untimed Time Trial attempt started 05:45 and was `CORRECT` at 05:49, and the Mastery was `CORRECT` at 05:52. The other Single Solves were graded at 04:05 (warmups), 04:06 (metadata), 04:08 (mobile) and 04:10 (BRE). These record when the graded response arrived, not how long the solve took, and we have no start times for them.

Ranked reruns, as recorded in the leaves' `result.json` files (all 1 call, `CORRECT`): Warmups Time Trial 1.6 s, Warmups Mastery 2.0 to 4.9 s, Metadata 1.6 and 1.8 s, Mobile 1.4 and 2.3 s, Script Time Trial 1.9 and 2.6 s, Script Mastery 3.3 to 8.3 s, Binary Time Trial 5.8 to 10.4 s, Binary Mastery 8.5 to 22.1 s. BRE is slow mainly because of the 31 MB download.
