---
name: cross-study-retained-intron-comparison
description: Use when tallying internal alternative splicing events within circRNAs
  from a long-read full-length isoform catalog (isocirc.out), or when quantitatively
  comparing EIciRNA/retained-intron circRNA abundance between long-read and short-read
  RNA-seq studies.
when_to_use_negative:
- Input is short-read RNA-seq data, which does not experimentally determine full-length
  circRNA sequences or exact exonic compositions, so internal event classification
  is not directly supported
- The goal is BSJ-only circRNA detection without full-length internal isoform structure
  — a plain back-splice junction caller is the appropriate choice
- Estimating absolute circRNA abundance de novo rather than benchmarking event counts;
  isoCirc is far from saturation at current depth, so counts are lower bounds
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: isoCirc
  role: Produces the full-length circRNA isoform catalog (isocirc.out) whose block
    structure and readCount column are parsed and tallied for event classification
  repo: https://github.com/Xinglab/isoCirc
- name: Python
  role: Language in which isoCirc is written and used for parsing isocirc.out and
    tallying event counts per category and cutoff
- name: minimap2
  role: Dependency of the isoCirc pipeline; maps consensus sequences underlying the
    high-confidence isoform calls used for event counting (>= 2.11)
  repo: https://github.com/lh3/minimap2
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-020-20459-8
    title: isoCirc catalogs full-length circular RNA isoforms in human transcriptomes
  source_content_hash: sha256:83ebc563238c823601e0c4285dd16a5303a6d7110e99b7de315b0265bca32001
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Xinglab/isoCirc
  tools:
  - Python
  - isoCirc
  - minimap2
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# cross-study-retained-intron-comparison

## Summary

Reproduces isoCirc's internal alternative splicing event counts within circRNAs (5,325 total at minor-isoform readCount>=2, spanning skipped exon, alternative 5' splice site, alternative 3' splice site, and retained intron) and benchmarks the 720 EIciRNA retained-intron events against comparable short-read RNA-seq studies (~20-fold excess expected).

## When to use

Use when tallying internal alternative splicing events within circRNAs from a long-read full-length isoform catalog (isocirc.out), or when quantitatively comparing EIciRNA/retained-intron circRNA abundance between long-read and short-read RNA-seq studies.

## When NOT to use

- Input is short-read RNA-seq data, which does not experimentally determine full-length circRNA sequences or exact exonic compositions, so internal event classification is not directly supported
- The goal is BSJ-only circRNA detection without full-length internal isoform structure — a plain back-splice junction caller is the appropriate choice
- Estimating absolute circRNA abundance de novo rather than benchmarking event counts; isoCirc is far from saturation at current depth, so counts are lower bounds

## Inputs

- isocirc.out full-length circRNA isoform call files for 12 human tissues plus HEK293 (blockType, blockStarts, blockSize, readCount columns)
- Gene annotation GTF with exon coordinates for the reference genome
- Published event counts from a comparable short-read RNA-seq circRNA study for the cross-study comparison

## Outputs

- Summary table of internal alternative splicing event counts per category (skipped exon, alternative 5' splice site, alternative 3' splice site, retained intron) and per read cutoff (>=2, >=5, >=10)
- Count of EIciRNA retained-intron events (720 at primary cutoff; 177 retained introns at read>=10)
- Cross-study fold-excess comparison of EIciRNA abundance (~20-fold) versus short-read RNA-seq

## How to apply

Parse each isocirc.out (12 human tissues plus HEK293), recovering each full-length circRNA isoform's exact exonic composition from the blockType (E: exon, I: intron, N: intergenic), blockStarts, and blockSize fields, and its support from the readCount column. Compare each isoform's internal block structure against the annotated exon coordinates in the gene annotation GTF to classify internal events into skipped exon, alternative 5' splice site, alternative 3' splice site, and retained intron categories. Filter events by minor-isoform readCount at three support cutoffs (>=2, >=5, >=10 reads) and tally per-category counts into a summary table; the headline result is 5,325 events at the readCount>=2 cutoff. Mark retained-intron events (introns retained between exons within the circle) as EIciRNA events — 720 at the primary cutoff — and compare their abundance to the short-read RNA-seq study, expecting roughly a ~20-fold excess in favor of isoCirc. Validation invariants: the reproduced totals must match 5,325 internal AS events, 720 EIciRNA retained-intron events, and 177 retained introns at the read>=10 cutoff.

## Related tools

- **isoCirc** (Produces the full-length circRNA isoform catalog (isocirc.out) whose block structure and readCount column are parsed and tallied for event classification) — https://github.com/Xinglab/isoCirc
- **Python** (Language in which isoCirc is written and used for parsing isocirc.out and tallying event counts per category and cutoff)
- **minimap2** (Dependency of the isoCirc pipeline; maps consensus sequences underlying the high-confidence isoform calls used for event counting (>= 2.11)) — https://github.com/lh3/minimap2

## Examples

```
isocirc -t 1 read_toy.fa chr16_toy.fa chr16_toy.gtf chr16_circRNA_toy.bed output
```

## Evaluation signals

- Total internal alternative splicing events at the readCount>=2 cutoff across all 12 tissues plus HEK293 equals 5,325
- Retained-intron (EIciRNA) event count equals 720 at the primary cutoff, and 177 retained introns persist at the read>=10 cutoff
- Counts decrease monotonically when recomputed at stricter support cutoffs (>=2 → >=5 → >=10 reads)
- Every classified event's internal exon/intron blocks reconcile with the annotated exon coordinates in the gene annotation GTF
- The long-read EIciRNA retained-intron abundance exceeds the short-read RNA-seq study by roughly a ~20-fold margin

## Limitations

- Short-read RNA-seq reference counts are indirect: short reads do not experimentally determine full-length circRNA sequences or exact exonic compositions, so the ~20-fold comparison is approximate
- Filtering criteria are much stricter for BSJs involving novel splice sites than for known splice sites, so events near unannotated splice sites may be undercounted
- At current sequencing depth isoCirc is far from saturation, meaning the reproduced event counts are lower bounds that will grow with deeper sequencing

## Evidence

- [abstract] We identify widespread alternative splicing events within the internal part of circRNAs, including 720 retained intron events corresponding to a class of exon-intron circRNAs (EIciRNAs): "We identify widespread alternative splicing events within the internal part of circRNAs, including 720 retained intron events corresponding to a class of exon-intron circRNAs (EIciRNAs)"
- [abstract] Short-read RNA-seq limitation: "an inherent limitation of short-read RNA-seq is that it does not experimentally determine the full-length sequences and exact exonic compositions of circRNAs"
- [readme] Dependency versions: "isoCirc is dependent on two open-source software packages: [`bedtools`(>= v2.27.0)]"
