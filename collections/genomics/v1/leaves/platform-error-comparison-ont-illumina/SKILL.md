---
name: platform-error-comparison-ont-illumina
description: Use when a benchmark or analysis contains per-replicate consensus error
  measurements from both ONT and Illumina read sets (e.g. simulated SARS-CoV-2 reads
  at 30x/40x/50x) and the goal is to test whether ONT reconstruction error exceeds
  Illumina error at equal coverage.
when_to_use_negative:
- The dataset contains only one sequencing platform — there is no ONT/Illumina contrast
  to compute, so a within-platform coverage-trend analysis alone is more appropriate.
- The consensus caller or error metric differs from the ViralConsensus accuracy benchmark
  design (e.g. variant-calling pipelines like CoVpipe2 rather than direct consensus
  error rates), making fold-change comparisons non-comparable.
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_3673
tools:
- name: ViralConsensus
  role: Consensus caller used to reconstruct SARS-CoV-2 consensus genomes whose errors
    are compared across ONT and Illumina platforms and coverage levels
  repo: https://github.com/niemasd/ViralConsensus
- name: minimap2
  role: Read mapper whose BAM output can be piped directly into ViralConsensus to
    generate the alignments used for consensus calling
- name: samtools
  role: Used to convert mapper output to BAM (samtools view -b) in the ViralConsensus
    workflow before consensus calling
- name: pandas
  role: Used in the expert-analysis workflow to load and stratify the per-replicate
    error table by platform, lineage, and coverage
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioinformatics/btad317
    title: 'ViralConsensus: a fast and memory-efficient tool for calling viral consensus
      genome sequences directly from read alignment data'
  source_content_hash: sha256:8dc656b246b7602a0f06514d9e1f4bae571f979e2acc2b2ad2a4e6ba81cfd17a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/niemasd/ViralConsensus
  tools:
  - ViralConsensus
  - minimap2
  - pandas
  - samtools
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_3673
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 5
---

# Platform-error comparison of ONT vs Illumina consensus reconstruction

## Summary

Quantifies and compares viral consensus sequence reconstruction error between Oxford Nanopore (ONT) and Illumina sequencing platforms using ViralConsensus, benchmarked on simulated SARS-CoV-2 reads at matched coverage depths (30x, 40x, 50x).

## When to use

Use when a benchmark or analysis contains per-replicate consensus error measurements from both ONT and Illumina read sets (e.g. simulated SARS-CoV-2 reads at 30x/40x/50x) and the goal is to test whether ONT reconstruction error exceeds Illumina error at equal coverage.

## When NOT to use

- The dataset contains only one sequencing platform — there is no ONT/Illumina contrast to compute, so a within-platform coverage-trend analysis alone is more appropriate.
- The consensus caller or error metric differs from the ViralConsensus accuracy benchmark design (e.g. variant-calling pipelines like CoVpipe2 rather than direct consensus error rates), making fold-change comparisons non-comparable.

## Inputs

- Per-replicate consensus reconstruction error table (TSV/CSV) with columns for sequencing platform (Illumina/ONT), SARS-CoV-2 lineage (e.g. BA.1), and coverage depth (30x, 40x, 50x)
- Reference genome FASTA (e.g. SARS-CoV-2 NC_045512.2)
- Simulated read sets (FASTQ) from ONT and Illumina platforms per lineage and coverage level

## Outputs

- Per-condition mean error and per-replicate spread for ONT and Illumina at each coverage level
- ONT-vs-Illumina fold-change and Cohen's d effect size at each matched coverage
- Summary table written as error_trend_verification.tsv (TSV)

## How to apply

Load the per-replicate consensus error table with pandas, stratifying rows by sequencing platform (Illumina vs ONT), viral lineage, and simulated coverage depth (30x, 40x, 50x). For each platform at each matched coverage level, compute mean error and per-replicate spread, then compare ONT versus Illumina by reporting fold-change and a standardized effect size such as Cohen's d. Simultaneously check the monotonic decrease of error across 30x > 40x > 50x within each platform, since coverage-matched comparison is only meaningful if the coverage trend holds. The consensus sequences underlying these errors are produced with ViralConsensus, which can be piped directly from a read mapper (e.g. minimap2) via standard input for on-the-fly calling. Compile verdicts and effect sizes into a single TSV summary; because the paper reports no numeric threshold, acceptance rests on trend direction and consistently signed, non-negligible effect sizes across replicates.

## Related tools

- **ViralConsensus** (Consensus caller used to reconstruct SARS-CoV-2 consensus genomes whose errors are compared across ONT and Illumina platforms and coverage levels) — https://github.com/niemasd/ViralConsensus
- **minimap2** (Read mapper whose BAM output can be piped directly into ViralConsensus to generate the alignments used for consensus calling)
- **samtools** (Used to convert mapper output to BAM (samtools view -b) in the ViralConsensus workflow before consensus calling)
- **pandas** (Used in the expert-analysis workflow to load and stratify the per-replicate error table by platform, lineage, and coverage)

## Examples

```
minimap2 -t 1 -a -x sr reference.fas reads.fastq.gz | viral_consensus -i - -r reference.fas -o consensus.fas
```

## Evaluation signals

- ONT consensus error exceeds Illumina error at each matched coverage level (30x, 40x, 50x) with a consistently positive Cohen's d.
- Within each platform, error decreases monotonically: error_50x < error_40x < error_30x.
- Effect sizes (fold-change and Cohen's d) are non-negligible and consistently signed across replicates, since no numeric threshold is reported in the paper.
- BA.1 Illumina-replicate errors sit noticeably above other Illumina lineages, confirming the lineage-level elevated-error observation.
- Output TSV (error_trend_verification.tsv) parses cleanly and contains one verdict row per tested trend.

## Limitations

- The paper reports no numeric significance threshold, so verdicts rest on qualitative consistency of trend direction and effect-size sign rather than predefined cutoffs.
- ViralConsensus memory consumption scales linearly with reference genome length, so ultra-long references may be infeasible (though SARS-CoV-2 is small).
- Coverage comparisons are grounded in simulated reads at 30x/40x/50x; extrapolation to real-world ONT/Illumina error profiles is untested in this context.

## Evidence

- [methods] Piping minimap2 output into ViralConsensus: "minimap2 -t 1 -a -x sr reference.fas reads.fastq.gz | viral_consensus -i - -r reference.fas -o consensus.fas"
- [abstract] On-the-fly consensus calling design: "unlike existing methods, ViralConsensus can pipe data directly from a read mapper via standard input and performs viral consensus calling on-the-fly"
- [readme] Tool availability and implementation: "ViralConsensus is written in C++ and depends on htslib"
- [intro] Memory scaling caveat: "memory consumption scales linearly with reference genome length, meaning ultra-long reference genomes (e.g. human chromosomes) may require prohibitively large amounts of memory"
