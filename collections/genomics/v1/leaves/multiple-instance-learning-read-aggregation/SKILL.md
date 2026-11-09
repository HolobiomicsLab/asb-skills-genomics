---
name: multiple-instance-learning-read-aggregation
description: Use when read-level feature rows or read-level modification probabilities
  must be collapsed into per-site (transcript, pos, strand) methylation probabilities
  from Nanopore DRS data, provided site coverage is sufficient.
when_to_use_negative:
- When only single-read-level evaluation is desired (e.g. read-level ACC/SEN/SPE/AUC
  benchmarking against nanom6A) — no site aggregation is needed
- When a candidate site has fewer than 20 supporting reads, as the coverage filter
  ($4>=20) would exclude it and site-level probabilities would be unreliable
edam_operation: http://edamontology.org/operation_3802
edam_topics:
- http://edamontology.org/topic_3524
- http://edamontology.org/topic_3168
tools:
- name: RedNano
  role: Provides scripts/aggre_features_to_site_level.py and the MIL-based comb_basecall_raw
    model that combines read-level predictions into a site-level probability Pt
  repo: https://github.com/Derryxu/RedNano
- name: PyTorch
  role: Framework in which the dual-ResNet read-level encoder and MIL aggregation
    heads are implemented and trained
- name: Python
  role: Runtime for the aggregation and prediction scripts
provenance:
  source_task_ids:
  - task_004
  - task_003
  source_papers:
  - doi: 10.1093/bioinformatics/btae375
    title: RNA m6A detection using raw current signals and basecalling errors from
      Nanopore direct RNA sequencing reads
  source_content_hash: sha256:20f4dbf80fbb669665f63a025f37b679b640b78e39bc9bb36a328eecb95ac505
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Derryxu/RedNano
  tools:
  - CUDA
  - PyTorch
  - Python
  - RedNano
  - nanom6A
  edam_operation: http://edamontology.org/operation_3802
  edam_topics:
  - http://edamontology.org/topic_3524
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 10
---

# Multiple-instance-learning read aggregation

## Summary

Aggregate RedNano read-level m6A predictions into transcriptome-level site probabilities using a multiple-instance-learning approach, grouping read-level features by transcript position and retaining only sites with >= 20 reads of coverage.

## When to use

Use when read-level feature rows or read-level modification probabilities must be collapsed into per-site (transcript, pos, strand) methylation probabilities from Nanopore DRS data, provided site coverage is sufficient.

## When NOT to use

- When only single-read-level evaluation is desired (e.g. read-level ACC/SEN/SPE/AUC benchmarking against nanom6A) — no site aggregation is needed
- When a candidate site has fewer than 20 supporting reads, as the coverage filter ($4>=20) would exclude it and site-level probabilities would be unreliable

## Inputs

- Sorted read-level feature TSV (test.features.sort.tsv) with columns: chrom, pos, alignstrand, loc_in_ref, readname, strand, k_mer, k_signals_rect, qual, mis, ins, dele
- Aggregated site-level feature TSV (test.features.sort.aggre.tsv) before coverage filtering
- Trained RedNano MIL checkpoint (e.g. models/mil_allspecies_model_states.pt)
- Read counts per site used for the coverage>=20 filter

## Outputs

- Coverage-filtered site-level feature file (test.features.sort.aggre.c20.tsv)
- Per-site modification probability table with columns: transcript, pos, strand, prob, label (values in [0,1])

## How to apply

After RedNano feature extraction, sort the per-read feature rows by chrom, position, and readname (sort -k1,1 -k2,2n -k5,5) and run scripts/aggre_features_to_site_level.py to group read-level features by site. Apply a coverage filter of >= 20 reads per site (awk '$4>=20') so that site-level aggregation is statistically supported by multiple read instances. Train or predict with the MIL-aware model (model_type comb_basecall_raw, hidden_size 128, embedding_size 4, seq_lens 5, signal_lens 65), where read-level predictions from the dual-ResNet branches (raw signal + basecalling error) are combined into a site-level probability Pt per transcript position. The rationale is that a modified site yields a mixture of modified/unmodified reads (instances), and MIL aggregates these into a single site call rather than treating each read independently.

## Related tools

- **RedNano** (Provides scripts/aggre_features_to_site_level.py and the MIL-based comb_basecall_raw model that combines read-level predictions into a site-level probability Pt) — https://github.com/Derryxu/RedNano
- **PyTorch** (Framework in which the dual-ResNet read-level encoder and MIL aggregation heads are implemented and trained)
- **Python** (Runtime for the aggregation and prediction scripts)

## Examples

```
sort -k1,1 -k2,2n -k5,5 -T./ --parallel=30 -S30G test/features/* > test.features.sort.tsv && python /path/to/RedNano/scripts/aggre_features_to_site_level.py --input test.features.sort.tsv --output test.features.sort.aggre.tsv && awk 'BEGIN{ FS=OFS="\t" } $4>=20 {print $0}' test.features.sort.aggre.tsv > test.features.sort.aggre.c20.tsv
```

## Evaluation signals

- Coverage-filtered file contains only rows whose 4th column (read count) is >= 20
- predict.py output probabilities all fall in the range [0, 1], consistent with the sigmoid site-level output
- Aggregated output has one row per (transcript, pos, strand) triplet rather than one row per read
- Site-level probabilities reproduce expected m6A enrichment at RRACH/DRACH motifs (RedNano achieves >0.95 accuracy and >0.99 AUC at RRACH read level, >0.94/>0.98 at DRACH on synthesized data)

## Limitations

- Sites with coverage below 20 reads are discarded by the aggregation filter, so low-coverage positions receive no site-level call
- Aggregation requires prior read-level sorting (sort -k1,1 -k2,2n -k5,5); unsorted input will mis-group reads into sites

## Evidence

- [readme] To use the multiple instance learning approach, the read-level features should be aggregated as follows:: "To use the multiple instance learning approach, the read-level features should be aggregated as follows:"
- [methods] coverage filter evidence: "awk 'BEGIN{ FS=OFS="\t" } $4>=20 {print $0}' test.features.sort.aggre.tsv > test.features.sort.aggre.c20.tsv"
- [methods] aggregation step evidence: "python /path/to/RedNano/scripts/aggre_features_to_site_level.py --input test.features.sort.tsv --output test.features.sort.aggre.tsv"
- [methods] grouped by site coverage >= 20: [evidence span withheld — anchor status: not_found]
