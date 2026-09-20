---
name: chain-score-feature-aggregation
description: Use when you have a per-read feature table from RawHash2 signal mapping
  and need to decide mapped vs unmapped by thresholding a weighted combination of
  mapping-quality and chain-score ratios.
when_to_use_negative:
- You still need raw signal chaining or seed matching — produce the feature table
  with RawHash2 first; this skill only performs the decision step
- You need alignment-level scoring (e.g. DTW signal alignment via RawAlign) rather
  than a thresholded aggregate of mapping-quality and chain-score ratios
- The goal is reference-free all-vs-all overlapping (Rawsamble), which produces PAF
  overlaps rather than per-read mapped/unmapped decisions
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: RawHash2
  role: Generates the per-read mapping-quality ratio and best/mean chain-score ratios
    that feed the weighted-sum decision; its threshold rule replaces RawHash's fixed
    conditional checks
  repo: https://github.com/CMU-SAFARI/RawHash
- name: RawHash
  role: Baseline hash-based raw nanopore signal mapper whose fixed conditional mapping
    checks this weighted aggregation mechanism replaces
  repo: https://github.com/CMU-SAFARI/RawHash
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/bioinformatics/btae478
    title: 'RawHash2: mapping raw nanopore signals using hash-based seeding and adaptive
      quantization'
  source_content_hash: sha256:27b7710b7b62457d93a302e066fd3378916ad1c57f1e6c72e1705a8ed9ff18d4
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/CMU-SAFARI/RawHash
  tools:
  - RawHash
  - RawHash2
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# chain-score-feature-aggregation

## Summary

Reimplements RawHash2's weighted mapping-decision mechanism: per-read ratios (mapping-quality ratio, best and mean chain-score ratios) are aggregated into a weighted sum and compared against a configurable threshold, replacing RawHash's fixed conditional checks for real-time nanopore read mapping.

## When to use

Use when you have a per-read feature table from RawHash2 signal mapping and need to decide mapped vs unmapped by thresholding a weighted combination of mapping-quality and chain-score ratios.

## When NOT to use

- You still need raw signal chaining or seed matching — produce the feature table with RawHash2 first; this skill only performs the decision step
- You need alignment-level scoring (e.g. DTW signal alignment via RawAlign) rather than a thresholded aggregate of mapping-quality and chain-score ratios
- The goal is reference-free all-vs-all overlapping (Rawsamble), which produces PAF overlaps rather than per-read mapped/unmapped decisions

## Inputs

- Per-read feature table (TSV/CSV) with columns: read_id, mapping-quality ratio, best chain-score ratio, mean chain-score ratio
- Module configuration supplying per-feature weights and the mapped/unmapped decision threshold (explicit config inputs, no defaults)

## Outputs

- mapping_decisions.tsv with columns: read_id, wsum, decision (decision ∈ {'mapped','unmapped'})

## How to apply

Load the per-read feature table (read_id, mapping-quality ratio, best chain-score ratio, mean chain-score ratio) into a pandas DataFrame, then read the feature weights and the mapped/unmapped decision threshold explicitly from configuration, since no default values are prescribed by RawHash2. For each read, compute the weighted sum wsum = Σ(r_i · w_i) over the mapping-quality ratio and the best/mean chain-score ratios using vectorized numpy arithmetic. Apply RawHash2's decision rule: a read is considered mapped if the weighted sum exceeds (meets) the threshold, otherwise it is unmapped. This threshold test replaces RawHash's fixed conditional checks and underpins RawHash2's improved F1 accuracy (on average +10.57%, up to +20.25%) and throughput (on average 4.0×, up to 9.9× over RawHash) for real-time raw nanopore signal mapping.

## Related tools

- **RawHash2** (Generates the per-read mapping-quality ratio and best/mean chain-score ratios that feed the weighted-sum decision; its threshold rule replaces RawHash's fixed conditional checks) — https://github.com/CMU-SAFARI/RawHash
- **RawHash** (Baseline hash-based raw nanopore signal mapper whose fixed conditional mapping checks this weighted aggregation mechanism replaces) — https://github.com/CMU-SAFARI/RawHash

## Examples

```
python map_decide.py --features features.tsv --weights mq=0.5,best_chain=0.3,mean_chain=0.2 --threshold 0.6 --output mapping_decisions.tsv
```

## Evaluation signals

- mapping_decisions.tsv exists and parses as a valid TSV
- The decision column contains only the values 'mapped' and 'unmapped'
- Every read_id in the input feature table receives exactly one decision (no missing or duplicated read_ids)
- wsum values are reproducible from the input feature table via wsum = Σ(r_i · w_i) with the configured weights
- Decision boundary matches the rule: 'mapped' if wsum >= threshold, else 'unmapped'

## Limitations

- The feature weights and the decision threshold have no prescribed default values and must be supplied as explicit configuration inputs; poor choices directly skew mapped/unmapped calls
- The decision is only as good as the upstream RawHash2 chaining features (mapping-quality ratio, best/mean chain-score ratios); it cannot recover from bad seed matches
- RawHash2's accuracy/throughput gains (e.g. 4.0–9.9× throughput) are reported for its full pipeline, not for this decision step in isolation

## Evidence

- [intro] Hash-based real-time mapping context: "RawHash (and RawHash2) is a hash-based mechanism to map raw nanopore signals to a reference genome in real-time."
- [methods] Chaining produces the chain scores: "To map the raw signal to a reference genome, RawHash performs chaining over the seed matches."
- [abstract] Accuracy advantage of the RawHash2 pipeline: "RawHash2 provides better F1 accuracy (on average by 10.57% and up to 20.25%)"
- [abstract] Throughput advantage supporting real-time use: "better throughput (on average by 4.0× and up to 9.9×) than RawHash"
