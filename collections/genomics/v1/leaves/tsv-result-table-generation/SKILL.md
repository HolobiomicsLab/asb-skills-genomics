---
name: tsv-result-table-generation
description: Use when RawHash2 per-read outputs (mapping-quality ratio, chain-score
  ratio, mean chain-score ratio) must be converted into explicit mapped/unmapped calls
  via a weighted-sum threshold, replacing RawHash's fixed conditional checks.
when_to_use_negative:
- The input already contains mapped/unmapped labels or a final decision column — re-deriving
  wsum would duplicate an existing result
- The task is to run the RawHash2 mapping itself (indexing the reference, chaining
  over seed matches, or DTW signal alignment) rather than post-hoc tabulation of its
  per-read decision features
- Base-level sequence alignment quality (CIGAR/SAM) is required — the weighted-sum
  decision is a coarse mapping call, not a base-level alignment
tools:
- name: RawHash2
  role: Source mechanism whose mapping-quality ratio and best/mean chain-score ratios
    are combined into the weighted sum compared against the threshold
  repo: https://github.com/CMU-SAFARI/RawHash
- name: RawHash
  role: Predecessor tool whose fixed conditional mapping checks this weighted-sum
    decision mechanism replaces; used to produce comparable per-read mapping features
  repo: https://github.com/CMU-SAFARI/RawHash
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/bioinformatics/btae478
    title: 'RawHash2: mapping raw nanopore signals using hash-based seeding and adaptive
      quantization'
  source_content_hash: sha256:fddde2a4d034b6ae48ba28dce66f1f9d5908a6b68aa9e1c854a98793153b7d41
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/CMU-SAFARI/RawHash
  tools:
  - RawHash
  - RawHash2
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# tsv-result-table-generation

## Summary

Reconstructs RawHash2's weighted mapping-decision rule: for each raw nanopore read, compute a weighted sum over the mapping-quality ratio and the best/mean chain-score ratios, compare it against a predefined threshold, and emit a per-read 'mapped'/'unmapped' decision table (mapping_decisions.tsv).

## When to use

Use when RawHash2 per-read outputs (mapping-quality ratio, best chain-score ratio, mean chain-score ratio) must be converted into explicit mapped/unmapped calls via a weighted-sum threshold, replacing RawHash's fixed conditional checks.

## When NOT to use

- The input already contains mapped/unmapped labels or a final decision column — re-deriving wsum would duplicate an existing result
- The task is to run the RawHash2 mapping itself (indexing the reference, chaining over seed matches, or DTW signal alignment) rather than post-hoc tabulation of its per-read decision features
- Base-level sequence alignment quality (CIGAR/SAM) is required — the weighted-sum decision is a coarse mapping call, not a base-level alignment

## Inputs

- Per-read feature table (TSV/CSV) with read_id, mapping-quality ratio, best chain-score ratio, and mean chain-score ratio from RawHash2 mapping output
- Module configuration specifying feature weights and the mapped/unmapped decision threshold (explicit inputs, no defaults)
- Optionally RawHash2 mapping output from FAST5, POD5, SLOW5, or BLOW5 raw signal files against a reference genome index

## Outputs

- mapping_decisions.tsv: a per-read TSV with columns read_id, wsum, and decision ('mapped' or 'unmapped')

## How to apply

Load the per-read feature table (read_id, mapping-quality ratio, best chain-score ratio, mean chain-score ratio) into a pandas DataFrame. Read the feature weights and the decision threshold from module configuration — no default values are prescribed, so both must be supplied as explicit config inputs. For each read, compute the weighted sum wsum = Σ(r_i · w_i) over the three ratios with vectorized numpy arithmetic, then apply the rule: emit 'mapped' if wsum >= threshold, otherwise 'unmapped'. Write the per-read results table (read_id, wsum, decision) to mapping_decisions.tsv. This mirrors RawHash2's replacement of RawHash's fixed conditional checks with a weighted-sum comparison against a threshold during real-time mapping of raw signals.

## Related tools

- **RawHash2** (Source mechanism whose mapping-quality ratio and best/mean chain-score ratios are combined into the weighted sum compared against the threshold) — https://github.com/CMU-SAFARI/RawHash
- **RawHash** (Predecessor tool whose fixed conditional mapping checks this weighted-sum decision mechanism replaces; used to produce comparable per-read mapping features) — https://github.com/CMU-SAFARI/RawHash

## Examples

```
import pandas as pd, numpy as np; df = pd.read_csv('per_read_features.tsv', sep='\t'); df['wsum'] = w[0]*df['mapq_ratio'] + w[1]*df['best_chain_ratio'] + w[2]*df['mean_chain_ratio']; df['decision'] = np.where(df['wsum'] >= threshold, 'mapped', 'unmapped'); df[['read_id','wsum','decision']].to_csv('mapping_decisions.tsv', sep='\t', index=False)
```

## Evaluation signals

- mapping_decisions.tsv exists and parses as a valid TSV
- The decision column contains only the values 'mapped' and 'unmapped'
- Every read_id in the input feature table is assigned exactly one decision (no missing or duplicated reads)
- Each row's wsum equals Σ(r_i · w_i) over the mapping-quality ratio and the best/mean chain-score ratios when recomputed with the configured weights
- Decision flips to 'unmapped' for any read whose wsum falls just below the configured threshold (boundary check on the >= rule)

## Limitations

- No default feature weights or threshold values are prescribed by RawHash2 — they must be supplied as explicit configuration inputs, and the decision outcome is entirely sensitive to the chosen values
- The weighted sum is a heuristic composite of mapping-quality and chain-score ratios, so a 'mapped' call does not certify base-level alignment accuracy
- RawHash2 only supported legacy flows in its RawHash incarnation; decision features assume the newer RawHash2 pipeline (R10.4 support, POD5/SLOW5 inputs) rather than older fixed-check outputs

## Evidence

- [other] weighted-sum threshold decision: [evidence span withheld — anchor status: not_found]
- [readme] real-time raw signal mapping: "RawHash (and RawHash2) is a hash-based mechanism to map raw nanopore signals to a reference genome in real-time."
- [methods] supported input formats: "RawHash can be used to map reads from **FAST5, POD5, SLOW5, or BLOW5** files to a reference genome in sequence format."
- [methods] chaining step producing mapping features: "To map the raw signal to a reference genome, RawHash performs chaining over the seed matches."
- [intro] hash-based query mechanism: "In the mapping step, RawHash generates the hash values from raw signals and queries the hash table"
