---
name: per-read-mapping-quality-analysis
description: Use when you have a per-read feature table from raw nanopore signal mapping
  (read_id, mapping-quality ratio, chain-score ratio, mean chain-score ratio) and
  need to classify reads as mapped or unmapped with a weighted threshold rule.
when_to_use_negative:
- Input is already a mapping decision file or PAF alignment output rather than a per-read
  feature ratio table
- You need RawHash's legacy fixed conditional checks rather than RawHash2's weighted-sum
  threshold mechanism
- Feature weights or the decision threshold cannot be supplied explicitly, since no
  default values are prescribed
edam_operation: http://edamontology.org/operation_3198
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3168
tools:
- name: RawHash2
  role: Source of the weighted mapping decision mechanism being reconstructed; maps
    raw nanopore signals to a reference genome in real time
  repo: https://github.com/CMU-SAFARI/RawHash
- name: RawHash
  role: Predecessor whose fixed conditional checks the weighted-sum decision rule
    replaces; produces the raw signal mapping context
  repo: https://github.com/CMU-SAFARI/RawHash
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/bioinformatics/btae478
    title: 'RawHash2: mapping raw nanopore signals using hash-based seeding and adaptive
      quantization'
  source_content_hash: sha256:d14221a5231fabb43d1b02952fb8b54fb5b67cc184f1cebc8a1019bd0a2640f9
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/CMU-SAFARI/RawHash
  tools:
  - RawHash
  - RawHash2
  edam_operation: http://edamontology.org/operation_3198
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# per-read-mapping-quality-analysis

## Summary

Reconstructs RawHash2's weighted mapping decision mechanism: each read's mapping-quality ratio and best/mean chain-score ratios are combined into a weighted sum that is compared against a predefined threshold to emit a mapped/unmapped decision, replacing RawHash's fixed conditional checks.

## When to use

Use when you have a per-read feature table from raw nanopore signal mapping (read_id, mapping-quality ratio, best chain-score ratio, mean chain-score ratio) and need to classify reads as mapped or unmapped with a weighted threshold rule.

## When NOT to use

- Input is already a mapping decision file or PAF alignment output rather than a per-read feature ratio table
- You need RawHash's legacy fixed conditional checks rather than RawHash2's weighted-sum threshold mechanism
- Feature weights or the decision threshold cannot be supplied explicitly, since no default values are prescribed

## Inputs

- Per-read feature table (TSV/CSV) with read_id, mapping-quality ratio, best chain-score ratio, mean chain-score ratio
- Configuration specifying per-feature weights and the mapped/unmapped decision threshold (no defaults exist)

## Outputs

- mapping_decisions.tsv with columns read_id, wsum, decision where decision is 'mapped' or 'unmapped' for every input read

## How to apply

Load the per-read feature table (read_id plus mapping-quality ratio, best chain-score ratio, mean chain-score ratio) into a pandas DataFrame, and read the feature weights and decision threshold from explicit module configuration since RawHash2 prescribes no defaults. For each read, compute the weighted sum wsum = Σ(r_i · w_i) over the three ratios using vectorized numpy arithmetic. Apply the decision rule per read: a read is considered mapped if the weighted sum exceeds the threshold value, otherwise unmapped. Write the per-read results table (read_id, wsum, decision) to mapping_decisions.tsv. This reconstructs Section 2.4 of the RawHash2 paper, where the weighted-sum mechanism replaced RawHash's fixed conditional checks for real-time mapping decisions.

## Related tools

- **RawHash2** (Source of the weighted mapping decision mechanism being reconstructed; maps raw nanopore signals to a reference genome in real time) — https://github.com/CMU-SAFARI/RawHash
- **RawHash** (Predecessor whose fixed conditional checks the weighted-sum decision rule replaces; produces the raw signal mapping context) — https://github.com/CMU-SAFARI/RawHash

## Examples

```
python map_decision.py --features features.tsv --weights 0.4,0.3,0.3 --threshold 0.6 --output mapping_decisions.tsv
```

## Evaluation signals

- mapping_decisions.tsv exists and parses as a valid TSV
- The decision column contains only 'mapped' or 'unmapped' values
- Exactly one decision is assigned to every read_id present in the input feature table
- wsum values equal the dot product of the feature ratios and configured weights (verify on a sample of reads)
- Every read with wsum >= threshold is labeled 'mapped' and every read below is 'unmapped'

## Limitations

- RawHash2 prescribes no default feature weights or threshold; they must be supplied as explicit configuration inputs
- The decision quality depends on the upstream feature ratios (mapping-quality ratio, best and mean chain-score ratios) produced during chaining over seed matches
- Only valid for raw nanopore signal mappings generated by hash-based mechanisms like RawHash/RawHash2, not basecalled-read alignments

## Evidence

- [methods] RawHash2 compares the weighted sum wsum against a predefined threshold to decide mapping: [evidence span withheld — anchor status: not_found]
- [intro] RawHash (and RawHash2) is a hash-based mechanism: "RawHash (and RawHash2) is a hash-based mechanism to map raw nanopore signals to a reference genome in real-time."
- [methods] To map the raw signal to a reference genome, RawHash performs chaining: "To map the raw signal to a reference genome, RawHash performs chaining over the seed matches."
