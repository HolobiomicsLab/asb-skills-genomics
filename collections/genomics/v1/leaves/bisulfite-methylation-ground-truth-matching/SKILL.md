---
name: bisulfite-methylation-ground-truth-matching
description: Use when validating Nanopore per-site CpG methylation frequencies (e.g.
  DeepMod2 output.per_site.aggregated) against short-read bisulfite sequencing or
  methylation microarray ground truth via Pearson correlation on shared CpG sites.
when_to_use_negative:
- Evaluating per-read methylation accuracy instead of per-site methylation frequency
  — the paper uses F1-score against binary per-read ground truth (sites ≥90% or <10%
  methylated) for that, not correlation
- Detection or quantification of modifications other than 5mCpG — DeepMod2 currently
  only provides 5mCpG models and there is a lack of available training datasets for
  other modification types
- Sites with intermediate 20–80% methylation — EpiQC reported poor concordance at
  these sites both between ONT and short-read methods and among short-read methods,
  so they should be excluded from the benchmark
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_3168
tools:
- name: DeepMod2
  role: Produces per-site CpG methylation frequencies (output.per_site, output.per_site.aggregated)
    from Nanopore POD5/FAST5 signal plus aligned BAM that are correlated against ground
    truth
  repo: https://github.com/WGLab/DeepMod2
- name: modkit
  role: Converts DeepMod2 modBAM output to bedMethyl tables using ONT best practices
    for downstream per-site comparisons
  repo: https://github.com/nanoporetech/modkit
- name: Dorado
  role: Basecalls FAST5/POD5 files with --emit-moves to produce the move-table BAM
    required upstream of DeepMod2 detect
  repo: https://github.com/nanoporetech/dorado
- name: bedtools
  role: Intersects DeepMod2 per-site CpG methylation frequencies with short-read ground-truth
    methylation at shared CpG positions
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-024-45778-y
    title: A signal processing and deep learning framework for methylation detection
      using Oxford Nanopore sequencing
  source_content_hash: sha256:5ae8141fa94a7742582798442e7248e313ddc91b60e827d0b958429d3155bc7b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/WGLab/DeepMod2
  tools:
  - DeepMod2
  - Dorado
  - bedtools
  - modkit
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# bisulfite-methylation-ground-truth-matching

## Summary

Benchmark Nanopore per-site CpG methylation calls (e.g. DeepMod2 output.per_site.aggregated) against short-read bisulfite sequencing or methylation microarray ground truth by intersecting shared CpG positions and computing Pearson correlation. This is the correlation-based validation strategy the DeepMod2 paper uses to demonstrate r > 0.95 agreement with short-read sequencing.

## When to use

Use when validating Nanopore per-site CpG methylation frequencies (e.g. DeepMod2 output.per_site.aggregated) against short-read bisulfite sequencing or methylation microarray ground truth via Pearson correlation on shared CpG sites.

## When NOT to use

- Evaluating per-read methylation accuracy instead of per-site methylation frequency — the paper uses F1-score against binary per-read ground truth (sites ≥90% or <10% methylated) for that, not correlation
- Detection or quantification of modifications other than 5mCpG — DeepMod2 currently only provides 5mCpG models and there is a lack of available training datasets for other modification types
- Sites with intermediate 20–80% methylation — EpiQC reported poor concordance at these sites both between ONT and short-read methods and among short-read methods, so they should be excluded from the benchmark

## Inputs

- DeepMod2 per-site aggregated methylation output (MOD_CALLS/output.per_site.aggregated) with mod_percentage per CpG
- Short-read bisulfite sequencing or methylation microarray ground-truth methylation levels (e.g. GSE186383 EpiQC study) for the Ashkenazim trio (HG002, HG003, HG004)
- Aligned Nanopore BAM file with move tables and reference genome FASTA

## Outputs

- pearson_correlation_table.csv with columns sample, flowcell, pearson_r, n_cpg_sites
- Set of shared CpG positions with paired DeepMod2 and ground-truth methylation frequencies after coverage filtering

## How to apply

