---
name: cross-dataset-read-depth-normalization
description: Use when circRNA BSJ or isoform counts must be compared across independent
  datasets (e.g., isoCirc nanopore libraries vs. short-read RNA-seq) generated at
  different sequencing depths over multiple tissues.
when_to_use_negative:
- Comparing circRNA expression within a single sample or between matched replicates
  sequenced at the same depth — depth normalization across independent datasets is
  not needed
- Short-read-only circRNA studies where the goal is full-length isoform reconstruction;
  the article notes short-read RNA-seq does not determine full-length circRNA sequences
  or internal alternative splicing events
- Subcellular-fraction RNA data; the article states isoCirc was applied only to whole
  cells and tissues
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: isoCirc
  role: Produces the per-tissue 35-column isocirc.out tables whose readCount column
    supplies the full-length isoform counts to be depth-normalized and correlated
  repo: https://github.com/Xinglab/isoCirc
- name: Python
  role: Used to aggregate isocirc.out tables across tissues, normalize BSJ numbers
    by sequencing depth, and compute the Pearson correlation against short-read BSJ
    counts
- name: pip
  role: Installs the isoCirc pipeline (pip install isocirc) that generates the isocirc.out
    tables used for cross-dataset counting
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-020-20459-8
    title: isoCirc catalogs full-length circular RNA isoforms in human transcriptomes
  source_content_hash: sha256:1f211950f5d42d19d7a12db2a281d233f4cfe0f7a30b4772a4a7bd42ccef12a6
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Xinglab/isoCirc
  tools:
  - Python
  - bedtools
  - isoCirc
  - minimap2
  - pip
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# cross-dataset-read-depth-normalization

## Summary

Normalizes circRNA back-splice junction (BSJ) counts by sequencing depth so that isoCirc long-read catalogs and short-read RNA-seq BSJ detections can be compared across independently sequenced human tissues. Applied to reproduce the tissue-level agreement (Pearson r ≈ 0.86) between isoCirc and short-read circRNA complexity.

## When to use

Use when circRNA BSJ or isoform counts must be compared across independent datasets (e.g., isoCirc nanopore libraries vs. short-read RNA-seq) generated at different sequencing depths over multiple tissues.

## When NOT to use

- Comparing circRNA expression within a single sample or between matched replicates sequenced at the same depth — depth normalization across independent datasets is not needed
- Short-read-only circRNA studies where the goal is full-length isoform reconstruction; the article notes short-read RNA-seq does not determine full-length circRNA sequences or internal alternative splicing events
- Subcellular-fraction RNA data; the article states isoCirc was applied only to whole cells and tissues

## Inputs

- 35-column isocirc.out tables (per-tissue, with readCount column) from isoCirc nanopore runs on 12 human tissues
- Short-read RNA-seq circRNA BSJ count table (article dataset: BIGD PRJCA000751)
- Per-dataset sequencing depth values for normalization

## Outputs

- Per-tissue full-length circRNA isoform counts at read >= 1 (testis 115,348; prostate 18,312; skeletal muscle 19,261 in the reference run)
- Read-depth-normalized BSJ numbers per tissue for each dataset
- Pearson correlation coefficient between isoCirc and short-read BSJ numbers across the eight shared tissues (expected ≈ 0.86)

## How to apply

First generate or obtain the 35-column isocirc.out tables for each tissue by running the isocirc pipeline on the nanopore FASTA/FASTQ inputs with a reference genome FASTA, gene annotation GTF, and circRNA annotation BED/GTF (dependencies: bedtools >= v2.27.0 and minimap2 >= 2.11). Aggregate per-tissue full-length isoform counts from the readCount column at the read >= 1 cutoff (expected values from the article: testis 115,348; prostate 18,312; skeletal muscle 19,261). Next, normalize the per-tissue circRNA BSJ numbers by sequencing depth before any cross-dataset comparison, since isoCirc is far from saturation at the depth used. Finally, restrict the comparison to tissues shared between datasets (eight shared tissues in the article) and compute the Pearson correlation between depth-normalized isoCirc BSJ numbers and short-read RNA-seq BSJ numbers (short-read data from BIGD PRJCA000751); accept the analysis if the correlation approximates 0.86.

## Related tools

- **isoCirc** (Produces the per-tissue 35-column isocirc.out tables whose readCount column supplies the full-length isoform counts to be depth-normalized and correlated) — https://github.com/Xinglab/isoCirc
- **Python** (Used to aggregate isocirc.out tables across tissues, normalize BSJ numbers by sequencing depth, and compute the Pearson correlation against short-read BSJ counts)
- **pip** (Installs the isoCirc pipeline (pip install isocirc) that generates the isocirc.out tables used for cross-dataset counting)

## Examples

```
cd isoCirc/test_data && isocirc -t 1 read_toy.fa chr16_toy.fa chr16_toy.gtf chr16_circRNA_toy.bed output
```

## Evaluation signals

- Per-tissue full-length isoform counts at read >= 1 reproduce the reported values: testis 115,348, prostate 18,312, skeletal muscle 19,261
- Pearson correlation of depth-normalized BSJ numbers across the eight shared tissues equals approximately 0.86
- isocirc.out tables contain the expected 35 columns including the readCount column used for counting
- Normalization is applied before comparison (i.e., raw BSJ counts are never correlated directly across datasets)
- Comparison restricted to the eight tissues shared between the isoCirc and short-read (PRJCA000751) datasets

## Limitations

- isoCirc is far from saturation at current sequencing depth — more circRNA isoforms will be identified with deeper sequencing, so count-based comparisons are depth-sensitive
- Short-read RNA-seq does not experimentally determine full-length circRNA sequences or exact exonic compositions, limiting what a cross-dataset BSJ correlation can validate
- Low throughput and high per-sequence cost of long-read RNA-seq make deep transcriptome sampling prohibitively expensive, constraining depth-matched comparisons

## Evidence

- [abstract] Catalog of 107,147 full-length circRNA isoforms across 12 human tissues and one human cell line (HEK293): "we generate a comprehensive catalog of 107,147 full-length circRNA isoforms across 12 human tissues and one human cell line (HEK293), including 40,628 isoforms ≥500 nt in length"
- [discussion] isoCirc far from saturation at current depth: "At the current sequencing depth, we expect that isoCirc is far from saturation, and more circRNA isoforms will be identified with deeper sequencing"
- [discussion] Over 70-fold BSJ read enrichment in isoCirc libraries: "an over 70-fold enrichment of circRNA BSJ reads in isoCirc libraries compared to RNase R-treated short-read libraries"
- [readme] isoCirc dependencies: [evidence span withheld — anchor status: not_found]
