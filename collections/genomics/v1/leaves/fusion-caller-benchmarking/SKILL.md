---
name: fusion-caller-benchmarking
description: Use when you must compare long-read gene-fusion detection methods on
  Iso-seq data with a known ground truth, or when selecting a fusion caller based
  on sensitivity/F-score evidence.
when_to_use_negative:
- Input is short-read RNA-seq data rather than full-length PacBio Iso-seq/HiFi transcripts
  — these callers and this benchmark design target long reads
- There is no ground truth (real tumor data without validated fusions) — sensitivity/F-score
  benchmarking requires simulated or truth-set fusions
- The goal is transcript annotation or isoform quantification rather than fusion-caller
  comparison
edam_operation: http://edamontology.org/operation_0279
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0085
tools:
- name: TAGET
  role: primary fusion caller under benchmark; run via TAGET_fusion_2-3_ajust.r using
    minimap2 and HISAT2 bed inputs plus hg38.gtf
  repo: github.com__XiDsLab__TAGET
- name: SQANTI
  role: independent long-read fusion/annotation comparator run on the same simulated
    datasets
- name: JAFFAL
  role: second comparator fusion caller run on the same simulated datasets
- name: Minimap2
  role: long-read aligner producing the minimap2 bed file consumed by TAGET fusion
    detection
- name: HISAT2
  role: short-read-style aligner producing the HISAT2 bed file consumed by TAGET fusion
    detection
- name: samtools
  role: post-processing of alignments to produce per-sample bed files
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-023-41649-0
    title: 'TAGET: a toolkit for analyzing full-length transcripts from long-read
      sequencing'
  source_content_hash: sha256:d5af74243700fb5a4ab5ce49802912a6dae27a1ddf6e25d918f1702328aecb8b
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - HISAT2
  - JAFFAL
  - Minimap2
  - SQANTI
  - TAGET
  - samtools
  edam_operation: http://edamontology.org/operation_0279
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0085
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# fusion-caller-benchmarking

## Summary

Benchmarks long-read gene-fusion callers (TAGET, SQANTI, JAFFAL) on simulated PacBio Iso-seq datasets with known fusion ground truth, scoring sensitivity, specificity, and F-score to quantify caller performance.

## When to use

Use when you must compare long-read gene-fusion detection methods on Iso-seq data with a known ground truth, or when selecting a fusion caller based on sensitivity/F-score evidence.

## When NOT to use

- Input is short-read RNA-seq data rather than full-length PacBio Iso-seq/HiFi transcripts — these callers and this benchmark design target long reads
- There is no ground truth (real tumor data without validated fusions) — sensitivity/F-score benchmarking requires simulated or truth-set fusions
- The goal is transcript annotation or isoform quantification rather than fusion-caller comparison

## Inputs

- simulated full-length transcript FASTA datasets (per sample, ~300 fusions each, 1-5 supporting reads per fusion)
- hg38 reference genome FASTA (hg38.fa)
- hg38 Ensembl GTF annotation (hg38.gtf / hg38.ensembl.gtf)
- per-sample minimap2 alignment bed files ({sample}.fa.minimap2.bed)
- per-sample HISAT2 alignment bed files ({sample}.fa.hisat2.bed)
- per-sample TAGET annotation stat file ({sample}.fa.anno.tmp.stat)
- simulated fusion ground-truth list

## Outputs

- per-caller fusion prediction call sets (TAGET, SQANTI, JAFFAL) per dataset
- per-dataset sensitivity, specificity, and F-score for each caller
- aggregated benchmark metrics table (e.g., fusion_benchmark_metrics.csv)

## How to apply

Simulate full-length transcript FASTA datasets (e.g., ten datasets, each with 300 gene fusions built from random partner genes and breakpoints, spiked with indels/SNPs and 1-5 supporting transcripts per fusion) on a GM12878 transcript background with CCLE fusions removed so no ground-truth fusion is leaked into the background. Align each simulated FASTA with Minimap2 and HISAT2 to hg38 (hg38.fa), post-process with samtools to produce per-sample bed files, then run TAGET fusion detection with 'Rscript TAGET_fusion_2-3_ajust.r' supplying the minimap2 bed, HISAT2 bed, annotation stat file, and hg38.gtf. Run SQANTI and JAFFAL independently on the same datasets as comparator callers. Score every predicted fusion call against the simulated ground truth and compute sensitivity, specificity, and F-score per tool per dataset, then aggregate metrics (e.g., fusion_benchmark_metrics.csv).

## Related tools

- **TAGET** (primary fusion caller under benchmark; run via TAGET_fusion_2-3_ajust.r using minimap2 and HISAT2 bed inputs plus hg38.gtf) — github.com__XiDsLab__TAGET
- **SQANTI** (independent long-read fusion/annotation comparator run on the same simulated datasets)
- **JAFFAL** (second comparator fusion caller run on the same simulated datasets)
- **Minimap2** (long-read aligner producing the minimap2 bed file consumed by TAGET fusion detection)
- **HISAT2** (short-read-style aligner producing the HISAT2 bed file consumed by TAGET fusion detection)
- **samtools** (post-processing of alignments to produce per-sample bed files)

## Examples

```
Rscript TAGET_fusion_2-3_ajust.r -j Jin_fusion_select.py -e STAT_select.py -l ${i}.fa.minimap2.bed -s ${i}.fa.hisat2.bed -a ${i}.fa.anno.tmp.stat -t hg38.gtf -f ${i}.fa -n ${i} -o ./output
```

## Evaluation signals

- Each simulated dataset yields a metrics row with sensitivity, specificity, and F-score for all three callers (TAGET, SQANTI, JAFFAL); missing values indicate a caller failed on that dataset
- Aggregated benchmark (e.g., fusion_benchmark_metrics.csv) should reproduce TAGET achieving the best sensitivity and F-score versus SQANTI and JAFFAL on ten simulated Iso-seq datasets
- Ground truth must contain exactly 300 fusions per dataset; per-caller calls scored against this list should include true positives from all expression levels and 1-5 supporting transcript groups
- Simulated background should be GM12878 transcripts with CCLE fusions removed, preventing caller predictions from matching background fusions counted as true positives
- Alignment intermediates exist per sample: {sample}.fa.minimap2.bed and {sample}.fa.hisat2.bed before TAGET fusion detection is invoked

## Limitations

- Benchmark conclusions rest on simulated fusions; simulation choices (fusion count, expression levels, indels/SNPs, 1-5 supporting reads) may not reflect real tumor Iso-seq complexity
- TAGET's discussion notes that TAGET and SQANTI analyze long reads independently and generally have higher sensitivity — relative caller behavior may depend on alignment strategy
- Metrics are limited to sensitivity, specificity, and F-score; other criteria (runtime, precision per fusion type) are not covered by this benchmark design

## Evidence

- [readme] TAGET fusion detection command from README: "Rscript TAGET_fusion_2-3_ajust.r -j Jin_fusion_select.py -e STAT_select.py -l ${i}.fa.minimap2.bed -s ${i}.fa.hisat2.bed -a ${i}.fa.anno.tmp.stat -t hg38.gtf -f ${i}.fa -n ${i} -o ./output"
- [readme] hg38 reference requirement: "Here we used hg38 to annotate transcripts,human reference hg38.fa and Ensemble gtf format files were needed. HISAT2 and minimap2 need to index this reference."
- [discussion] comparator sensitivity context: "TAGET and SQANTI analyze long-reads independently and generally have higher sensitivity."
- [intro] fusion detection as TAGET component: "This tool consists of four components, including transcript alignment, annotation, quantification, and gene fusion detection."
