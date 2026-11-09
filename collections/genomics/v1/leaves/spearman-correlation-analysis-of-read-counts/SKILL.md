---
name: spearman-correlation-analysis-of-read-counts
description: Use when comparing per-feature (e.g. BSJ) read-count quantitation across
  two sequencing platforms or library preparations and you need a rank-correlation
  coefficient restricted to features detected above a minimum count threshold.
when_to_use_negative:
- Your goal is full-length circRNA isoform reconstruction rather than cross-platform
  BSJ quantitation comparison — use the isoCirc full-length isoform cataloging workflow
  instead
- The feature counts are paired normally distributed measurements where Pearson correlation
  is appropriate, or one platform has an unbounded count scale requiring a different
  concordance metric
- Input data is subcellular-fraction RNA (e.g. chromatin-associated RNAs) — the article
  notes isoCirc was applied only to whole cells and tissues
edam_operation: http://edamontology.org/operation_0564
edam_topics:
- http://edamontology.org/topic_0080
- http://edamontology.org/topic_3308
tools:
- name: isoCirc
  role: Processes isoCirc long-read data to produce per-isoform BSJ read counts (isocirc.out)
    and summary statistics (isocirc_stats.out) used as one arm of the correlation
  repo: https://github.com/Xinglab/isoCirc
- name: minimap2
  role: Spliced long-read alignment of isoCirc consensus sequences against the reference
    genome within the isocirc pipeline
  repo: https://github.com/lh3/minimap2
- name: bedtools
  role: Dependency of the isocirc pipeline for interval operations on BSJ annotations
    and alignments
  repo: https://github.com/Xinglab/isoCirc
- name: Python
  role: Runs the correlation analysis via scipy.stats.spearmanr on paired BSJ read
    counts and writes circRNA_bsj_enrichment_stats.csv
  repo: https://github.com/Xinglab/isoCirc
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-020-20459-8
    title: isoCirc catalogs full-length circular RNA isoforms in human transcriptomes
  source_content_hash: sha256:b822ccb377168ff3f3a759bc8f51b95456f28179f2d2cb73900a997f90fd5300
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Xinglab/isoCirc
  tools:
  - Python
  - bedtools
  - isoCirc
  - minimap2
  edam_operation: http://edamontology.org/operation_0564
  edam_topics:
  - http://edamontology.org/topic_0080
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 5
---

# spearman-correlation-analysis-of-read-counts

## Summary

Rank-based correlation analysis (scipy.stats.spearmanr) of circRNA back-splice junction read counts between isoCirc long-read data and RNase R-treated short-read Illumina RNA-seq, used to quantify cross-platform agreement of BSJ quantitation in HEK293 cells.

## When to use

Use when comparing per-feature (e.g. BSJ) read-count quantitation across two sequencing platforms or library preparations and you need a rank-correlation coefficient restricted to features detected above a minimum count threshold.

## When NOT to use

- Your goal is full-length circRNA isoform reconstruction rather than cross-platform BSJ quantitation comparison — use the isoCirc full-length isoform cataloging workflow instead
- The feature counts are paired normally distributed measurements where Pearson correlation is appropriate, or one platform has an unbounded count scale requiring a different concordance metric
- Input data is subcellular-fraction RNA (e.g. chromatin-associated RNAs) — the article notes isoCirc was applied only to whole cells and tissues

## Inputs

- isocirc.out per-isoform BSJ read counts and isocirc_stats.out summary counts from the isoCirc HEK293 long-read pipeline run
- Aligned RNase R-treated Illumina RNA-seq library with BSJ-spanning read counts
- Aligned poly(A)-selected Illumina RNA-seq library with BSJ-spanning read counts
- Reference genome and annotation (ref.fa, anno.gtf, circRNA.bed/gtf) used for isocirc and short-read alignment

## Outputs

