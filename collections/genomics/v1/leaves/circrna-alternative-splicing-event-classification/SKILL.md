---
name: circrna-alternative-splicing-event-classification
description: Use when you have full-length circRNA isoform structures (e.g., isocirc.out
  with blockType/blockStarts/blockSize and readCount fields) plus a gene-annotation
  GTF, and need to count internal AS event types or retained-intron EIciRNA events.
when_to_use_negative:
- Input consists of short-read RNA-seq BSJ counts only — short reads do not determine
  full-length circRNA sequences or exact exonic compositions, so internal AS events
  cannot be classified from them
- CircRNAs were reconstructed by short-read-only methods applicable only to 200–500
  nt circRNAs, not longer full-length isoforms
- You need BSJ-level circRNA detection rather than internal isoform structure — use
  the isoCirc BSJ-detection filtering criteria instead
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_0080
- http://edamontology.org/topic_3308
tools:
- name: isoCirc
  role: Produces the full-length circRNA isoform catalog (isocirc.out) whose block
    structures and readCount column are parsed for event classification
  repo: https://github.com/Xinglab/isoCirc
- name: Python
  role: Language used to implement isoCirc and to parse blockType/blockStarts/blockSize,
    classify events against the GTF, and tally counts at cutoffs
  repo: https://github.com/Xinglab/isoCirc
- name: minimap2
  role: Upstream splice-aware aligner used by isoCirc to map consensus sequences from
    which isoform block structures are derived
  repo: https://github.com/lh3/minimap2
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-020-20459-8
    title: isoCirc catalogs full-length circular RNA isoforms in human transcriptomes
  source_content_hash: sha256:fe69f63883d223423f203ec4958f6341a6ad7c47609d3818afc7beb9965e4e97
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Xinglab/isoCirc
  tools:
  - Python
  - isoCirc
  - minimap2
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_0080
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 14
---

# circRNA-alternative-splicing-event-classification

## Summary

Classifies internal alternative splicing events within full-length circRNA isoforms into four categories (skipped exon, alternative 5' splice site, alternative 3' splice site, retained intron) by comparing isoCirc-parsed exonic block structures against annotated exon coordinates, then tallying events at isoform read-support cutoffs to reproduce the paper's 5,325-event catalog.

## When to use

Use when you have full-length circRNA isoform structures (e.g., isocirc.out with blockType/blockStarts/blockSize and readCount fields) plus a gene-annotation GTF, and need to count internal AS event types or retained-intron EIciRNA events.

## When NOT to use

- Input consists of short-read RNA-seq BSJ counts only — short reads do not determine full-length circRNA sequences or exact exonic compositions, so internal AS events cannot be classified from them
- CircRNAs were reconstructed by short-read-only methods applicable only to 200–500 nt circRNAs, not longer full-length isoforms
- You need BSJ-level circRNA detection rather than internal isoform structure — use the isoCirc BSJ-detection filtering criteria instead

## Inputs

- isocirc.out per-sample output files (with blockType, blockStarts, blockSize, readCount columns) for 12 human tissues plus HEK293
- Gene annotation GTF with exon coordinates (anno.gtf, as consumed by isoCirc)
- isocirc.bed / isocirc_stats.out supporting files from the isoCirc pipeline

## Outputs

- Tally of internal alternative splicing events per category (SE, A5SS, A3SS, RI) at read-support cutoffs of >=2, >=5, >=10
- Summary table of event counts (target: 5,325 total internal AS events at minor-isoform read >=2)
- Set of retained-intron events classified as EIciRNA events (target: 720 EIciRNA events; 177 retained introns at read>=10)

## How to apply

Parse the blockType (E: exon, I: intron, N: intergenic), blockStarts, and blockSize fields from isocirc.out for all 12 tissues plus HEK293 to recover each circRNA isoform's exact exonic composition. Compare each isoform's internal block structure against annotated exon coordinates from the gene-annotation GTF to classify internal events into the four categories: skipped exon, alternative 5' splice site, alternative 3' splice site, and retained intron. Filter events using the readCount column of isocirc.out at three support cutoffs (>=2, >=5, and >=10 isoform reads across all samples combined) and tally counts per category and cutoff into a summary table. Mark retained-intron events — introns retained between exons within the circle — as EIciRNA events and compare their abundance to short-read RNA-seq studies. Rationale: short-read RNA-seq does not experimentally determine full-length circRNA sequences, so internal AS events are only reliably recoverable from isoCirc full-length isoforms.

## Related tools

- **isoCirc** (Produces the full-length circRNA isoform catalog (isocirc.out) whose block structures and readCount column are parsed for event classification) — https://github.com/Xinglab/isoCirc
- **Python** (Language used to implement isoCirc and to parse blockType/blockStarts/blockSize, classify events against the GTF, and tally counts at cutoffs) — https://github.com/Xinglab/isoCirc
- **minimap2** (Upstream splice-aware aligner used by isoCirc to map consensus sequences from which isoform block structures are derived) — https://github.com/lh3/minimap2

## Examples

```
cd isoCirc/test_data && isocirc -t 1 read_toy.fa chr16_toy.fa chr16_toy.gtf chr16_circRNA_toy.bed output  # then parse isocirc.out readCount/blockType columns to classify events
```

## Evaluation signals

- Reproduced total internal AS events matches 5,325 when requiring the minor isoform to have at least 2 isoCirc reads across all samples combined
- Retained-intron/EIciRNA event count matches 720 events
- Retained-intron count at the read>=10 cutoff matches 177 events
- Event counts rise monotonically as the read-support cutoff is loosened from >=10 to >=5 to >=2
- Each classified event's category is consistent with the isoform's blockType annotation (E/I/N) versus GTF exon coordinates

## Limitations

- At the current sequencing depth, isoCirc is far from saturation; more circRNA isoforms (and thus events) will be identified with deeper sequencing
- Events are restricted to internal regions of circRNAs; only exons within the circle are compared — intergenic (N) blocks bound the internal portion
- Event calls depend on isoCirc's stringent false-positive filtering; criteria are much stricter for BSJs involving novel splice sites, which can affect event discovery
- EIciRNA abundance comparison relies on the expectation of ~20-fold excess versus short-read RNA-seq studies, which assumes full-length isoform detection sensitivity

## Evidence

- [abstract] We identify widespread alternative splicing events within the internal part of circRNAs, including 720 retained intron events corresponding to a class of exon-intron circRNAs (EIciRNAs): "We identify widespread alternative splicing events within the internal part of circRNAs, including 720 retained intron events corresponding to a class of exon-intron circRNAs (EIciRNAs)"
- [abstract] an inherent limitation of short-read RNA-seq is that it does not experimentally determine the full-length sequences and exact exonic compositions of circRNAs: "an inherent limitation of short-read RNA-seq is that it does not experimentally determine the full-length sequences and exact exonic compositions of circRNAs"
- [methods] isocirc: circular RNA profiling and analysis using long-read sequencing: "isocirc: circular RNA profiling and analysis using long-read sequencing"
- [readme] isoCirc is written with `python`, please use `pip` to install **isoCirc**:: "isoCirc is written with `python`, please use `pip` to install **isoCirc**:"
- [discussion] we use stringent criteria to remove potential false-positive detection of circRNA BSJs and full-length circRNA isoforms: "we use stringent criteria to remove potential false-positive detection of circRNA BSJs and full-length circRNA isoforms"
