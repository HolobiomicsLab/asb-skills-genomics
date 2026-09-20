---
name: signal-chunk-count-aggregation
description: Use when PAF mapping outputs (UNCALLED-style template) from RawHash and
  RawHash2 are available for FAST5 test datasets and the goal is to compute mean bases/chunks
  processed before the mapping decision, e.g. to reproduce Table 2.
when_to_use_negative:
- The mapper did not run in early-termination/real-time mode, so PAF records do not
  contain the mapping-decision prefix length or chunk count.
- Comparing throughput or F1 accuracy metrics instead of pre-decision sequencing length
  — those require different workflow steps (accuracy scoring, bases/second measurements).
edam_topics:
- http://edamontology.org/topic_0102
tools:
- name: RawHash
  role: Baseline mapper whose PAF outputs provide pre-decision bases/chunk counts
    for comparison
  repo: https://github.com/CMU-SAFARI/RawHash
- name: RawHash2
  role: Improved mapper whose PAF outputs are aggregated to quantify the sequencing
    length reduction
  repo: https://github.com/CMU-SAFARI/RawHash
- name: UNCALLED
  role: Defines the PAF output template that the mapping-decision records follow
  repo: https://github.com/skovaka/UNCALLED
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioinformatics/btae478
    title: 'RawHash2: mapping raw nanopore signals using hash-based seeding and adaptive
      quantization'
  source_content_hash: sha256:d906ca0bffd56570846d403a8b807c4f82af7f45ad9090207e5ce52a61e234fe
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/CMU-SAFARI/RawHash
  tools:
  - RawHash
  - RawHash2
  - UNCALLED
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# signal-chunk-count-aggregation

## Summary

Aggregates per-read bases and signal chunks processed before the mapping decision from RawHash/RawHash2 PAF outputs to quantify how much earlier RawHash2 terminates mapping (~1.9x shorter reads on average). This exploits the real-time early-termination behavior of hash-based raw nanopore signal mapping to measure read-until efficiency.

## When to use

Use when PAF mapping outputs (UNCALLED-style template) from RawHash and RawHash2 are available for FAST5 test datasets and the goal is to compute mean bases/chunks processed before the mapping decision, e.g. to reproduce Table 2.

## When NOT to use

- The mapper did not run in early-termination/real-time mode, so PAF records do not contain the mapping-decision prefix length or chunk count.
- Comparing throughput or F1 accuracy metrics instead of pre-decision sequencing length — those require different workflow steps (accuracy scoring, bases/second measurements).

## Inputs

- PAF mapping outputs (mapping.paf, modified UNCALLED-template format) from RawHash and RawHash2 runs
- Test FAST5 dataset labels (d1_sars-cov-2_r94, d3_yeast_r94, d4_green_algae_r94, d5_human_na12878_r94, d6_ecoli_r104)

## Outputs

- bases_chunks_summary.csv with per-tool/per-dataset mean bases processed, mean chunks processed, and reduction ratios
- Overall RawHash2-vs-RawHash sequencing length reduction ratio (expected ~1.9x)

## How to apply

Load the mapping.paf outputs (modified PAF format following the UNCALLED template) previously produced by RawHash and RawHash2 for each test FAST5 dataset. For every mapped read, parse the mapping-decision record to recover the read-prefix length in bases and the number of signal chunks processed before the mapper stopped, exploiting early termination (mapping stops once the read prefix can be mapped to the reference). Group per-read values by tool (RawHash vs RawHash2) and dataset (d1_sars-cov-2_r94, d3_yeast_r94, d4_green_algae_r94, d5_human_na12878_r94, d6_ecoli_r104) and compute mean bases and mean chunks with pandas. Compute the reduction ratio (RawHash mean bases / RawHash2 mean bases) per dataset and overall, then validate the overall ratio falls within a tolerance band around 1.9 before writing bases_chunks_summary.csv.

## Related tools

- **RawHash** (Baseline mapper whose PAF outputs provide pre-decision bases/chunk counts for comparison) — https://github.com/CMU-SAFARI/RawHash
- **RawHash2** (Improved mapper whose PAF outputs are aggregated to quantify the sequencing length reduction) — https://github.com/CMU-SAFARI/RawHash
- **UNCALLED** (Defines the PAF output template that the mapping-decision records follow) — https://github.com/skovaka/UNCALLED

## Examples

```
df.groupby(['tool','dataset'])[['bases_processed','chunks_processed']].mean().assign(reduction=lambda d: d.loc['RawHash','bases_processed']/d.loc['RawHash2','bases_processed'])
```

## Evaluation signals

- Overall RawHash2/RawHash average-bases ratio falls within a tolerance band around 1.9 (Section 3.4 value).
- Every per-dataset group has both a RawHash and RawHash2 mean, so no dataset is missing from bases_chunks_summary.csv.
- Per-read bases and chunks values are non-negative integers recovered from the mapping-decision records of mapped reads only.
- Chunks processed correlates monotonically with bases processed within each dataset (chunks are fixed-length signal units).

## Limitations

- Only mapped reads carry the mapping-decision record; unmapped reads that exhaust the signal provide no early-termination sample.
- The 1.9x figure is an average across evaluated datasets; per-dataset ratios can deviate from it.
- Chunk counts depend on the mapper's chunking configuration, so comparisons are only valid between matching RawHash/RawHash2 runs.

## Evidence

- [readme] RawHash (and RawHash2) is a hash-based mechanism to map raw nanopore signals to a reference genome in real-time.: "RawHash (and RawHash2) is a hash-based mechanism to map raw nanopore signals to a reference genome in real-time."
- [methods] We follow the similar PAF template used in [UNCALLED]: "We follow the similar PAF template used in [UNCALLED]"
- [results] RawHash2 reduces the average sequencing length before the mapping decision by 1.9× compared to RawHash, mainly due to improvements in mapping accuracy that enable quicker mapping decisions.: [evidence span withheld — anchor status: not_found]
