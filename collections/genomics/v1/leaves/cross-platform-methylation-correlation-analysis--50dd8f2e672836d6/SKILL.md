---
name: cross-platform-methylation-correlation-analysis
description: Use when validating Nanopore methylation caller output (per-site CpG
  frequencies from DeepMod2 detect on POD5/FAST5 data) against short-read bisulfite
  or methylation microarray ground truth via Pearson correlation.
when_to_use_negative:
- You lack matched short-read or microarray ground truth for the same sample — correlation
  cannot be computed without shared CpG positions across platforms
- Input is raw POD5/FAST5 signal that has not been basecalled with move tables — DeepMod2
  requires move-table-aligned basecalls before per-site frequencies exist
- Per-read (not per-site) caller accuracy is the goal — the article separately benchmarks
  per-read F1 (~95%), and correlation analysis applies only to aggregated per-site
  methylation frequencies
edam_operation: http://edamontology.org/operation_3435
edam_topics:
- http://edamontology.org/topic_3674
- http://edamontology.org/topic_3295
tools:
- name: DeepMod2
  role: Detect per-read and per-site 5mC CpG methylation from Nanopore signal; produces
    output.per_site.aggregated used for correlation
  repo: https://github.com/WGLab/DeepMod2
- name: Dorado
  role: Basecall FAST5/POD5 with move tables (--emit-moves) to enable signal-to-basecall
    alignment required by DeepMod2
  repo: https://github.com/nanoporetech/dorado
- name: bedtools
  role: Intersect DeepMod2 per-site CpG methylation frequencies with short-read ground
    truth at shared CpG positions
- name: modkit
  role: Convert modBAM to bedMethyl summary tables for per-site methylation level
    manipulation
  repo: https://github.com/nanoporetech/modkit
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-024-45778-y
    title: A signal processing and deep learning framework for methylation detection
      using Oxford Nanopore sequencing
  source_content_hash: sha256:c0425f1dd4c9370394127fa00957144bc07ef8e9da45efb6b7f1872be06965bc
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/WGLab/DeepMod2
  tools:
  - DeepMod2
  - Dorado
  - bedtools
  - modkit
  edam_operation: http://edamontology.org/operation_3435
  edam_topics:
  - http://edamontology.org/topic_3674
  - http://edamontology.org/topic_3295
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# cross-platform-methylation-correlation-analysis

## Summary

Reproduce the benchmark validation that DeepMod2 per-site CpG methylation calls on Nanopore R9.4.1/R10.4.1 data correlate at r > 0.95 with short-read bisulfite sequencing / methylation microarray ground truth on the Ashkenazim trio (HG002, HG003, HG004), by intersecting per-site methylation frequencies across platforms and computing Pearson correlations.

## When to use

Use when validating Nanopore methylation caller output (per-site CpG frequencies from DeepMod2 detect on POD5/FAST5 data) against short-read bisulfite or methylation microarray ground truth via Pearson correlation.

## When NOT to use

- You lack matched short-read or microarray ground truth for the same sample — correlation cannot be computed without shared CpG positions across platforms
- Input is raw POD5/FAST5 signal that has not been basecalled with move tables — DeepMod2 requires move-table-aligned basecalls before per-site frequencies exist
- Per-read (not per-site) caller accuracy is the goal — the article separately benchmarks per-read F1 (~95%), and correlation analysis applies only to aggregated per-site methylation frequencies

## Inputs

- Ashkenazim trio (HG002, HG003, HG004) Nanopore signal files (POD5) for R9.4.1 and R10.4.1 flowcells
- Basecalled BAM with move tables (Dorado --emit-moves or Guppy --bam_out --moves_out), aligned to reference genome (minimap2 map-ont)
- Reference genome FASTA (ref.fa)
- Short-read bisulfite sequencing / methylation microarray ground-truth methylation levels (e.g., EpiQC study GSE186383)

## Outputs

- MOD_CALLS/output.per_site and MOD_CALLS/output.per_site.aggregated per-site methylation frequency tables
- Methylation-tagged BAM (MM/ML tags), e.g., MOD_CALLS/output.bam
- pearson_correlation_table.csv with columns sample, flowcell, pearson_r, n_cpg_sites

## How to apply