- circRNA_bsj_enrichment_stats.csv containing isoCirc, RNase R Illumina, and poly(A) Illumina BSJ-read proportions, the isoCirc/RNase R fold enrichment, and the Spearman correlation at read count >= 2

## How to apply

Run the isocirc pipeline (bedtools >= v2.27.0, minimap2 >= 2.11) on the deposited HEK293 long-read data to obtain per-isoform BSJ read counts from isocirc.out and summary counts from isocirc_stats.out; independently align RNase R-treated and poly(A)-selected Illumina RNA-seq to the reference genome and count reads spanning each BSJ. Compute per-library BSJ proportions (BSJ reads / total reads) for isoCirc (~3.5-4.0%), RNase R-treated Illumina (~0.045-0.048%), and poly(A)-selected Illumina (~0.001%), then calculate the isoCirc-over-RNase R fold enrichment (>70). For the correlation, restrict to BSJs detected at read count >= 2 in both the isoCirc and RNase R short-read data and call scipy.stats.spearmanr on the paired read counts; the expected value is approximately 0.5. Write the three proportions, the fold enrichment, and the Spearman correlation to circRNA_bsj_enrichment_stats.csv.

## Related tools

- **isoCirc** (Processes isoCirc long-read data to produce per-isoform BSJ read counts (isocirc.out) and summary statistics (isocirc_stats.out) used as one arm of the correlation) — https://github.com/Xinglab/isoCirc
- **minimap2** (Spliced long-read alignment of isoCirc consensus sequences against the reference genome within the isocirc pipeline) — https://github.com/lh3/minimap2
- **bedtools** (Dependency of the isocirc pipeline for interval operations on BSJ annotations and alignments) — https://github.com/Xinglab/isoCirc
- **Python** (Runs the correlation analysis via scipy.stats.spearmanr on paired BSJ read counts and writes circRNA_bsj_enrichment_stats.csv) — https://github.com/Xinglab/isoCirc

## Examples

```
python -c "from scipy.stats import spearmanr; rho,_=spearmanr(isocirc_bsj_counts, rnaser_bsj_counts)"
```

## Evaluation signals

- isoCirc BSJ-read proportion falls in the reported 3.5-4.0% range
- RNase R-treated Illumina proportion falls in 0.045-0.048%; poly(A)-selected Illumina proportion near 0.001%
- Fold enrichment of isoCirc over RNase R-treated Illumina BSJ proportion exceeds 70
- Spearman correlation of BSJ read counts (restricted to read count >= 2 in both platforms) is approximately 0.5
- circRNA_bsj_enrichment_stats.csv contains all five summary values with no missing fields

## Limitations

- Spearman correlation of only ~0.5 indicates moderate rank agreement; BSJ quantitation between isoCirc and short-read data is correlated but not concordant
- At current sequencing depth isoCirc is far from saturation, so BSJ counts (and hence the correlation) may shift with deeper sequencing
- Long-read RNA-seq has relatively low throughput and high per-sequence cost, limiting depth of BSJ sampling
- Comparison is limited to HEK293 whole-cell libraries (RNase R-treated and poly(A)-selected Illumina); subcellular fractions were not analyzed

## Evidence

- [discussion] an over 70-fold enrichment of circRNA BSJ reads in isoCirc libraries compared to RNase R-treated short-read libraries: "an over 70-fold enrichment of circRNA BSJ reads in isoCirc libraries compared to RNase R-treated short-read libraries"
- [methods] isocirc: circular RNA profiling and analysis using long-read sequencing: "isocirc: circular RNA profiling and analysis using long-read sequencing"
- [readme] isoCirc is dependent on two open-source software packages: bedtools(>= v2.27.0) and minimap2(>= 2.11): "isoCirc is dependent on two open-source software packages: [`bedtools`(>= v2.27.0)]"
- [other] Restrict to BSJs detected at read count >= 2 in both isoCirc and RNase R short-read data and compute the Spearman correlation: [evidence span withheld — anchor status: not_found]
