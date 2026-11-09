---
name: short-read-rnaseq-bsj-read-mapping
description: Use when Illumina short-read RNA-seq (FASTQ) needs BSJ-spanning read
  counts per library to quantify circRNA back-splice junction proportions, e.g. as
  the comparator arm of an isoCirc enrichment analysis.
when_to_use_negative:
- Input data are isoCirc long-read (RCA + nanopore) reads — those are processed by
  the isoCirc pipeline, not short-read BSJ counting
- The goal is full-length circRNA isoform reconstruction; short-read RNA-seq cannot
  determine full-length circRNA sequences or internal alternative splicing
- BSJs already have adequate long-read support and no short-read comparator library
  is available
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: minimap2
  role: splice-aware aligner for mapping short reads to the reference genome (splice:sr
    preset for short-read RNA-seq); also a required isoCirc dependency (>= 2.11)
  repo: https://github.com/lh3/minimap2
- name: bedtools
  role: genomic interval operations supporting junction interval overlap during BSJ
    read counting (isoCirc dependency, >= v2.27.0)
  repo: https://github.com/Xinglab/isoCirc
- name: Python
  role: computes BSJ proportions, fold enrichment, and the Spearman correlation (scipy.stats.spearmanr)
    and writes circRNA_bsj_enrichment_stats.csv
  repo: https://github.com/Xinglab/isoCirc
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-020-20459-8
    title: isoCirc catalogs full-length circular RNA isoforms in human transcriptomes
  source_content_hash: sha256:071f279a671e4fe9f51838e835b685fad05c133312c02ca15d76fe0ac72241ed
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/lh3/minimap2
  tools:
  - Python
  - bedtools
  - isoCirc
  - minimap2
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.45454545454545453
n_claims: 11
---

# short-read-rnaseq-bsj-read-mapping

## Summary

Aligns short-read Illumina RNA-seq libraries (RNase R-treated and poly(A)-selected) to a reference genome and counts reads spanning circRNA back-splice junctions (BSJs), establishing the short-read baseline that isoCirc long-read BSJ enrichment is benchmarked against.

## When to use

Use when Illumina short-read RNA-seq (FASTQ) needs BSJ-spanning read counts per library to quantify circRNA back-splice junction proportions, e.g. as the comparator arm of an isoCirc enrichment analysis.

## When NOT to use

- Input data are isoCirc long-read (RCA + nanopore) reads — those are processed by the isoCirc pipeline, not short-read BSJ counting
- The goal is full-length circRNA isoform reconstruction; short-read RNA-seq cannot determine full-length circRNA sequences or internal alternative splicing
- BSJs already have adequate long-read support and no short-read comparator library is available

## Inputs

- Illumina short-read RNA-seq FASTQ files for RNase R-treated and poly(A)-selected HEK293 libraries
- Reference genome FASTA
- Gene annotation GTF (for splice-aware alignment and junction definition)
- isoCirc per-isoform BSJ read counts (isocirc.out) for the matched comparison set

## Outputs

- BSJ-spanning read counts per short-read library
- Per-library BSJ-read proportions (BSJ reads / total reads)
- Fold enrichment of isoCirc vs RNase R-treated short-read BSJ proportion
- Spearman correlation of BSJ read counts across platforms (restricted to count >= 2 BSJs)
- circRNA_bsj_enrichment_stats.csv

## How to apply

Align each Illumina RNA-seq library (RNase R-treated and poly(A)-selected) to the reference genome using a splice-aware short-read mapper, then identify reads whose mates span a circular back-splice junction and count them per library. Compute the BSJ-read proportion per library as BSJ reads divided by total reads, which for the HEK293 isoCirc benchmark lands near ~0.045-0.048% for RNase R-treated and ~0.001% for poly(A)-selected libraries. For cross-platform comparison, restrict to BSJs detected at read count >= 2 in both the isoCirc output (isocirc.out) and the short-read data, and compute Spearman correlation of BSJ read counts (target ~0.5) with scipy.stats.spearmanr. Fold enrichment of isoCirc over RNase R-treated short-read proportions should exceed 70, per the published comparison.

## Related tools

- **minimap2** (splice-aware aligner for mapping short reads to the reference genome (splice:sr preset for short-read RNA-seq); also a required isoCirc dependency (>= 2.11)) — https://github.com/lh3/minimap2
- **bedtools** (genomic interval operations supporting junction interval overlap during BSJ read counting (isoCirc dependency, >= v2.27.0)) — https://github.com/Xinglab/isoCirc
- **Python** (computes BSJ proportions, fold enrichment, and the Spearman correlation (scipy.stats.spearmanr) and writes circRNA_bsj_enrichment_stats.csv) — https://github.com/Xinglab/isoCirc

## Examples

```
minimap2 -ax splice:sr ref.fa r1.fq r2.fq > aln.sam
```

## Evaluation signals

- RNase R-treated Illumina BSJ-read proportion falls in the reported ~0.045-0.048% range
- poly(A)-selected Illumina BSJ-read proportion is near ~0.001%
- isoCirc BSJ-read proportion falls in 3.5-4.0%, giving fold enrichment over RNase R short-read > 70
- Spearman correlation of BSJ read counts (BSJs with count >= 2 in both platforms) is approximately 0.5
- circRNA_bsj_enrichment_stats.csv contains the three proportions, fold enrichment, and Spearman correlation

## Limitations

- Short-read RNA-seq does not experimentally determine full-length circRNA sequences or internal alternative splicing, so BSJ counts are the only comparable signal
- poly(A)-selected libraries capture very few BSJ reads (~0.001%), making proportion estimates noisy
- Cross-platform correlation is modest (~0.5), so per-BSJ abundance agreement is limited even with read count >= 2 filtering

## Evidence

- [discussion] Short-read arm counts and proportions vs isoCirc: "an over 70-fold enrichment of circRNA BSJ reads in isoCirc libraries compared to RNase R-treated short-read libraries"
- [abstract] Short-read cannot recover full-length circRNA sequence: "an inherent limitation of short-read RNA-seq is that it does not experimentally determine the full-length sequences and exact exonic compositions of circRNAs"
- [readme] minimap2 short-read RNA-seq alignment command: [evidence span withheld — anchor status: external_doc]
- [readme] isoCirc dependency versions for the shared toolchain: "isoCirc is dependent on two open-source software packages: [`bedtools`(>= v2.27.0)]"
- [methods] Python is the scripting environment for the downstream statistics: "**isoCirc** is written with `python`, please use `pip` to install **isoCirc**"
