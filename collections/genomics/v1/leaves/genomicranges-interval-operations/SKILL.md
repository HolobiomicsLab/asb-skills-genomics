---
name: genomicranges-interval-operations
description: Use when dRNA-seq read alignments (BAM) must be assigned to genes and
  split into premature vs mature classes by intronic overlap, as in Nanodynamo's fractionsReadsClassification
  step.
when_to_use_negative:
- When premature calls would be interpreted in cytoplasmic or polysomal samples, where
  they are only interpretable in chromatin/nuclear fractions
- When reads cannot be unambiguously assigned because they overlap more than one gene
  (such reads are discarded rather than classified)
- For short-read RNA-seq where isoform-level intronic overlap logic of Nanodynamo's
  native-RNA classification is not designed to apply
edam_operation: http://edamontology.org/operation_0233
edam_topics:
- http://edamontology.org/topic_0102
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: R
  role: Host language for the GenomicRanges findOverlaps-based read classification
    (fractionsReadsClassification.R)
  repo: https://github.com/mfurla/Nanodynamo
- name: Nanodynamo
  role: Pipeline whose fractionsReadsClassification step consumes the premature/mature
    read classifications produced by these interval operations
  repo: https://github.com/mfurla/Nanodynamo
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-024-51917-2
    title: Nanodynamo quantifies subcellular RNA dynamics revealing extensive coupling
      between steps of the RNA life cycle
  source_content_hash: sha256:0f85bbe1f9fdfe9a4a8c7d485a23069badf47fb043a8f80a1be898ed1734091d
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/mfurla/Nanodynamo
  repo_url: https://github.com/mfurla/Nanodynamo
  tools:
  - Nanodynamo
  - R
  edam_operation: http://edamontology.org/operation_0233
  edam_topics:
  - http://edamontology.org/topic_0102
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# GenomicRanges interval operations

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Uses R/Bioconductor GenomicRanges findOverlaps to assign Nanopore direct RNA-seq reads to genes and classify each read as premature (≥10 bases of annotated intronic overlap) or mature, enabling Nanodynamo's per-fraction premature/mature quantification.

## When to use

Use when dRNA-seq read alignments (BAM) must be assigned to genes and split into premature vs mature classes by intronic overlap, as in Nanodynamo's fractionsReadsClassification step.

## When NOT to use

- When premature calls would be interpreted in cytoplasmic or polysomal samples, where they are only interpretable in chromatin/nuclear fractions
- When reads cannot be unambiguously assigned because they overlap more than one gene (such reads are discarded rather than classified)
- For short-read RNA-seq where isoform-level intronic overlap logic of Nanodynamo's native-RNA classification is not designed to apply

## Inputs

- Chromatin-fraction Nanopore direct RNA-seq read alignments (BAM)
- Gene annotation in GTF format (matching genome build)
- Annotated intronic regions derived from the GTF

## Outputs

- Per-read classification table (read ID, gene assignment, mature/premature status)
- classified_reads.rds with one labelled record per retained read
- Premature/mature read proportions per gene/fraction (reproducing Figures 3C, S10, S17)

## How to apply

Load the chromatin-fraction Nanopore dRNA-seq alignments (BAM) and the matching gene annotation (GTF) into R and coerce both to GRanges. Use GenomicRanges findOverlaps to assign each read to the gene(s) it overlaps, discarding reads that overlap more than one gene to keep unambiguous gene assignments. For each gene-assigned read, compute overlap with annotated intronic intervals and flag reads whose intronic overlap is at least 10 bases as premature. When processing cytoplasmic or polysomal samples, remove premature calls since they are only interpretable in chromatin/nuclear fractions. Write the per-read classification table (read ID, gene assignment, mature/premature status) to classified_reads.rds for downstream Nanodynamo modelling.

## Related tools

- **R** (Host language for the GenomicRanges findOverlaps-based read classification (fractionsReadsClassification.R)) — https://github.com/mfurla/Nanodynamo
- **Nanodynamo** (Pipeline whose fractionsReadsClassification step consumes the premature/mature read classifications produced by these interval operations) — https://github.com/mfurla/Nanodynamo

## Examples

```
hits <- findOverlaps(query=granges(readGAlignments("chromatin_dRNA.bam")), subject=genes_from_GTF, ignore.strand=FALSE); keep <- countQueryHits(hits)==1
```

## Evaluation signals

- classified_reads.rds exists and contains one labelled record per retained read
- Reads overlapping more than one gene are absent from the output (discarded by design)
- Premature calls are restricted to ≥10 bases of annotated intronic overlap
- No premature reads remain in cytoplasmic or polysomal sample outputs
- Resulting premature/mature read proportions reproduce the fractions-reads-classification trends reported in Figures 3C, S10 and S17

## Limitations

- Reads overlapping more than one gene are discarded, so multi-isoform or overlapping-gene loci lose reads from the classification
- Premature/mature classification is only interpretable for chromatin/nuclear fractions; premature calls must be removed for cytoplasmic and polysomal samples
- A minimum intronic overlap of 10 bases is required, so short intronic overlaps are not counted as premature evidence

## Evidence

- [methods] Reads are assigned to genes by overlap; reads whose genes overlap annotated intronic regions are classified as premature and others as mature: [evidence span withheld — anchor status: not_found]
