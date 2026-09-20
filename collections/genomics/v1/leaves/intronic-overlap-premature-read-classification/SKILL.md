---
name: intronic-overlap-premature-read-classification
description: Use when processing chromatin/nuclear-fraction Nanopore dRNA-seq alignments
  (BAM) with a matching gene annotation (GTF) and you need per-read premature/mature
  labels before estimating RNA processing and dynamics rates.
when_to_use_negative:
- Processing cytoplasmic or polysomal samples where premature calls are not interpretable
  — premature classifications should be removed there rather than generated
- Short-read RNA-seq data where single reads cannot span introns to establish >= 10
  base intronic overlap
- Input is already a gene-level count matrix without per-read alignment information
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: Nanodynamo
  role: Provides the fractionsReadsClassification.R workflow and internal functions
    that implement gene assignment and intronic-overlap-based premature/mature read
    classification
  repo: github.com/mfurla/Nanodynamo
- name: R
  role: Analysis language; GenomicRanges findOverlaps is used to assign reads to genes
    and test >= 10 base intronic overlap
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-024-51917-2
    title: Nanodynamo quantifies subcellular RNA dynamics revealing extensive coupling
      between steps of the RNA life cycle
  source_content_hash: sha256:f6718d3b773d688e85aee9e67ec138035b0d7d2f9de737fa03b3c9fd7b1b87a1
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
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# intronic-overlap-premature-read-classification

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Classifies each Nanopore direct RNA-seq read from subcellular fractions as premature or mature by intersecting gene-assigned read alignments (BAM) with annotated introns (GTF) in R, using a minimum intronic overlap of 10 bases. This per-read labeling underpins Nanodynamo's quantification of co- vs post-transcriptional processing.

## When to use

Use when processing chromatin/nuclear-fraction Nanopore dRNA-seq alignments (BAM) with a matching gene annotation (GTF) and you need per-read premature/mature labels before estimating RNA processing and dynamics rates.

## When NOT to use

- Processing cytoplasmic or polysomal samples where premature calls are not interpretable — premature classifications should be removed there rather than generated
- Short-read RNA-seq data where single reads cannot span introns to establish >= 10 base intronic overlap
- Input is already a gene-level count matrix without per-read alignment information

## Inputs

- Chromatin-fraction Nanopore direct RNA-seq read alignments (BAM)
- Matching gene annotation (GTF) including annotated intronic regions
- Sample fraction identity (chromatin/nuclear vs cytoplasmic vs polysomal)

## Outputs

- Per-read classification table (read ID, gene assignment, mature/premature status) saved as classified_reads.rds
- Premature/mature read proportions per gene/fraction

## How to apply

Load the chromatin-fraction dRNA-seq BAM alignments and the matching GTF annotation into R (GenomicRanges/Bioconductor). Assign each read to gene(s) via findOverlaps and discard reads overlapping more than one gene to keep unambiguous gene assignments. For each gene-assigned read, compute overlap with annotated introns and flag reads with intronic overlap >= 10 bases as premature; the remainder are mature. For cytoplasmic or polysomal samples, drop premature calls since they are only interpretable in chromatin/nuclear fractions. Write the per-read table (read ID, gene assignment, mature/premature status) to classified_reads.rds via the Nanodynamo fractionsReadsClassification.R workflow.

## Related tools

- **Nanodynamo** (Provides the fractionsReadsClassification.R workflow and internal functions that implement gene assignment and intronic-overlap-based premature/mature read classification) — github.com/mfurla/Nanodynamo
- **R** (Analysis language; GenomicRanges findOverlaps is used to assign reads to genes and test >= 10 base intronic overlap)

## Examples

```
Rscript fractionsReadsClassification.R
```

## Evaluation signals

- Output file classified_reads.rds exists and contains one labelled record per retained read
- Every retained read maps to exactly one gene (multi-gene overlapping reads were discarded)
- A read is labelled premature only if its intronic overlap is >= 10 bases
- Resulting premature/mature read proportions reproduce the fractions-reads-classification trends reported in Figures 3C, S10 and S17
- No premature classifications remain in cytoplasmic or polysomal sample outputs

## Limitations

- Reads overlapping more than one gene are discarded, so ambiguous multi-gene loci lose reads
- Premature calls are only interpretable in chromatin/nuclear fractions and must be removed for cytoplasmic/polysomal samples
- The 10-base intronic overlap threshold ties sensitivity to the accuracy and completeness of the supplied intron annotation

## Evidence

- [full_text] premature classification rule: [evidence span withheld — anchor status: not_found]
