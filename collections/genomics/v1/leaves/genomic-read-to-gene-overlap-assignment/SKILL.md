---
name: genomic-read-to-gene-overlap-assignment
description: Use when Nanopore dRNA-seq read alignments (BAM) must be assigned to
  genes and split into mature vs premature classes using GTF intron overlap, e.g.
  before quantifying RNA pools in subcellular fractions.
when_to_use_negative:
- Processing cytoplasmic or polysomal samples for premature calls — these are removed
  there because premature status is only interpretable in chromatin/nuclear fractions.
- Input is already a per-gene count matrix or the reads have already been assigned
  to genes — this skill's findOverlaps-based gene assignment and intron-overlap classification
  would be redundant.
- Reads that overlap more than one gene — these are discarded by design, so the skill
  cannot be used to force a unique gene assignment for ambiguous reads.
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_3168
tools:
- name: R
  role: Environment for the overlap-assignment script; GenomicRanges findOverlaps
    assigns reads to genes and quantifies intronic overlap.
  repo: github.com/mfurla/Nanodynamo
- name: Nanodynamo
  role: Provides fractionsReadsClassification.R, the R script that implements the
    read-to-gene assignment and premature/mature classification of native RNA Nanopore
    reads.
  repo: github.com/mfurla/Nanodynamo
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-024-51917-2
    title: Nanodynamo quantifies subcellular RNA dynamics revealing extensive coupling
      between steps of the RNA life cycle
  source_content_hash: sha256:391741fd288e5be4e240859dcb4b7d176667ac9366a602357f9a61f1e77b6f97
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
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# genomic-read-to-gene-overlap-assignment

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Assigns Nanopore direct RNA-seq reads to genes by genomic overlap (GenomicRanges findOverlaps in R) and classifies each read as mature or premature based on whether it overlaps annotated introns by at least 10 bases, as implemented in Nanodynamo's fractionsReadsClassification workflow.

## When to use

Use when Nanopore dRNA-seq read alignments (BAM) must be assigned to genes and split into mature vs premature classes using GTF intron overlap, e.g. before quantifying RNA pools in subcellular fractions.

## When NOT to use

- Processing cytoplasmic or polysomal samples for premature calls — these are removed there because premature status is only interpretable in chromatin/nuclear fractions.
- Input is already a per-gene count matrix or the reads have already been assigned to genes — this skill's findOverlaps-based gene assignment and intron-overlap classification would be redundant.
- Reads that overlap more than one gene — these are discarded by design, so the skill cannot be used to force a unique gene assignment for ambiguous reads.

## Inputs

- chromatin-fraction Nanopore dRNA-seq read alignments (BAM)
- matching gene annotation with intronic regions (GTF)
- cytoplasmic or polysomal Nanopore dRNA-seq read alignments (BAM), for downstream mature-read-only processing

## Outputs

- classified_reads.rds: per-read table with read ID, gene assignment, and mature/premature status (one labelled record per retained read)
- per-gene / per-fraction mature and premature read proportions used for fractions-reads-classification analysis

## How to apply

Load the chromatin-fraction Nanopore dRNA-seq read alignments (BAM) and the matching gene annotation (GTF) into R and use GenomicRanges findOverlaps to assign each read to the gene(s) it overlaps, discarding reads that overlap more than one gene to avoid ambiguous assignment. For each gene-assigned read, compute overlap with annotated introns and flag reads whose intronic overlap is >= 10 bases as premature; all other assigned reads are mature. When processing cytoplasmic or polysomal samples, remove premature calls because they are only interpretable in chromatin/nuclear fractions. Write the per-read classification table (read ID, gene assignment, mature/premature status) to classified_reads.rds. This classification feeds Nanodynamo's characterization of nascent and premature read fractions (the fractionsReadsClassification.R script), so its premature/mature proportions should be checked against the fractions-reads-classification results.

## Related tools

- **R** (Environment for the overlap-assignment script; GenomicRanges findOverlaps assigns reads to genes and quantifies intronic overlap.) — github.com/mfurla/Nanodynamo
- **Nanodynamo** (Provides fractionsReadsClassification.R, the R script that implements the read-to-gene assignment and premature/mature classification of native RNA Nanopore reads.) — github.com/mfurla/Nanodynamo

## Examples

```
Rscript fractionsReadsClassification.R
```

## Evaluation signals

- classified_reads.rds exists and contains one labelled record per retained read (read ID, gene assignment, mature/premature status).
- No retained read overlaps more than one gene — multi-gene-overlap reads must have been discarded during findOverlaps assignment.
- A read is flagged premature only when its overlap with annotated introns is >= 10 bases; shorter intronic overlaps remain mature.
- Resulting premature/mature read proportions reproduce the fractions-reads-classification trends reported in Figures 3C, S10 and S17.
- Cytoplasmic and polysomal classification tables contain no premature calls.

## Limitations

- Reads overlapping more than one gene are discarded, so ambiguous genomic loci lose their reads entirely.
- The >= 10-base intronic overlap threshold is a fixed heuristic; premature/mature proportions depend on the quality and completeness of the GTF intron annotation.
- Premature calls are only interpretable in chromatin/nuclear fractions and are removed for cytoplasmic and polysomal samples.

## Evidence

- [methods] Reads are assigned to genes by overlap; reads whose genes overlap annotated intronic regions are classified as premature and others as mature, with a minimum overlap of 10 bases required.: [evidence span withheld — anchor status: not_found]
- [readme] An R script to characterize nascent and premature reads fractions (fractionsReadsClassification.R): "An R script to characterize nascent and premature reads fractions (fractionsReadsClassification.R)"
- [readme] ~ _fractionsReadsClassification.R_: Figures 3C, S10, S17.: "~ _fractionsReadsClassification.R_: Figures 3C, S10, S17."
