---
name: nanopore-read-mapping-decision-scoring
description: Use when you have a per-read feature table (read_id, mapping-quality
  ratio, chain-score ratio, mean chain-score ratio) from RawHash2 output and need
  to reproduce or audit the weighted-sum mapped/unmapped decision.
when_to_use_negative:
- Input is raw FAST5/POD5/SLOW5/BLOW5 signal files rather than an already-extracted
  per-read feature table — indexing and mapping/chaining must run first.
- The goal is reference-free overlap discovery (all-vs-all), which is Rawsamble's
  task, not the RawHash2 mapping decision.
- Features or weights are unavailable and cannot be supplied — the threshold mechanism
  cannot be reproduced without explicit config values.
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_3168
tools:
- name: RawHash2
  role: Source of the weighted mapping decision mechanism being reconstructed; produces
    the chain scores and mapping quality that feed the weighted sum.
  repo: https://github.com/CMU-SAFARI/RawHash
- name: RawHash
  role: Baseline mechanism whose fixed conditional mapping checks the weighted-sum
    threshold decision replaces.
  repo: https://github.com/CMU-SAFARI/RawHash
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/bioinformatics/btae478
    title: 'RawHash2: mapping raw nanopore signals using hash-based seeding and adaptive
      quantization'
  source_content_hash: sha256:25daf3b0e63fe2a4d3e58ef7600496a9fb502e0310f0fd51e6c41694ca870f1c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/CMU-SAFARI/RawHash
  tools:
  - RawHash
  - RawHash2
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# nanopore-read-mapping-decision-scoring

## Summary

Reconstructs RawHash2's weighted mapping decision mechanism (Section 2.4): a per-read weighted sum over mapping-quality and chain-score ratios is compared against a configurable threshold to emit a mapped/unmapped call, replacing RawHash's fixed conditional checks.

## When to use

Use when you have a per-read feature table (read_id, mapping-quality ratio, best chain-score ratio, mean chain-score ratio) from RawHash2 output and need to reproduce or audit the weighted-sum mapped/unmapped decision.

## When NOT to use

- Input is raw FAST5/POD5/SLOW5/BLOW5 signal files rather than an already-extracted per-read feature table — indexing and mapping/chaining must run first.
- The goal is reference-free overlap discovery (all-vs-all), which is Rawsamble's task, not the RawHash2 mapping decision.
- Features or weights are unavailable and cannot be supplied — the threshold mechanism cannot be reproduced without explicit config values.

## Inputs

- Per-read feature table (TSV/CSV) with columns: read_id, mapping-quality ratio, best chain-score ratio, mean chain-score ratio
- Module configuration specifying the per-feature weights and the mapped/unmapped decision threshold (no defaults prescribed)

## Outputs

- mapping_decisions.tsv: per-read table with columns read_id, wsum, decision ('mapped'/'unmapped')

## How to apply

Load the per-read feature table into pandas, then read the feature weights and the mapped/unmapped decision threshold from the module configuration — the paper prescribes no default values, so both must be supplied explicitly as config inputs. For each read compute the weighted sum wsum = Σ(r_i · w_i) over the mapping-quality ratio and the best/mean chain-score ratios using vectorized numpy arithmetic. Apply the decision rule per read: emit 'mapped' if wsum >= threshold, otherwise 'unmapped'. This threshold comparison is the RawHash2 mechanism that replaces RawHash's fixed conditional checks for deciding whether a raw signal maps to the reference genome during real-time sequencing. Write the per-read results (read_id, wsum, decision) to mapping_decisions.tsv with pandas.

## Related tools

- **RawHash2** (Source of the weighted mapping decision mechanism being reconstructed; produces the chain scores and mapping quality that feed the weighted sum.) — https://github.com/CMU-SAFARI/RawHash
- **RawHash** (Baseline mechanism whose fixed conditional mapping checks the weighted-sum threshold decision replaces.) — https://github.com/CMU-SAFARI/RawHash

## Examples

```
import pandas as pd, numpy as np; f = pd.read_csv('per_read_features.tsv', sep='\t'); f['wsum'] = f['mq_ratio']*w_mq + f['best_chain_ratio']*w_best + f['mean_chain_ratio']*w_mean; f['decision'] = np.where(f['wsum'] >= threshold, 'mapped', 'unmapped'); f[['read_id','wsum','decision']].to_csv('mapping_decisions.tsv', sep='\t', index=False)
```

## Evaluation signals

- mapping_decisions.tsv exists and parses as TSV with a decision column containing only 'mapped' or 'unmapped' values.
- Every read_id present in the input feature table receives exactly one decision — no missing or duplicated rows.
- Each emitted wsum equals the recomputed Σ(r_i · w_i) over the mapping-quality ratio and best/mean chain-score ratios for that read.
- Decision agrees with the rule everywhere: 'mapped' exactly when wsum >= threshold, 'unmapped' otherwise.
- Rows where wsum lies just above vs. just below the threshold flip decisions at the boundary as expected.

## Limitations

- The paper prescribes no default feature weights or threshold; both must be supplied explicitly, and results depend entirely on the chosen configuration.
- The decision quality is bounded by upstream RawHash2 chaining and seed-matching quality; raw signals in FAST5 must also be VBZ-compressed readable inputs for the upstream mapping.
- RawHash2's accuracy gains (F1 on average +10.57%, up to +20.25% over RawHash) are claimed at the mechanism level; the threshold decision itself is a simplified reconstruction, not a validated classifier.

## Evidence

- [intro] RawHash (and RawHash2) is a hash-based mechanism to map raw nanopore signals to a reference genome in real-time.: "RawHash (and RawHash2) is a hash-based mechanism to map raw nanopore signals to a reference genome in real-time."
- [methods] Chaining over seed matches: "To map the raw signal to a reference genome, RawHash performs chaining over the seed matches."
- [abstract] F1 accuracy improvement over RawHash: "RawHash2 provides better F1 accuracy (on average by 10.57% and up to 20.25%)"
- [methods] FAST5/POD5/SLOW5/BLOW5 input support: "RawHash can be used to map reads from **FAST5, POD5, SLOW5, or BLOW5** files to a reference genome in sequence format."