Run DeepMod2 detect on the Ashkenazim trio Nanopore signal files (R9.4.1 and R10.4.1 POD5 datasets) with the aligned BAM and reference genome to produce methylation-tagged BAMs plus per-site outputs (output.per_site and output.per_site.aggregated). Extract mod_percentage methylation frequencies from output.per_site.aggregated after removing low-coverage sites: <10X on the Nanopore side and ≥10X coverage required in the Ashkenazim trio short-read ground truth. Intersect DeepMod2 per-site CpG frequencies with matched bisulfite sequencing or methylation microarray ground truth (e.g., EpiQC study, GSE186383) at shared CpG positions using bedtools intersect. Compute the Pearson correlation coefficient between DeepMod2 mod_percentage and ground-truth methylation frequency per sample per flowcell, and write all coefficients to pearson_correlation_table.csv with columns sample, flowcell, pearson_r, n_cpg_sites. Note the benchmark excluded sites with intermediate methylation (EpiQC reported poor concordance at 20–80% methylation), so high inter-platform r reflects the mostly methylated/unmethylated sites retained.

## Related tools

- **DeepMod2** (Detect per-read and per-site 5mC CpG methylation from Nanopore signal; produces output.per_site.aggregated used for correlation) — https://github.com/WGLab/DeepMod2
- **Dorado** (Basecall FAST5/POD5 with move tables (--emit-moves) to enable signal-to-basecall alignment required by DeepMod2) — https://github.com/nanoporetech/dorado
- **bedtools** (Intersect DeepMod2 per-site CpG methylation frequencies with short-read ground truth at shared CpG positions)
- **modkit** (Convert modBAM to bedMethyl summary tables for per-site methylation level manipulation) — https://github.com/nanoporetech/modkit

## Examples

```
python deepmod2 detect --bam reads.bam --input INPUT_DIR --model bilstm_r9.4.1 --file_type POD5 --threads 16 --ref ref.fa --output MOD_CALLS
```

## Evaluation signals

- Every Pearson r in pearson_correlation_table.csv must exceed 0.95, matching the reported benchmark (95–97.35% genome-wide CpG correlation for DeepMod2 BiLSTM on R9.4.1 and R10.4.1 Ashkenazim trio datasets)
- Coverage filtering verified: no CpG site included with <10X Nanopore coverage or <10X short-read ground-truth coverage
- output.per_site.aggregated exists and contains both-strand-aggregated per-site mod_percentage values for each sample/flowcell combination
- n_cpg_sites column is populated and consistent with the number of CpGs shared between DeepMod2 calls and ground truth after intersection
- Methylation-tagged BAM carries valid MM/ML tags and can be visualized in IGV when colored by base modifications (5mC)

## Limitations

- Move tables only approximate signal-to-basecall alignment, so boundary signals between consecutive bases can be uninformative and degrade per-site frequencies
- DeepMod2 models show a slight bias to overestimate methylation at some CpG sites, possibly from residual heterogeneity in training labels
- Sites with intermediate methylation (20–80%) have poor inter-platform concordance even among short-read methods, so high correlation partly reflects exclusion of intermediate sites from ground truth
- Filtering intermediate per-read predictions (user-specified probability thresholds instead of the default 0.5) reduces coverage and statistical power, especially for low-coverage samples

## Evidence

- [results] DeepMod2 BiLSTM achieves the highest genome-wide CpG methylation-frequency correlation: [evidence span withheld — anchor status: not_found]
- [results] Ground truth coverage thresholds: "the ground truth only included sites that were mostly methylated (≥80% methylation) and mostly unmethylated (<20% methylation) with ample coverage (≥10X for HG002-4 and ≥5X for NIH3T3)"
- [results] Correlation analysis coverage filter: "after removing low coverage sites (<10X for Nanopore methylation callers and 10X/5X for Ashkenazim trio/NIH3T3 ground truth, respectively)"
- [readme] DeepMod2 detect command from README: [evidence span withheld — anchor status: external_doc]
- [readme] Per-site outputs produced by DeepMod2: [evidence span withheld — anchor status: external_doc]
- [results] Intermediate methylation sites excluded from evaluation: "EpiQC reported that sites with 20–80% methylation had poor concordance not only between ONT and short-reads method, but also among the different short-read methods"
