# SHA1-Hulud

| | |
|---|---|
| Day / Lane | 30 / Binary Reverse Engineering |
| Challenge ID | `binary-reverse-engineering-day-30-sha1-hulud` |
| Technique | self-decrypting VEH "walls" + MBA anti-analysis chains in a flag-checker crackme; static reimplementation |
| Single Solve | Team solve (not ours) |
| Time Trial | Absent (Single Solve only) |
| Mastery | Absent (Single Solve only) |

## Summary

SHA1-Hulud is the one flag-checker crackme of the event: a Rust PE64 that reads a line, pushes it through a chain of self-modifying SHA-1 variants, a 45-million-iteration mixer, anti-analysis arithmetic and a custom two-pass block cipher, and prints the input back if it passes. A teammate scored the Single Solve about three hours after unlock, so our own submission returned `already_solved` and the challenge could not be re-issued for a graded attempt. We did recover the flag offline from our own copy of the binary, after roughly four hours stuck on a wrong emulation-based theory, and checked it cryptographically. That derivation is ours but ungraded, and the graded solve was a teammate's.

## The challenge

Prompt: "I made this challenge because I was really upset that TeamPCP dropped the ball with this payload name. Anyway, here's Wonder Wall!" The archive password was `infected`. Delivered: `sha1_hulud.zip` (AES ZIP, needs `pyzipper`) holding one file, `sha1-hulud.exe`, PE32+ AMD64 built from Rust (MSVC). There was no question list: `answer_ids` was `["flag"]`, a flag-style Single Solve with one free-text answer.

Static triage showed:

- A `.xsobf` section (429 bytes) holding encrypted strings, with a descriptor-based decryptor. All 37 strings it protects turned out to be analysis-tool process names and registry paths: anti-analysis decoys, not the flag.
- A small XOR helper (`{src,len,key,len}`) that decodes the prompt `What is the flag?` and the success text `correct, the flag is `. The success path then prints the trimmed *input line*, so no flag is stored anywhere. The flag is whatever input passes the checker.
- A `VirtualProtect` call that makes `.text` RWX, arms a vectored exception handler, and then calls a compression routine at RVA `0x1080` that raises `int3` over and over while it decrypts itself.
- Anti-analysis reads of every flavour: `BeingDebugged`/`NtGlobalFlag`, `NtQueryInformationProcess` via API hashes, debug registers, `rdtsc` plus `KUSER_SHARED_DATA` timing, registry enumeration, `GlobalMemoryStatusEx`, `GetSystemInfo`, host and user name.

The binary was never run by the automation. Everything below was done by static reading and Unicorn emulation of individual functions.

## Solution

The final pipeline, as we rebuilt it in Python, is: code-region hash, post-mix, anti-analysis scalar `Q`, key schedule, bit-stream readers over two 16 KB blobs, two-pass cipher. Each step was validated against the emulator or against the binary's own data before the next was built on it.

1. **Isolate each wall.** The four self-decrypting functions at RVAs `0x1080`, `0x10840`, `0x1dfc0` and `0x2b840` all take `rcx -> {u32 h[5]; u32 pad; u32 w[16]}` and `rdx = all-ones`, and write the new state back in place with no feed-forward. Each raises one or more `int3`s that the VEH answers by setting `R15` to the `int3` address and resuming. We mapped the PE image into Unicorn, called the wall RVA directly, and answered every `int3` the way the VEH does:

   ```python
   while True:
       uc.emu_start(pc, 0)
       if self.ev[0] == 'done':            # hit the ret sled
           break
       _, intno, rip = self.ev
       assert intno == 3
       uc.reg_write(UC_X86_REG_R15, rip - 1)   # VEH convention: R15 = int3 address
       pc = rip                                # resume after it
   ```

   That turns a self-modifying region into an ordinary function that can be called on random `(h, w)` inputs. The harness is now `wall_isolate.py` in our library (entry BRE-MBA-FLAGCHECK-001); it never starts the PE as a process.

2. **Reimplement each wall in Python.** Every wall is an 80-round SHA-1 compression with one small deviation, and a model grid over message-schedule taps, `Ch` argument order, the `Ch`/parity boundary round and the per-round rotate of `b` reproduces the oracle exactly. In about 2 s the grid named all four:

   | RVA | Variant |
   |---|---|
   | `0x1080` | standard SHA-1 |
   | `0x10840` | schedule taps `(3, 9, 14, 16)` instead of `(3, 8, 14, 16)` |
   | `0x1dfc0` | `Ch` arguments swapped in rounds 0-19 |
   | `0x2b840` | `Ch` through round 21, `c <- rol(b, 2)` |

   ```python
   def W10840(h, w16):         # schedule taps (3,9,14,16)
       w = _sched(w16, (3, 9, 14, 16)); a, b, c, d, e = h
       for i in range(80):
           ...                 # standard round function
           e, d, c, b, a = d, c, rol(b, 30), a, t
       return [a, b, c, d, e]
   ```

   Each lifted function was checked against the Unicorn oracle on many random inputs, never one. As a sanity check, hashing `"abc"` with the `0x1080` model and the usual feed-forward add equals `hashlib.sha1`.

