# Hook Line and Signal

| | |
|---|---|
| Day / Lane | 23 / Mobile Android |
| Challenge ID | `mobile-android-day-23-hook-line-and-signal` |
| Technique | answer computed on-device by the APK's own bytecode (Dalvik interpreter), with decoy methods and a decoy secret |
| Single Solve | Local mark only (progress row CORRECT on attempt 1, plus a ledger entry; the leaf has no journal or result file) |
| Time Trial | Independent |
| Mastery | Independent |

## Summary

The prompt is "Separate real instrumentation signals from generated false positives", and the APKs
are built to match: decoy secrets and never-called helper methods surround one real answer path. The
Single Solve was easy because the answer sat XOR-encoded in an asset file. Time Trial and Mastery
switched to a new asset schema (`huntress.mobile.puzzle.v2`, `codec: "analysis"`) that stores no
answer at all. The value is computed at runtime by `Release.code()`, so we wrote a small Dalvik
bytecode interpreter and ran that method's own instructions. It got the Time Trial and all seven
Mastery builds CORRECT on the first submission each.

## The challenge

Every mode asked for answers of the form `ANS-` plus exactly 16 uppercase hex characters, with
`exact` normalization: "Submit only the exact backend answer code returned for this case."

- **Single Solve:** one 12.7 KB APK, one answer id (`answer`).
- **Time Trial:** one 12.7 KB APK from a fresh variant, one answer id.
- **Mastery:** seven APKs under one attempt, answer ids `set1.answer` .. `set7.answer`. They
  reused earlier curriculum days (the `day` field inside each build's own asset reads 2, 6, 10,
  14, 15, 22, 23) under the Day 23 mechanic.

Each APK had a deep-link activity (`huntress23://hook-line-and-signal/unlock`, exported, no
permission) and a `client_secret=SECRET-...` constant in the DEX that the activity sends beside a
`proof=` value. Neither is the validator answer.

## Solution

### Single Solve (schema v1)

1. **Recon.** The APK's `assets/huntress_puzzle.json` is a 108-byte file:
   `{"schema":"huntress.mobile.puzzle.v1","day":23,"codec":"xor","data":"<base64>","key":81}`.
   `classes.dex` only forwards a value as `proof=` next to the decoy `SECRET-...` string.
2. **Decode.** Base64-decode `data` and XOR every byte with 81. The result is
   `ANS-75C47B4BF8B5B1C1`, which matches the answer pattern. The decode used the workspace's asset
   codec reader, from the APK bytes whose SHA-256 matched the start response.
3. **Submit.** One call, recorded CORRECT in `progress.md` and in the submission ledger
   (2026-10-23 23:36). The leaf kept the extractor (`answer_map.py`) and its output but no journal
   and no graded `result.json`, so we label this mode "Local mark only".

### Time Trial (schema v2, `codec: "analysis"`)

1. **The tool refused, correctly.** The asset JSON now carried `schema: huntress.mobile.puzzle.v2`,
   `codec: "analysis"` and a `build` id, with no `data` or `key`. The asset-codec reader reported
   an unsupported codec. That is a different mechanic, not a decoding bug.
2. **Find the real path.** APK triage showed `Release.code(Context)` building the answer in four
   stages:
   - `token()` is `HookScan.mixPatcheb(Util.scanSignal61(Util.mixTrampoline79(SEED)))`, where
     `scanSignal61` calls `Probe.s2`/`Probe.s3` and `HookLineAndSignal.instrumentationSignals`;
   - the token goes through `Mix.f1`, then `Util.finTrampoline2d`;
   - the result is XORed with a 64-bit constant;
   - `"ANS-"` is appended and 16 hex digits follow, built nibble by nibble (shift 60 down to 0)
     through a `"0123456789ABCDEF"` table. Zero padding is therefore built in.
3. **Identify the decoys.** `TrampolineMap.sealTrampolinec7`, `HookScan.sealHook1e` and
   `Util.sealHook30` exist but nothing in that chain calls them. `Release.proof()` is a second
   decoy: it hashes the raw `token()` with `Long.toHexString`, with no XOR fold and no padding, so
   it can be shorter than 16 digits. `ChallengeActivity.onCreate` calls both `code()` and
   `proof()`, and only `code()` is the answer.
4. **Execute rather than hand-decode.** We wrote `dalvik_mini_vm.py`, an interpreter for the
   int/long arithmetic, `invoke-static/virtual/direct`, and a small String/StringBuilder/Long
   model, on top of the library's DEX reader. It runs `Release.code()`'s own bytecode end-to-end,
   so no constant, loop bound or digit order is copied by hand:

   ```python
   vm = DalvikVM(load_classes_dex(apk_path), native_libs=load_native_libs(apk_path))
   owner = find_release_owner(vm)            # the class named .../Release;
   code_full = "%s->code(Landroid/content/Context;)Ljava/lang/String;" % owner
   result = vm.run(code_full, [0])           # Context is never dereferenced
   ```

   An independent by-hand trace of the same bytecode gave the identical final value before we
   submitted. The submission was CORRECT on attempt 1; the journal puts time to CORRECT at about 47
   minutes, most of it writing the interpreter.

