# Seeing Double

| | |
|---|---|
| Day / Lane | 30 / Metadata Forensics |
| Challenge ID | `metadata-forensics-day-30-seeing-double` |
| Technique | exact-identity grouping of a "cluster" ZIP (SHA-256 of real member bytes, not manifest claims) |
| Single Solve | Independent (graded CORRECT on the second submission call; a later re-check on Day 31 read the mode as already solved) |
| Time Trial | Independent (untimed regular attempt, plus a later ranked-style rerun, both CORRECT) |
| Mastery | Independent (untimed regular attempt, plus a later ranked-style rerun, both CORRECT) |

## Summary

Day 30's metadata challenge opened a "relationship map" block: given a cluster of files, find which
ones are byte-identical. The Single Solve asked only that one field. The answer is a JSON array of
groups, built from the SHA-256 of the real bytes of each member (the bundled manifest's hashes are
only claims), with singletons and near-duplicates left out. The hard part was not the hashing but the
unstated answer shape: member labels, member order and group order all had to match the validator.
All three modes went CORRECT.

## The challenge

Prompt: "The corpus is too large for manual browsing. Build a relationship map from exact identity,
semantic identity, containment, and claimed provenance, then expose the fabricated cluster."

- **Single Solve:** one question, `relation.identical_sha256_groups` for `cluster-001.zip`, answer
  format "compact valid JSON array with exact values and order; no Markdown or explanatory text",
  `case_insensitive` normalization. The download was an outer ZIP holding `cluster-001.zip`, which
  held a `delivery-manifest.json` plus 30 `artifacts/*` members: five families (alpha `.txt`,
  bravo `.ps1`, charlie `.json`, delta `.png`, echo `.zip`) of six members each.
- **Time Trial:** 35 fields, one of which was the same relation field. The other 34 were the earlier
  days' recurring metadata families (roster/evidence-note hashes, then one field per file format:
  JPEG, PNG, PDF, ZIP, PE, .NET, MSI, LNK, OOXML, email, audio/video, ELF, Mach-O, APK, JAR, SQLite,
  EVTX, registry, Prefetch, pcap, disk image, X.509) plus a synthesis ordering.
- **Mastery:** 14 sets of 35 fields each, 490 answers total, delivered as one ZIP of
  `evidence-NN.zip` bundles.

## Solution

1. **Open the cluster and look at its shape.** The Single Solve cluster had five families whose six
   members each shared identical bytes within the family and nothing across families. The manifest
   listed an `id`, `filename`, `path` and `sha256` for each member.
2. **Hash the bytes, ignore the manifest.** The prompt's "claimed provenance" wording points at the
   manifest, but the question asks for exact identity, so the solver SHA-256'd every `artifacts/*`
   leaf from its real bytes and grouped by digest. A group of one is not a duplicate, so singletons
   were dropped.
3. **Pick a shape.** Nothing in the prompt says what a group looks like. The working shape was an
   array of arrays of strings: members sorted within a group, groups sorted by their first member.

   ```python
   def groups_by(files, key):
       by = {}
       for f in files:
           by.setdefault(hashlib.sha256(f[3]).hexdigest(), []).append(f)
       gs = [sorted(g, key=lambda f: f[1]) for g in by.values() if len(g) > 1]
       gs.sort(key=lambda g: g[0][1])
       return [[key(f) for f in g] for g in gs], by
   ```

   `key` selects the label spelling, and that choice was the only real ambiguity.
4. **Label spelling.** The first submission, basenames with extensions (`alpha-01.txt`), was
   `incorrect`. The second, the manifest's `id` field, which is the filename without its extension
   (`alpha-01`), was CORRECT. The Single Solve therefore took two calls.

   Submitted answer shape (Single Solve): `[["alpha-01","alpha-02",...,"alpha-06"],["bravo-01",...],...]`
   with the five six-member groups in family order.

## Time Trial and Mastery