3. **Rebuild the code-region hash and post-mix.** The checker hashes its own mapped `.text` (516,096 bytes) plus a 68-byte PE-header trailer with the standard wall to get `K0`, then runs a self-mix over it to get `KA`. Both matched the values the emulator stored. A 44.9M-iteration mixing loop (`0x2aea540` iterations over `.text`, a multiply and xorshift on a 64-bit accumulator) was reimplemented as `mix45`:

   ```python
   for _ in range(COUNT):
       r10 = ((rcx >> 32) * r8) >> 32            # data-dependent offset into .text
       w8  = int.from_bytes(text[r10:r10 + 8], 'little')
       rcx = ((rcx ^ w8) * MUL) & M64
       rcx ^= rcx >> 29
   ```

   It was bit-exact against the emulator's recorded state before and after pass 1. The result is cached in `schedule.json`, since the loop is slow in pure Python.

4. **Collapse the MBA chains to clean-run constants.** This is the step that matters. The anti-analysis reads (debug registers, debugger flags, tick counts, memory size, CPU vendor, exec-memory checks) are not used as branches. They are folded through mixed-boolean-arithmetic chains into a single 64-bit scalar, `Q`, that is then mixed into the key schedule. On a clean machine each chain produces a fixed constant that depends on nothing about the host. A chain with the recovered linear shape `A + M*x == B + C (mod 2^64)` has the clean-run `x` directly from its own constants, with `M` odd and therefore invertible:

   ```python
   x = (B - A - C) * pow(M, -1, 1 << 64) % (1 << 64)
   ```

   Our solver did not apply that closed form chain by chain. It transcribed each chain as straight-line Python (`f63b2a`, `f63e18`, `f63ec6`, ...) and fed it the clean-run inputs, such as the debug-register block set to zero and no debugger, then combined them into `Q = f63d03(...)`. The closed form is the technique recorded in the library entry, which came from a teammate's notes (see Evidence); our own run is the straight-line version. Both give the same idea: evaluate on a clean run and stop pretending the machine matters.

5. **Derive the key schedule.** `K2 = hash(bufA || Q)` under the second wall variant and `K4` under the third give the keys for the two passes, each mixed again with `mix45`. A small parameter function turns each key into reader parameters (bit width, start offset, count, indices, rotate).

6. **Read the target streams.** Two 16 KB blobs at RVA `0x7fa50` and `0x83a50` are decoded by a hash-driven bit-stream reader: varint descriptors select a slice of a running SHA-1 state, XORed against bytes of `.text`. A 16-slot table selects which streams are live. Only "active" slots, those that share a tag, count, and the expected ciphertext for a candidate is `u3 ^ u4 ^ r6` from two slot readers and one record reader.

7. **Invert the cipher.** The expected ciphertext `E` is the output of a two-pass 128-bit block cipher: a forward CBC pass with per-block keys from a running SHA-1 state, then a backward reverse-CBC pass with a constant key. Each round is a T-function `y + (y*y | 5)`, rotates, and xorshifts, and every piece is bytewise invertible:

   ```python
   def inv_xsr(v, s):
       x = v
       for _ in range(64 // s + 1): x = v ^ (x >> s)
       return x
   def tf_inv(z):                      # solve y + (y*y | 5) == z one bit at a time
       y = 0
       for k in range(64):
           if (tf(y) ^ z) & ((1 << (k + 1)) - 1): y |= 1 << k
       return y
   ```

   The plaintext frame is `[u64 length][input bytes]`, zero-padded to 16-byte blocks, with the input trimmed and lowercased first.

8. **Cross the small search space and read off the answer.** The solver enumerated every active slot pair and every record, decrypted each, and kept anything printable. The only `flag{<32 hex>}` result came from slot pair `(8, 14)`, record 1, 3 blocks, length 38:

   `flag{3b483b1acf422ac4e167275f3b1d9be4}`

   The same machinery also decrypted the checker's other accepted inputs, both jokes: `flag{nice_try}` and `your flag is in another castle, sorry!`.

**Validation, all offline.** (a) Each lifted wall matched the Unicorn oracle on random inputs. (b) `K0`, `KA` and the post-mix matched the emulator's stored values. (c) Decrypting the binary's own embedded blobs with the static schedule gave three coherent plaintexts. A wrong schedule gives random bytes, so coherent decoy text is cryptographic evidence that schedule and cipher are right. (d) Forward-encrypting the recovered flag reproduced `E` byte-for-byte.

**Provenance, stated plainly.** The graded solve was a teammate's: they scored it at 2026-10-30 16:01 UTC, and our later submission (made to confirm, after the offline derivation) returned `already_solved`. For this ID `retry` returned 405 `operation_not_allowed` and `practice` returned 409 `retry_variants_unavailable`, with nothing consumed, so there was no way to obtain a fresh attempt. We therefore record this as a team solve, not ours. Our offline derivation is independent in the sense that it was rebuilt from our own binary and never used the teammate's flag as an input. It was not independent of their *notes*: their write-up of the MBA-collapse idea and the slot/record structure was the specification we ported from, and it is what unblocked us.

