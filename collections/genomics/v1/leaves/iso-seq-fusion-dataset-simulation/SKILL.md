---
name: iso-seq-fusion-dataset-simulation
description: Use when a gene-fusion detection pipeline for PacBio Iso-seq data needs
  a controlled benchmark with known fusion truth, or when comparing long-read fusion
  callers (e.g., TAGET vs SQANTI vs JAFFAL) on simulated datasets.
when_to_use_negative:
- The goal is detecting fusions in real tumor Iso-seq data rather than benchmarking
  caller performance on known ground truth
- Input data is short-read RNA-seq; the simulation and callers here target full-length
  PacBio Iso-seq (CCS) transcripts
- A published benchmark fusion dataset with experimental validation already exists
  for the comparison at hand
edam_operation: http://edamontology.org/operation_2423
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3512
tools:
- name: TAGET
  role: Primary fusion caller under evaluation; run via TAGET_fusion_2-3_ajust.r on
    Minimap2/HISAT2 bed inputs and hg38.gtf
  repo: https://github.com/XiDsLab/TAGET
- name: Minimap2
  role: Long-read aligner producing the minimap2 bed file consumed by TAGET fusion
    detection
- name: HISAT2
  role: Short/long aligner producing the hisat2 bed file consumed by TAGET fusion
    detection
- name: samtools
  role: Post-processing of alignments during bed file generation
- name: SQANTI
  role: Independent long-read fusion/annotation comparator run on the same simulated
    datasets
- name: JAFFAL
  role: Second comparator fusion caller run on the same simulated datasets
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-023-41649-0
    title: 'TAGET: a toolkit for analyzing full-length transcripts from long-read
      sequencing'
  source_content_hash: sha256:2c737a081834a6ee7fa7e629413622b19094ffadf7df96742ce73b089372bda4
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
  edam_operation: http://edamontology.org/operation_2423
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3512
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 15
---

# iso-seq-fusion-dataset-simulation

## Summary

Simulates full-length Iso-seq transcript datasets with known gene-fusion ground truth to benchmark long-read fusion callers such as TAGET against SQANTI and JAFFAL. Grounding fusions in a GM12878 transcript background (CCLE fusions removed) enables direct scoring of sensitivity, specificity, and F-score per caller.

## When to use

Use when a gene-fusion detection pipeline for PacBio Iso-seq data needs a controlled benchmark with known fusion truth, or when comparing long-read fusion callers (e.g., TAGET vs SQANTI vs JAFFAL) on simulated datasets.

## When NOT to use

- The goal is detecting fusions in real tumor Iso-seq data rather than benchmarking caller performance on known ground truth
- Input data is short-read RNA-seq; the simulation and callers here target full-length PacBio Iso-seq (CCS) transcripts
- A published benchmark fusion dataset with experimental validation already exists for the comparison at hand

## Inputs

- GM12878 transcript background (CCLE fusions removed) for simulation
- hg38 reference genome FASTA (hg38.fa) with Minimap2 and HISAT2 indexes
- hg38.ensembl.gtf annotation (pickle via gtf_db_make.py)
- Simulated full-length transcript FASTA per dataset (300 fusions, indels/SNPs, 1–5 supporting transcripts each)

## Outputs

- Per-sample alignment bed files ({i}.fa.minimap2.bed, {i}.fa.hisat2.bed)
- Per-tool fusion predictions (TAGET, SQANTI, JAFFAL) on each of ten simulated datasets
- Aggregated benchmark metrics: fusion_benchmark_metrics.csv with sensitivity, specificity, and F-score per tool

## How to apply