The cluster looked different outside the Single Solve:

- **Time Trial:** `cluster-001.zip` had no `delivery-manifest.json` and 31 extensionless members
  (`nectar-08`, `cobalt-60` and so on). It contained four duplicate groups (6, 4, 6 and 6 members) plus
  nine singletons. Several singletons were near-duplicates differing from a group member by one
  character, and a shared name prefix (`nectar-08/32/44/99`) spread across different groups, so any
  name-based or size-based heuristic would have merged things that SHA-256 keeps apart. With no
  manifest, the label is the archive basename (which equals the archive path minus `artifacts/`).
- **Mastery:** the same field in each of 14 sets, groups of 2 to 6 members, same basename labelling.
  One submission with all 490 fields was CORRECT.

The other 34 fields per set came from the existing metadata reader library; only the cluster field
needed new code. Each mode's solver output was submitted in one pass: Time Trial's other 34 fields
were accepted in a single call and the cluster field after that, Mastery went through in one call.

## Dead ends and gotchas

- **The labelling convention depends on whether a manifest is present.** With a manifest the
  validator wanted the manifest `id` (no extension); extension-bearing basenames were rejected. Without
  one it wanted the basename, and `artifacts/`-prefixed paths were rejected. Other rejected spellings,
  tried on the Time Trial: group order by SHA-256, archive (first-appearance) order, size-descending and
  count-descending group order, `{sha256, files}` objects, and a list that included singletons.
- **The regular Time Trial spent 14 calls before the answer was found.** The first pass reused the
  Single Solve winner as a variant *index* instead of as a spelling rule, and de-duplication of variants
  in the manifest-less shape shifted what that index meant, so the correct reading (variant 0) was
  never tried on its own until the fix round. Cost: wasted submissions only; see the day overview for
  the automation side of it.
- **Untested case.** A cluster with file extensions and no manifest was never seen, so whether the
  validator would want the stem or the basename there is unknown. The Time Trial and Mastery members
  were extensionless, so they cannot distinguish the two.
- The manifest hashes were a decoy for this field; the prompt's "fabricated cluster" language did not
  require anything beyond the byte-level grouping.

## Takeaways

Name the technique: **exact-identity grouping by content hash**, computed from the real bytes, with
singletons and near-duplicates excluded. When an answer shape is unstated, resolve each independent
choice (label, member order, group order) separately and carry the *rule* between modes, not a variant
index, because variant lists change length with the input shape. What we would do faster: on a
manifest-bearing cluster try the manifest `id` first and the bare basename second (the order they
were actually accepted in), and on a manifest-less cluster try the basename first.

## Evidence

Leaf base: `challenges/day30/metadata-forensics/metadata-forensics-day-30-seeing-double/`

- `single-solve/journal.md`, `single-solve/start_response.json`, `single-solve/accepted_plan.json`
  (it keeps the rejected extension-bearing variant first; the accepted one is variant 1, confirmed in
  the lane log), `single-solve/artifacts/cand-solo/solve.py`
- `single-solve/team_solved.json`: written by a Day 31 re-check; the Single Solve had already been graded
  CORRECT for us on Day 30 (lane log), so we treat the later marker as the re-check reading our own earlier solve
  as already correct, though the file itself cannot tell whose solve it was
- `time-trial/regular/result.json`, `time-trial/regular/accepted_plan.json`, `time-trial/ranked-1/result.json`
- `mastery/regular/result.json`, `mastery/regular/accepted_plan.json`, `mastery/ranked-1/result.json`
- `challenges/day30/_run/lane-metadata-forensics-day-30-seeing-double.log`
- `challenges/day30/_run/librarian/metadata-forensics-day-30-seeing-double.md`
- `library/metadata-forensics/cluster-identical-groups/README.md` (META-CLUSTER-IDENTICAL-001)
- `progress.md` (grep `^| 30 .*seeing-double`)