## Dead ends and gotchas

- **The "environment-keyed" theory (about 4 hours).** Our emulation-based solve built the cipher correctly and round-tripped its own test input, yet all 14 gate targets decoded to garbage. Changing emulated total RAM or computer/user name changed the keys, which looked like proof that the keys depended on the host, and we spent rounds trying to reproduce a plausible machine. That was a confound. The reads are anti-analysis theater folded through MBA chains whose clean-run value is a compile-time constant. The emulator computed a different `Q` while `K0`, `KA` and `mix45` matched. We never pinned exactly where the emulated `Q` diverged (candidates: the `rdtsc`/timing chain, or a VEH self-modifying wall during the integrated run). The static route sidesteps the question. Matching early stages does not vouch for later ones.
- **Breakpoints corrupt self-hashing code.** A debugger capture taken for us on an isolated machine used software breakpoints inside `.text`, which the program hashes over itself. Three leaked `int3` bytes at RVAs `0x3cbc9`, `0x3d0b3` and `0x3d1fe` changed the digest and produced a different parameter set and a different gate structure (block counts `{9,8,13,7,20,6,5,2}` instead of `{3,3,3,2,1,3,3}`). The capture misled us into thinking the emulator was wrong. Diffing the capture against the on-disk bytes showed exactly those three bytes. Use hardware breakpoints, or diff first.
- **"Bytecode VM" theory.** The gate loop does varint-decode instructions, so we concluded for a while that it was a bytecode VM whose accumulator could not be inverted by the plain cipher. Even on clean traces the emulated targets stayed unsatisfiable (we showed it down to single-block gates), which pointed at a wrong key scalar rather than the cipher. The static solve never needed the VM reading, and we did not determine whether it was ever correct.
- **Wrong subkey scope.** An earlier inversion used one key set's subkey for gates of the other set. It round-tripped our own buffer and still failed on the real targets.
- **Do not blame an API without checking the imports.** A theory that `CheckRemoteDebuggerPresent` returning 1 caused the divergence was wrong: that function is not imported, and the stubs already reported no debugger.
- **Decoys are accepted inputs too.** The checker passes `flag{nice_try}` and `your flag is in another castle, sorry!` as well as the real flag. Pick the one matching the stated flag format.

(Early automation rounds submitted an empty placeholder answer for this ID twice, since no candidate existed; see the Day 30 overview.)

## Takeaways

- **Do not fight anti-analysis in an emulator; compute the clean-run constants statically.** If a faithful emulator round-trips your own input but every expected target is garbage, some scalar in your emulated run differs from the clean-run constant. Search for the chain that folds environment reads into key material and evaluate it on a clean run (`x = (B - A - C) * M^-1 mod 2^64` where the chain is linear).
- **Wall isolation harness.** Treat a self-decrypting VEH region as a function: map the image into Unicorn, answer each `int3` the way the handler would, call it on random inputs, and grid-search SHA-1 deviations (taps, `Ch` swap, boundary round, rotate). If nothing fits, lift the executed instructions to Python and verify on many inputs.
- **Use decoys as a check.** When a checker has joke accepted inputs, coherent decoy plaintexts out of the embedded blobs prove your schedule right before you ever look at the real flag.
- **Check team status before a validation-only re-derivation.** A Single-Solve-only challenge that a teammate has already solved cannot be re-graded, so a long offline re-derivation can show your workflow works but cannot score for you.
- Technique names to search: self-modifying SHA-1 variant, VEH int3 harness, mixed boolean arithmetic collapse, flag checker with decoy inputs.

## Evidence

Leaf base: `challenges/day30/binary-reverse-engineering/binary-reverse-engineering-day-30-sha1-hulud/single-solve/`

- `journal.md` (full history: stalled emulation rounds, breakpoint-contamination finding, the teammate-shared notes, our independent re-derivation, the `already_solved` outcome).
- `start_response.json` (prompt, one `flag` answer id, artifact hash).
- `artifacts/static-solver/compress.py` (the four lifted walls, checked against the oracle), `mix45.py`, `sol.py` (key schedule and MBA chains), `recover.py` (readers, cipher, enumeration), `RESULT.json` (recovered slot pair, record, and the forward-encrypt check).
- `artifacts/r2/` (the Unicorn oracle and first-round cipher), `artifacts/escalation-brief.md`.
- No `result.json`, `accepted_plan.json` or `manual_solve.json`: nothing was graded for us. `progress.md` records the row as Team solve (not ours).
- Related: `challenges/day30/_run/lane-binary-reverse-engineering-day-30-sha1-hulud.log`, `challenges/day30/_run/supervisor-report.md`, library entry BRE-MBA-FLAGCHECK-001 and `library/binary-reverse-engineering/wall_isolate.py`.
