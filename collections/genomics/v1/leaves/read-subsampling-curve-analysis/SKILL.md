---
name: read-subsampling-curve-analysis
description: Use when planning or validating nanoSHAPE experiments on ONT direct RNA
  data and you need to know the minimum read count at which per-read KS-derived reactivity
  correlations (current or dwell) plateau.
when_to_use_negative:
- Fewer than 100 reads are available in modified or control condition, below the --minimum-test-reads
  threshold tombo needs for in vitro sample comparison
- The goal is modification identification from trained kmer models rather than sample-comparison
  reactivity profiling against an unmodified control
- No SHAPE-MaP reactivity reference exists for the target, so the subsampled correlations
  cannot be anchored to an orthogonal profile
tools:
- name: tombo detect_modifications model_sample_compare
  role: Compares modified vs control fast5s and generates the per-read statistics
    file used to compute per-position KS statistics and downstream correlations
  repo: https://nanoporetech.github.io/tombo/
- name: extract_array.py
  role: Extracts per-read signal arrays (current/dwell/stdev) from single fast5s into
    a .npy array with reads sorted, feeding the KS reactivity computation
  repo: https://github.com/physnano/rRNA_nanoSHAPE
- name: python
  role: Runs extract_array.py and performs the subsampling, KS statistics, and Spearman
    correlation against the SHAPE-MaP profile
  repo: https://github.com/physnano/rRNA_nanoSHAPE
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1016/j.xgen.2022.100097
    title: Direct detection of RNA modifications and structure using single-molecule
      nanopore sequencing
  source_content_hash: sha256:67315a776f1ea41a2e4989d72c00529e91fef132a3ef1aec21d819cacc7e8f58
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/physnano/rRNA_nanoSHAPE
  tools:
  - extract_array.py
  - python
  - tombo
  - tombo detect_modifications model_sample_compare
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# read-subsampling-curve-analysis

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Assesses how many nanopore reads are needed for a stable nanoSHAPE reactivity profile by subsampling 1 to N reads and plotting Spearman correlation of the KS-derived per-read signal against the AcIm SHAPE-MaP reactivity profile. Establishes read-depth sufficiency (95% of maximum correlation at ~200 reads) for single-molecule structure probing.

## When to use

Use when planning or validating nanoSHAPE experiments on ONT direct RNA data and you need to know the minimum read count at which per-read KS-derived reactivity correlations (current or dwell) plateau.

## When NOT to use

- Fewer than 100 reads are available in modified or control condition, below the --minimum-test-reads threshold tombo needs for in vitro sample comparison
- The goal is modification identification from trained kmer models rather than sample-comparison reactivity profiling against an unmodified control
- No SHAPE-MaP reactivity reference exists for the target, so the subsampled correlations cannot be anchored to an orthogonal profile

## Inputs

- AcIm-modified single fast5 directory (direct RNA nanopore reads, e.g., pri-miR-17~92)
- Control/unmodified single fast5 directory for the same RNA
- FASTA reference file for the target RNA
- AcIm SHAPE-MaP reactivity profile for the same RNA (correlation reference)
- tombo per-read statistics file from model_sample_compare

## Outputs

- Per-read signal numpy array (.npy) for current, dwell, or stdev channels
- Per-position nanoSHAPE reactivity profile from KS statistics (per signal channel)
- Correlation-versus-read-count subsampling curve with read count at 95% of maximum correlation (~200 reads)

## How to apply

Run 'tombo detect_modifications model_sample_compare' on AcIm-modified versus control single fast5s with --minimum-test-reads 100 (typical for in vitro) and --per-read-statistics-basename to emit the per-read statistics file. Extract per-read signal arrays (current, dwell, or stdev) from the single fast5s with 'python extract_array.py' (--fa, --f5, --rna-name, --sort-reads, --current/--dwell/--stdev), producing a .npy array. Compute per-position Kolmogorov-Smirnov statistics between modified and control per-read signal distributions to obtain the nanoSHAPE reactivity profile per channel. Correlate this profile with the AcIm SHAPE-MaP reactivity profile (e.g., for pri-miR-17~92: current rho = 0.51, dwell rho = 0.28, maximized at 150 mM AcIm). Then subsample reads from 1 up to the full read count, recomputing the correlation at each depth, and plot correlation versus number of reads to find where the curve reaches 95% of its maximum.

## Related tools

- **tombo detect_modifications model_sample_compare** (Compares modified vs control fast5s and generates the per-read statistics file used to compute per-position KS statistics and downstream correlations) — https://nanoporetech.github.io/tombo/
- **extract_array.py** (Extracts per-read signal arrays (current/dwell/stdev) from single fast5s into a .npy array with reads sorted, feeding the KS reactivity computation) — https://github.com/physnano/rRNA_nanoSHAPE
- **python** (Runs extract_array.py and performs the subsampling, KS statistics, and Spearman correlation against the SHAPE-MaP profile) — https://github.com/physnano/rRNA_nanoSHAPE

## Examples

```
tombo detect_modifications model_sample_compare --fast5-basedirs <MODIFIED_single_fast5> --control-fast5-basedirs <CONTROL_single_fast5> --per-read-statistics-basename per_read_stats --minimum-test-reads 100 --multiprocess-region-size 250
```

## Evaluation signals

- At full read depth, the current-signal KS-derived nanoSHAPE profile reaches rho = 0.51 and dwell rho = 0.28 against the AcIm SHAPE-MaP profile at 150 mM AcIm
- The subsampling curve reaches 95% of its maximum correlation at approximately 200 reads
- Correlation versus read count increases monotonically (with noise) before plateauing; a non-plateauing curve indicates insufficient read depth
- model_sample_compare completed with --minimum-test-reads 100 and produced a valid per-read statistics file via --per-read-statistics-basename
- Per-channel comparison confirms current outperforms dwell time (rho 0.51 vs 0.28), matching the expected signal-channel ranking

## Limitations

- High rates of AcIm modification lower the number of full-length reads, overall yield, and alignment rates, distorting the subsampling curve at high modification levels
- nanoSHAPE has poor resolution of reactivity profiles at the 5' ends of longer RNA molecules, which can depress correlations regardless of read depth
- The subsampling estimate is anchored to an orthogonal SHAPE-MaP profile; without that reference, plateau position indicates internal consistency only

## Evidence