Simulate ten full-length transcript FASTA datasets, each containing 300 gene fusions built from random partner genes and breakpoints, spiked with indels/SNPs and 1–5 supporting transcripts per fusion, on a GM12878 transcript background with known CCLE fusions removed so the truth set is unambiguous. Align each simulated FASTA to hg38 (hg38.fa) with both Minimap2 and HISAT2, post-processing with samtools to produce per-sample bed files. Run TAGET fusion detection via 'Rscript TAGET_fusion_2-3_ajust.r' with the minimap2 bed, HISAT2 bed, annotation stat file, and hg38.gtf, then run SQANTI and JAFFAL on the same datasets as independent comparators. Score every predicted fusion call against the simulated ground truth, computing sensitivity, specificity, and F-score per tool per dataset, and aggregate into fusion_benchmark_metrics.csv for ranking (TAGET achieved the best sensitivity and F-score in this evaluation).

## Related tools

- **TAGET** (Primary fusion caller under evaluation; run via TAGET_fusion_2-3_ajust.r on Minimap2/HISAT2 bed inputs and hg38.gtf) — https://github.com/XiDsLab/TAGET
- **Minimap2** (Long-read aligner producing the minimap2 bed file consumed by TAGET fusion detection)
- **HISAT2** (Short/long aligner producing the hisat2 bed file consumed by TAGET fusion detection)
- **samtools** (Post-processing of alignments during bed file generation)
- **SQANTI** (Independent long-read fusion/annotation comparator run on the same simulated datasets)
- **JAFFAL** (Second comparator fusion caller run on the same simulated datasets)

## Examples

```
Rscript TAGET_fusion_2-3_ajust.r -j Jin_fusion_select.py -e STAT_select.py -l ${i}.fa.minimap2.bed -s ${i}.fa.hisat2.bed -a ${i}.fa.anno.tmp.stat -t hg38.gtf -f ${i}.fa -n ${i} -o ./output
```

## Evaluation signals

- Every predicted fusion call from each tool is matchable against the simulated ground truth of 300 fusions per dataset
- Ten replicate datasets all yield sensitivity/specificity/F-score values in fusion_benchmark_metrics.csv for TAGET, SQANTI, and JAFFAL
- Per-sample bed files exist for both Minimap2 ({i}.fa.minimap2.bed) and HISAT2 ({i}.fa.hisat2.bed) before TAGET is invoked
- Expected benchmark outcome: TAGET achieves the best sensitivity and F-score compared with SQANTI and JAFFAL across the ten simulated datasets
- Simulated truth is unambiguous because CCLE fusions were removed from the GM12878 background before spiking fusions

## Limitations

- Results depend on simulation assumptions (random partner genes/breakpoints, 1–5 supporting transcripts, spiked indels/SNPs) and may not reflect real tumor fusion spectra
- TAGET fusion detection requires HISAT2/Minimap2 (or GMAP) indexes of the hg38 reference plus a GTF pickle generated by gtf_db_make.py
- Iso-seq CCS-read filtering discards ~80% of CCS reads, many from true expressed isoforms, limiting gene detection in downstream analyses

## Evidence

- [other] On ten simulated Iso-seq datasets each containing 300 gene fusions at various expression levels, TAGET achieved the best sensitivity and F-score compared with SQANTI and JAFFAL.: [evidence span withheld — anchor status: not_found]
- [intro] TAGET comprises transcript alignment, annotation, quantification, and gene fusion detection components.: "This tool consists of four components, including transcript alignment, annotation, quantification, and gene fusion detection."
- [readme] README gives the exact TAGET fusion detection invocation with minimap2/hisat2 beds, stat file, and hg38.gtf.: "Rscript TAGET_fusion_2-3_ajust.r -j Jin_fusion_select.py -e STAT_select.py -l ${i}.fa.minimap2.bed -s ${i}.fa.hisat2.bed -a ${i}.fa.anno.tmp.stat -t hg38.gtf"
- [readme] Reference genome hg38.fa and Ensemble gtf files are required and must be indexed for HISAT2 and minimap2.: "Here we used hg38 to annotate transcripts,human reference hg38.fa and Ensemble gtf format files were needed. HISAT2 and minimap2 need to index this reference."
- [discussion] TAGET and SQANTI independently analyze long reads and generally have higher sensitivity.: "TAGET and SQANTI analyze long-reads independently and generally have higher sensitivity."
