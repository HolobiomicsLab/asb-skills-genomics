---
name: eicircrna-retained-intron-identification
description: Use when full-length circRNA isoform structures from isocirc.out (blockType/blockStarts/blockSize
  with readCount) must be compared to GTF exon coordinates to classify internal alternative
  splicing and EIciRNA retained introns.
when_to_use_negative:
- Input is short-read RNA-seq data only — short reads do not experimentally determine
  full-length circRNA sequences or internal alternative splicing events, so EIciRNA
  retained introns cannot be reliably called
- Goal is BSJ discovery or isoform catalog construction rather than classifying internal
  splicing within already-characterized full-length circRNA isoforms
- Only candidate BSJ coordinates (circRNA.bed) are available without the per-isoform
  block structure of isocirc.out
edam_operation: http://edamontology.org/operation_0226
edam_topics:
- http://edamontology.org/topic_0102
- http://edamontology.org/topic_3308
tools:
- name: isoCirc
  role: Produces isocirc.out containing full-length circRNA isoforms with readCount,
    blockType, blockStarts, and blockSize fields that this skill parses for event
    classification
  repo: https://github.com/Xinglab/isoCirc
- name: Python
  role: Used to parse isocirc.out, compare isoform block structures against GTF exon
    coordinates, and tally events per category and cutoff
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-020-20459-8
    title: isoCirc catalogs full-length circular RNA isoforms in human transcriptomes
  source_content_hash: sha256:7409768ef4662331f189b3f00355012ce97590cb48152a5d79a2ab986cafc945
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Xinglab/isoCirc
  tools:
  - Python
  - isoCirc
  edam_operation: http://edamontology.org/operation_0226
  edam_topics:
  - http://edamontology.org/topic_0102
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# eicircrna-retained-intron-identification

## Summary

Classifies internal alternative splicing events within full-length circRNA isoforms into skipped exon, alternative 5' splice site, alternative 3' splice site, and retained intron categories using isoCirc's isocirc.out isoform structures, and marks retained-intron events as EIciRNAs.

## When to use

Use when full-length circRNA isoform structures from isocirc.out (blockType/blockStarts/blockSize with readCount) must be compared to GTF exon coordinates to classify internal alternative splicing and EIciRNA retained introns.

## When NOT to use

- Input is short-read RNA-seq data only — short reads do not experimentally determine full-length circRNA sequences or internal alternative splicing events, so EIciRNA retained introns cannot be reliably called
- Goal is BSJ discovery or isoform catalog construction rather than classifying internal splicing within already-characterized full-length circRNA isoforms
- Only candidate BSJ coordinates (circRNA.bed) are available without the per-isoform block structure of isocirc.out

## Inputs

- isocirc.out full-length circRNA isoform table (readCount, blockType with E/I/N codes, blockStarts, blockSize)
- Gene annotation GTF with exon coordinates
- isoCirc catalog across 12 human tissues plus HEK293 cell line

## Outputs

- Summary table of internal alternative splicing event counts per category (SE, A5SS, A3SS, RI) and per read-count cutoff (>=2, >=5, >=10)
- List of retained-intron events classified as EIciRNAs
- Count of retained introns observed at the readCount >=10 cutoff

## How to apply

Load full-length circRNA isoforms from isocirc.out for the 12 tissues plus HEK293, parsing blockType (E: exon, I: intron, N: intergenic), blockStarts, and blockSize to recover each isoform's exact exonic composition. Compare each isoform's internal block structure against annotated exon coordinates from the gene annotation GTF to assign each difference to one of four classes: skipped exon, alternative 5' splice site, alternative 3' splice site, or retained intron. Filter events by the readCount column of isocirc.out at three cutoffs (>=2, >=5, >=10 reads), requiring the minor isoform to have at least 2 isoCirc reads across all samples combined at the primary cutoff. Tally event counts per category and cutoff, and mark retained-intron events (introns retained between exons within the circle) as EIciRNA events. Rationale: only long-read isoCirc data determines exact full-length exonic composition, so EIciRNA retained introns identifiable here are invisible to short-read RNA-seq; the reproduced EIciRNA abundance is expected to show ~20-fold excess over a comparable short-read RNA-seq study.

## Related tools

- **isoCirc** (Produces isocirc.out containing full-length circRNA isoforms with readCount, blockType, blockStarts, and blockSize fields that this skill parses for event classification) — https://github.com/Xinglab/isoCirc
- **Python** (Used to parse isocirc.out, compare isoform block structures against GTF exon coordinates, and tally events per category and cutoff)

## Examples

```
isocirc -t 1 read_toy.fa chr16_toy.fa chr16_toy.gtf chr16_circRNA_toy.bed output
```

## Evaluation signals

- Reproduced total internal alternative splicing events at minor-isoform readCount >=2 equals 5,325 across the four splicing pattern types
- Retained-intron events marked as EIciRNAs total 720
- Retained introns at the readCount >=10 cutoff total 177
- Recovered exonic composition from blockType/blockStarts/blockSize matches annotated exon coordinates from the gene GTF
- EIciRNA retained-intron abundance shows the expected ~20-fold excess compared to the comparable short-read RNA-seq study

## Limitations

- isoCirc is far from saturation at the current sequencing depth, so more circRNA isoforms (and internal splicing events) would be identified with deeper sequencing
- Filtering criteria are stricter for BSJs involving novel splice sites than for known splice sites, which can affect event recovery
- isoCirc was applied only to whole cells and tissues; subcellular-fraction RNAs (e.g., chromatin-associated) were not analyzed

## Evidence

- [abstract] 720 retained-intron EIciRNA events from abstract: "We identify widespread alternative splicing events within the internal part of circRNAs, including 720 retained intron events corresponding to a class of exon-intron circRNAs (EIciRNAs)"
- [intro] Short-read RNA-seq limitation motivating long-read approach: "an inherent limitation of this approach is that short-read RNA-seq does not experimentally determine the full-length sequences and internal alternative splicing events within circRNAs"
- [readme] isoCirc pipeline description: "isocirc: circular RNA profiling and analysis using long-read sequencing"
- [readme] isoCirc dependencies for running the pipeline: "isoCirc is dependent on two open-source software packages: [`bedtools`(>= v2.27.0)]"