## Time Trial and Mastery

- **Time Trial** went CORRECT on call 1 in an untimed attempt and again in a ranked rerun (50 s
  window, 3.35 s end to end) using the same interpreter, by then promoted into the library.
  Ranked runs were bragging rights only.
- **Mastery** delivered seven APKs with three internal shapes, all with `codec: "analysis"`:
  - **plain chain** (d06, d22, d23): the same shape as the Time Trial build.
  - **table mux** (d02, d10, d14): two `int[8]` arrays built with `new-array`/`aput`, then read
    with `aget` under a data-dependent `if-nez` branch, roughly "use `table_b[i]` if non-zero, else
    `table_a[i]`", before folding into the accumulator. We added array ops to the interpreter.
  - **native hop** (d15): one step in the chain, `Util.nativeSecret03`, calls
    `NativeLibrarySecrets.nativeShare`. It is a real JNI export in `lib/x86_64/libarena.so`, an
    88-byte branch-free x86-64 function (add, xor, rol, shift, imul). The interpreter resolves the
    export from the dynsym table, disassembles it with capstone, and runs it with a small
    register interpreter. We cross-checked it against a hand-written Python replica of the same
    disassembly. This is why the mapper runs under `python -X utf8 -P` rather than `-I`.
  - The mapper takes `answer_ids` and each question's delivery filename from `start_response.json`
    and runs every APK through the same `run_code`. The full 7-field map was CORRECT on
    attempt 1, and a ranked rerun with the same solver also went CORRECT on call 1 (230 s window,
    5.06 s end to end).

## Dead ends and gotchas

- **The decoys are the point.** `ChallengeActivity` hands `proof()` and a `SECRET-...`
  `client_secret` to a backend-style URL, and a reader who stops at the activity's strings submits
  neither the right value nor the right shape. The never-called `seal*` helpers read like
  instrumentation checks: the journals counted 24 across the eight v2 builds, and none ran.
- **Single Solve technique does not transfer.** A mapper proven on the v1 `xor` asset gave nothing
  for the v2 builds, which have no stored data to decode. The schema is in the asset JSON, so
  dispatch on it before choosing a method.
- **Padding.** `Release.code()` always emits 16 digits; `proof()` does not. The format check
  `ANS-[0-9A-F]{16}` catches a `proof()`-style value.
- No wrong submissions were recorded for either mode. The journals do not say whether the
  Single Solve mapping was ever tested against the validator before its one CORRECT call.

## Takeaways

When an asset schema carries a `codec` but no payload, the answer is computed by code. Start at the
method the entry activity treats as the answer path (here `Release.code()`), then interpret its
bytecode instead of hand-decoding constants. That keeps the solver correct when the next build
randomizes constants, class names or digit order. Use the executed-versus-never-executed split as a
decoy filter: a method that nothing on the answer path calls is a false positive however much it
looks like a hook check. For a bundle, expect different obfuscation shapes per member (array
tables, a JNI hop) and add each opcode or native routine to the interpreter when it first appears.
Technique names to search: Dalvik bytecode interpretation, `codec: "analysis"` mobile puzzles, JNI
export emulation.

## Evidence

Leaf base path: `challenges/day23/mobile-android/mobile-android-day-23-hook-line-and-signal/`

- `single-solve/start_response.json`, `single-solve/artifacts/answer_map.py`,
  `single-solve/artifacts/asset_codecs_output.json`, `single-solve/artifacts/apk_triage_findings.txt`
  (no journal or result file in this mode)
- `time-trial/journal.md`, `time-trial/artifacts/dalvik_mini_vm.py`,
  `time-trial/artifacts/submission-attempt-log.json`, `time-trial/untimed-1/result.json`,
  `time-trial/ranked-1/result.json`
- `mastery/journal.md`, `mastery/artifacts/answer_map.py`, `mastery/artifacts/dalvik_mini_vm.py`,
  `mastery/artifacts/submission-attempt-log.json`, `mastery/ranked-1/result.json`
- Library: MOBILE-ANDROID-DALVIKVM-001 (Dalvik mini-VM), MOBILE-PLAYBOOK-001 (lane evolution)
- `progress.md` (grep `^| 23 .*hook-line-and-signal`)
- No `_run` librarian note or lane log exists for this challenge (the Day 23 `_run/` folder covers
  only the warmups and web lanes).