First produce methylation-tagged BAMs and per-site outputs by running DeepMod2 detect on the Nanopore signal files (POD5/FAST5 from R9.4.1 or R10.4.1 flowcells) against the aligned BAM and reference genome. Extract per-site CpG methylation frequencies (mod_percentage) from output.per_site.aggregated, removing low-coverage sites (<10X for the Nanopore caller and ≥10X for the Ashkenazim trio short-read/microarray ground truth; ≥5X for NIH3T3 ground truth). Intersect the filtered DeepMod2 CpG frequencies with the ground-truth methylation levels at shared CpG positions using bedtools intersect, excluding intermediate 20–80% methylation sites where concordance among platforms is known to be poor. Compute the Pearson correlation between DeepMod2 mod_percentage and ground-truth methylation frequency per sample per flowcell, writing coefficients to pearson_correlation_table.csv with columns sample, flowcell, pearson_r, n_cpg_sites. Validation: each Pearson r must exceed 0.95, matching the reported benchmark that DeepMod2 achieves r = 95–97.35% correlation against short-read ground truth on the Ashkenazim trio (HG002, HG003, HG004).

## Related tools

- **DeepMod2** (Produces per-site CpG methylation frequencies (output.per_site, output.per_site.aggregated) from Nanopore POD5/FAST5 signal plus aligned BAM that are correlated against ground truth) — https://github.com/WGLab/DeepMod2
- **modkit** (Converts DeepMod2 modBAM output to bedMethyl tables using ONT best practices for downstream per-site comparisons) — https://github.com/nanoporetech/modkit
- **Dorado** (Basecalls FAST5/POD5 files with --emit-moves to produce the move-table BAM required upstream of DeepMod2 detect) — https://github.com/nanoporetech/dorado
- **bedtools** (Intersects DeepMod2 per-site CpG methylation frequencies with short-read ground-truth methylation at shared CpG positions)

## Examples

```
python deepmod2 detect --bam reads.bam --input INPUT_DIR --model bilstm_r9.4.1 --file_type POD5 --threads 16 --ref ref.fa --output MOD_CALLS
```

## Evaluation signals

- Every Pearson r in pearson_correlation_table.csv exceeds 0.95, matching the published benchmark (DeepMod2 BiLSTM reaches 95–97.35% genome-wide CpG methylation-frequency correlation against short-read ground truth)
- Coverage filters are demonstrably applied: Nanopore CpG sites with <10X removed and ground-truth sites below 10X (Ashkenazim trio) or 5X (NIH3T3) removed before intersecting
- Output CSV conforms to the schema sample, flowcell, pearson_r, n_cpg_sites with one row per sample per flowcell
- Only CpG sites shared between DeepMod2 calls and short-read/microarray ground truth (bedtools intersect output) contribute to the correlation
- Sites with 20–80% methylation are absent from the evaluation set, consistent with the EpiQC concordance caveat

## Limitations

- Sites with 20–80% methylation had poor concordance not only between ONT and short-read methods but also among the different short-read methods, so intermediate methylation levels cannot be reliably benchmarked
- DeepMod2 models show a slight bias to overestimate methylation at some CpG sites, possibly from residual heterogeneity in training labels
- DeepMod2 only provides models for 5mCpG methylation because of a lack of available datasets for other types of methylations, so this matching workflow is restricted to 5mCpG
- Ground-truth availability is limited: the correlation benchmark relies on Ashkenazim trio (HG002, HG003, HG004) R9.4.1 and R10.4.1 datasets and NIH3T3 with matching short-read/microarray data (GSE186383)

## Evidence

- [intro] Ground truth sources: "we evaluate the performance of DeepMod2 with short-read sequencing and methylation microarrays as ground truth"
- [intro] Benchmark threshold: "DeepMod2 achieves ~95% and ~99% F1-score, respectively, with a correlation of r > 0.95 with short-read sequencing"
- [results] Coverage filtering: "after removing low coverage sites (<10X for Nanopore methylation callers and 10X/5X for Ashkenazim trio/NIH3T3 ground truth, respectively)"
- [results] Ground-truth site criteria: "the ground truth only included sites that were mostly methylated (≥80% methylation) and mostly unmethylated (<20% methylation) with ample coverage"
- [readme] Per-site output files: [evidence span withheld — anchor status: external_doc]
- [results] Intermediate-methylation exclusion rationale: "sites with 20–80% methylation had poor concordance not only between ONT and short-reads method, but also among the different short-read methods"
