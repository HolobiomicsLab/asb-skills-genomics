---
name: sensitivity-specificity-fscore-computation
description: Use when benchmarking gene-fusion detection tools on data with known
  ground truth (e.g., simulated Iso-seq datasets with spiked-in fusions) and you need
  sensitivity, specificity, and F-score per caller.
when_to_use_negative:
- Input data lacks ground-truth fusion annotations — metrics cannot be computed without
  a known truth set
- The analysis goal is detecting fusions in real (non-simulated) tumor data, where
  sensitivity/specificity cannot be directly estimated and experimental validation
  (e.g., ~86% validation rate reported for TAGET) is the appropriate check
edam_operation: http://edamontology.org/operation_3649
edam_topics:
- http://edamontology.org/topic_0080
- http://edamontology.org/topic_3170
tools:
- name: TAGET
  role: Fusion detection caller being benchmarked; run via TAGET_fusion_2-3_ajust.r
    on minimap2/HISAT2 bed files
  repo: https://github.com/XiDsLab/TAGET
- name: SQANTI
  role: Independent long-read fusion/annotation comparator scored against the same
    ground truth
- name: JAFFAL
  role: Second comparator fusion caller scored against the same ground truth
- name: Minimap2
  role: Aligns simulated full-length transcripts to hg38 to produce bed files fed
    to TAGET
- name: HISAT2
  role: Second aligner producing bed files for TAGET fusion detection
- name: samtools
  role: Post-processing of alignments to generate per-sample bed files
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-023-41649-0
    title: 'TAGET: a toolkit for analyzing full-length transcripts from long-read
      sequencing'
  source_content_hash: sha256:6a160114505f7e908e5bbc0ee34950e7c8c8086d7dace0ac47eaa2b0db11b3ad
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/XiDsLab/TAGET
  tools:
  - HISAT2
  - JAFFAL
  - Minimap2
  - SQANTI
  - TAGET
  - samtools
  edam_operation: http://edamontology.org/operation_3649
  edam_topics:
  - http://edamontology.org/topic_0080
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# sensitivity-specificity-fscore-computation

## Summary

Scores every gene-fusion call from a long-read caller (e.g., TAGET, SQANTI, JAFFAL) against simulated Iso-seq ground truth and computes sensitivity, specificity, and F-score per dataset, as done in the TAGET benchmark of ten simulated Iso-seq datasets with 300 fusions each.

## When to use

Use when benchmarking gene-fusion detection tools on data with known ground truth (e.g., simulated Iso-seq datasets with spiked-in fusions) and you need sensitivity, specificity, and F-score per caller.

## When NOT to use

- Input data lacks ground-truth fusion annotations — metrics cannot be computed without a known truth set
- The analysis goal is detecting fusions in real (non-simulated) tumor data, where sensitivity/specificity cannot be directly estimated and experimental validation (e.g., ~86% validation rate reported for TAGET) is the appropriate check

## Inputs

- Simulated full-length transcript FASTA datasets with known fusion ground truth (e.g., ten datasets, 300 fusions each, 1-5 supporting transcripts per fusion)
- Per-tool predicted fusion call lists (TAGET, SQANTI, JAFFAL outputs)
- Ground-truth fusion annotation (partner genes and breakpoints)

## Outputs

- Per-tool sensitivity, specificity, and F-score per dataset
- Aggregated benchmark metrics table (e.g., fusion_benchmark_metrics.csv)

## How to apply

First generate ground truth by simulating full-length transcript FASTA datasets — in the TAGET benchmark, ten datasets each with 300 gene fusions built from random partner genes and breakpoints, spiked with indels/SNPs and 1-5 supporting transcripts per fusion on a GM12878 transcript background with CCLE fusions removed. Align each FASTA with Minimap2 and HISAT2 to hg38 (samtools post-processing), then run each fusion caller — TAGET via 'Rscript TAGET_fusion_2-3_ajust.r' with the minimap2 bed, HISAT2 bed, annotation stat file, and hg38.gtf; SQANTI and JAFFAL independently on the same datasets. Score every predicted fusion call against the simulated ground truth by matching called partner genes/breakpoints to the truth set, and compute sensitivity (true positives / ground-truth fusions), specificity (true negatives / negatives), and F-score (harmonic mean of precision and recall) per tool per dataset. Aggregate the per-tool metrics across all ten datasets into a metrics table (e.g., fusion_benchmark_metrics.csv) and compare callers; TAGET achieved the best sensitivity and F-score versus SQANTI and JAFFAL.

## Related tools

- **TAGET** (Fusion detection caller being benchmarked; run via TAGET_fusion_2-3_ajust.r on minimap2/HISAT2 bed files) — https://github.com/XiDsLab/TAGET
- **SQANTI** (Independent long-read fusion/annotation comparator scored against the same ground truth)
- **JAFFAL** (Second comparator fusion caller scored against the same ground truth)
- **Minimap2** (Aligns simulated full-length transcripts to hg38 to produce bed files fed to TAGET)
- **HISAT2** (Second aligner producing bed files for TAGET fusion detection)
- **samtools** (Post-processing of alignments to generate per-sample bed files)

## Examples

```
Rscript TAGET_fusion_2-3_ajust.r -j Jin_fusion_select.py -e STAT_select.py -l ${i}.fa.minimap2.bed -s ${i}.fa.hisat2.bed -a ${i}.fa.anno.tmp.stat -t hg38.gtf -f ${i}.fa -n ${i} -o ./output
```

## Evaluation signals

- Every simulated ground-truth fusion (300 per dataset) is matchable in the truth table — counts of truth fusions equal 300 per dataset before scoring
- Each predicted call is classified as true positive, false positive, or false negative against ground truth; TP + FN should equal the number of ground-truth fusions
- Sensitivity, specificity, and F-score are finite values in [0,1] for each tool on each of the ten datasets
- Per-tool metrics aggregate into a complete fusion_benchmark_metrics.csv covering TAGET, SQANTI, and JAFFAL across all ten datasets
- Cross-check: TAGET's reported result is the best sensitivity and F-score compared with SQANTI and JAFFAL on these simulated Iso-seq datasets

## Limitations

- Metric estimates depend on simulation assumptions (fusion expression levels, 1-5 supporting transcripts per fusion, indels/SNPs) and may not transfer to real tumor Iso-seq data
- Ground truth was built on a GM12878 transcript background with CCLE fusions removed, so the negative space reflects that specific background
- Results are contingent on alignment choices (Minimap2 and HISAT2 to hg38 with hg38.ensembl.gtf annotation)

## Evidence

- [other] On ten simulated Iso-seq datasets each containing 300 gene fusions at various expression levels, TAGET achieved the best sensitivity and F-score compared with SQANTI and JAFFAL.: [evidence span withheld — anchor status: not_found]
- [readme] Run TAGET fusion detection with alignment beds and annotation: "Rscript TAGET_fusion_2-3_ajust.r -j Jin_fusion_select.py -e STAT_select.py -l ${i}.fa.minimap2.bed -s ${i}.fa.hisat2.bed -a ${i}.fa.anno.tmp.stat -t hg38.gtf"
- [other] Score predicted calls against ground truth and compute sensitivity, specificity, F-score: [evidence span withheld — anchor status: not_found]
- [other] Simulation design with supporting transcripts per fusion: [evidence span withheld — anchor status: not_found]
- [discussion] TAGET and SQANTI sensitivity claim: "TAGET and SQANTI analyze long-reads independently and generally have higher sensitivity."
