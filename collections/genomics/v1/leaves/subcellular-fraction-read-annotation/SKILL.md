---
name: subcellular-fraction-read-annotation
description: Use when native RNA Nanopore reads from subcellular fractions need gene
  assignment and mature vs premature classification (e.g., inputs are BAM alignments
  plus a matching GTF) before quantifying fraction-specific transcript populations.
when_to_use_negative:
- When reads cannot be unambiguously assigned — reads overlapping more than one gene
  are discarded, so multi-mapper-heavy datasets will lose coverage
- When premature/mature calls are needed for cytoplasmic or polysomal fractions, since
  premature calls are only interpretable in chromatin/nuclear fractions and are removed
  there
- For short-read RNA-seq quantification, where read length prevents reliable intron-spanning
  premature/mature discrimination with a 10-base overlap rule
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: R (GenomicRanges)
  role: Implements findOverlaps-based assignment of reads to genes and computation
    of intronic overlap used for premature/mature classification
- name: Nanodynamo
  role: Provides the fractionsReadsClassification.R script that reproduces the read
    classification analysis reported in Figures 3C, S10 and S17
  repo: github.com/mfurla/Nanodynamo
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-024-51917-2
    title: Nanodynamo quantifies subcellular RNA dynamics revealing extensive coupling
      between steps of the RNA life cycle
  source_content_hash: sha256:57e262815c071558629943f58c549abdcb2e7914a2adbe5ce79d0f75c4429931
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/mfurla/Nanodynamo
  tools:
  - Nanodynamo
  - R
  - R (GenomicRanges)
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 6
---

# subcellular-fraction-read-annotation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Annotates Nanopore direct RNA-seq reads from subcellular fractions (chromatin/nuclear, cytoplasmic, polysomal) by overlap with gene annotations, then classifies each read as premature or mature based on intronic overlap, as implemented in Nanodynamo's fractionsReadsClassification workflow.

## When to use

Use when native RNA Nanopore reads from subcellular fractions need gene assignment and mature vs premature classification (e.g., inputs are BAM alignments plus a matching GTF) before quantifying fraction-specific transcript populations.

## When NOT to use

- When reads cannot be unambiguously assigned — reads overlapping more than one gene are discarded, so multi-mapper-heavy datasets will lose coverage
- When premature/mature calls are needed for cytoplasmic or polysomal fractions, since premature calls are only interpretable in chromatin/nuclear fractions and are removed there
- For short-read RNA-seq quantification, where read length prevents reliable intron-spanning premature/mature discrimination with a 10-base overlap rule

## Inputs

- BAM file of chromatin-fraction Nanopore direct RNA-seq read alignments
- Matching gene annotation in GTF format
- Additional fraction alignments (cytoplasmic, polysomal) when applicable

## Outputs

- classified_reads.rds — per-read classification table (read ID, gene assignment, mature/premature status)
- Premature/mature read proportions per gene/fraction for downstream Nanodynamo rate estimation

## How to apply

Load the chromatin-fraction dRNA-seq read alignments (BAM) and the matching gene annotation (GTF) into R and use GenomicRanges findOverlaps to assign each read to the gene(s) it overlaps, discarding reads that overlap more than one gene. For each gene-assigned read, compute overlap with annotated introns and flag reads with intronic overlap >= 10 bases as premature; remaining reads are mature. When processing cytoplasmic or polysomal samples, remove premature calls since they are only interpretable in chromatin/nuclear fractions. Write the per-read classification table (read ID, gene assignment, mature/premature status) to classified_reads.rds for downstream Nanodynamo modelling.

## Related tools

- **R (GenomicRanges)** (Implements findOverlaps-based assignment of reads to genes and computation of intronic overlap used for premature/mature classification)
- **Nanodynamo** (Provides the fractionsReadsClassification.R script that reproduces the read classification analysis reported in Figures 3C, S10 and S17) — github.com/mfurla/Nanodynamo

## Examples

```
Rscript fractionsReadsClassification.R
```

## Evaluation signals

- The output file classified_reads.rds exists and contains one labelled record per retained read
- Every retained read has exactly one gene assignment (reads overlapping multiple genes must be absent from the table)
- Premature/mature read proportions reproduce the fractions-reads-classification trends reported in Figures 3C, S10 and S17
- No premature calls appear among cytoplasmic or polysomal sample classifications

## Limitations

- Reads overlapping more than one gene are discarded, reducing coverage near overlapping genes
- Premature/mature distinction depends on the completeness of annotated introns in the GTF; misannotation propagates to classification errors
- Minimum intronic overlap of 10 bases means reads with marginal intronic overlap may be misclassified as mature

## Evidence

- [methods] Reads are assigned to genes by overlap; reads whose genes overlap annotated intronic regions are classified as premature and others as mature, with a minimum overlap of 10 bases required.: [evidence span withheld — anchor status: not_found]
- [methods] When processing cytoplasmic or polysomal samples, remove premature calls since they are only interpretable in chromatin/nuclear fractions.: [evidence span withheld — anchor status: not_found]
