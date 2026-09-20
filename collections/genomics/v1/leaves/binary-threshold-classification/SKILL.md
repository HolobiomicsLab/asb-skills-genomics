---
name: binary-threshold-classification
description: Use when you have a per-read feature table from raw nanopore signal mapping
  (read_id with mapping-quality and chain-score ratios) and must decide each read
  as 'mapped' or 'unmapped' via a weighted-sum threshold.
when_to_use_negative:
- The task is to actually run the RawHash2 mapper over FAST5/POD5/SLOW5/BLOW5 signals
  against a reference genome — use the RawHash2 indexing and mapping pipeline, not
  the post-hoc decision reconstruction.
- The features are not the weighted ratios described here (e.g., you have base-level
  alignments or minimap2 PAF scores); the weighted-sum threshold is specific to the
  mapping-quality and chain-score ratio features.
- Multi-class or probabilistic classification is needed; this mechanism produces exactly
  one binary decision per read.
edam_topics:
- http://edamontology.org/topic_3168
tools:
- name: RawHash2
  role: Source mechanism whose weighted-sum threshold rule replaces RawHash's fixed
    conditional checks for deciding whether a read maps to the reference genome during
    real-time sequencing
  repo: https://github.com/CMU-SAFARI/RawHash
- name: RawHash
  role: Prior hash-based mechanism for mapping raw nanopore signals to a reference
    genome in real-time, whose fixed conditional checks this classification scheme
    replaces
  repo: https://github.com/CMU-SAFARI/RawHash
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/bioinformatics/btae478
    title: 'RawHash2: mapping raw nanopore signals using hash-based seeding and adaptive
      quantization'
  source_content_hash: sha256:d43af180f2621dd8093f9e7b2b58584cfc0d79c5a513043a5e5df27e81cbae84
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/CMU-SAFARI/RawHash
  tools:
  - RawHash
  - RawHash2
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# binary-threshold-classification

## Summary

Reconstructs RawHash2's mapped/unmapped decision mechanism, in which a weighted sum of per-read score ratios (mapping-quality ratio, best and mean chain-score ratios) is compared against a predefined threshold instead of RawHash's fixed conditional checks. It provides a transparent, reproducible binary classifier over real-time nanopore signal mapping features.

## When to use

Use when you have a per-read feature table from raw nanopore signal mapping (read_id with mapping-quality and chain-score ratios) and must decide each read as 'mapped' or 'unmapped' via a weighted-sum threshold.

## When NOT to use

- The task is to actually run the RawHash2 mapper over FAST5/POD5/SLOW5/BLOW5 signals against a reference genome — use the RawHash2 indexing and mapping pipeline, not the post-hoc decision reconstruction.
- The features are not the weighted ratios described here (e.g., you have base-level alignments or minimap2 PAF scores); the weighted-sum threshold is specific to the mapping-quality and chain-score ratio features.
- Multi-class or probabilistic classification is needed; this mechanism produces exactly one binary decision per read.

## Inputs

- Per-read feature table (TSV/CSV) with read_id, mapping-quality ratio, best chain-score ratio, mean chain-score ratio
- Module configuration supplying feature weights (one per ratio feature) and the mapped/unmapped decision threshold (no defaults exist, so both must be given explicitly)

## Outputs

- mapping_decisions.tsv with columns read_id, wsum, decision, where decision is 'mapped' or 'unmapped'

## How to apply

Load the per-read feature table (read_id, mapping-quality ratio, best chain-score ratio, mean chain-score ratio) into a pandas DataFrame. Read the feature weights and the decision threshold from explicit module configuration, since RawHash2 prescribes no default values. Compute the weighted sum per read as wsum = Σ(r_i · w_i) over the mapping-quality ratio and the best/mean chain-score ratios using vectorized numpy arithmetic, then emit 'mapped' if wsum >= threshold and 'unmapped' otherwise. Write the per-read results (read_id, wsum, decision) to mapping_decisions.tsv. The rationale, from the RawHash2 paper, is to replace RawHash's fixed conditional checks with a single weighted threshold so that reads are classified during real-time sequencing.

## Related tools

- **RawHash2** (Source mechanism whose weighted-sum threshold rule replaces RawHash's fixed conditional checks for deciding whether a read maps to the reference genome during real-time sequencing) — https://github.com/CMU-SAFARI/RawHash
- **RawHash** (Prior hash-based mechanism for mapping raw nanopore signals to a reference genome in real-time, whose fixed conditional checks this classification scheme replaces) — https://github.com/CMU-SAFARI/RawHash

## Examples

```
wsum = (df[['mapq_ratio','best_chain_ratio','mean_chain_ratio']] * weights).sum(axis=1); df['decision'] = np.where(wsum >= threshold, 'mapped', 'unmapped')
```

## Evaluation signals

- mapping_decisions.tsv exists and parses as a valid TSV with a decision column
- decision column contains only 'mapped' or 'unmapped' values
- exactly one decision is assigned to every read_id present in the input feature table (no reads dropped or duplicated)
- wsum values equal Σ(r_i · w_i) over the supplied weights for each read (spot-checkable against a manual recomputation)
- reads with equal-or-higher weighted sums than the threshold are always labeled 'mapped' and those below 'unmapped'

## Limitations

- Neither the feature weights nor the mapped/unmapped threshold have prescribed defaults in the module configuration; both must be supplied explicitly, so results are only as valid as the user-provided configuration.
- The decision rule is a single threshold on a linear weighted sum, so it cannot capture interactions among the mapping-quality and chain-score ratio features.

## Evidence

- [methods] RawHash2 replaces RawHash's fixed conditional checks with a weighted-sum threshold decision: "To map the raw signal to a reference genome, RawHash performs chaining over the seed matches."
- [abstract] RawHash2's accuracy improvements motivate revisiting the mapping decision mechanism: "RawHash2 provides better F1 accuracy (on average by 10.57% and up to 20.25%)"
- [readme] The mapping operates in real-time on raw nanopore signals: "RawHash (and RawHash2) is a hash-based mechanism to map raw nanopore signals to a reference genome in real-time."
