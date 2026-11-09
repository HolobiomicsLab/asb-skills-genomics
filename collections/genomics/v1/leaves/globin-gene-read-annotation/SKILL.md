---
name: globin-gene-read-annotation
description: Use when whole-blood nanopore DRS reads must be labeled by gene/biotype
  to benchmark a globin classifier — e.g., building the reserved 84,831-signal test
  set with HBA1/HBA2/HBB positives against non-globin negatives.
when_to_use_negative:
- Input reads are not human whole-blood DRS data or cannot be mapped against GENCODE
  GRCh38.p13 — the HBA1/HBA2/HBB positive-class definition is specific to human globin
  transcripts
- The goal is real-time in-pore rejection via the ReadUntil API rather than post-hoc
  labeling of a reserved read set for accuracy evaluation
- Reads have not been trimmed of sequencing adapter and poly(A) tail with BoostNano,
  so signal-level evaluation would be confounded rather than isolating classification
  performance
edam_operation: http://edamontology.org/operation_0362
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0102
tools:
- name: Guppy (v4.0.14)
  role: Basecalls FAST5 DRS files with --flowcell FLO-MIN106 --kit SQK-RNA002 to obtain
    sequences for mapping
- name: minimap2 (v2.17)
  role: Maps basecalled reads to the GENCODE release 34 (GRCh38.p13) transcriptome
    with '-ax map-ont --secondary=no' to derive gene/biotype labels
- name: samtools (v1.10)
  role: Filters mappings with option -F 2324 to remove reverse-strand, secondary/supplementary,
    and unmapped reads, retaining only high-confidence assignments
- name: BoostNano
  role: Trims sequencing adapter and poly(A) tail from raw signals so annotation-based
    evaluation isolates model performance
  repo: https://github.com/haotianteng/BoostNano
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-024-48673-8
    title: Biochemical-free enrichment or depletion of RNA classes in real-time during
      direct RNA sequencing with RISER
  source_content_hash: sha256:5625d5456fd3b5fcdc1a65c97bf004152a193d6b23138e41241452e7d619edc6
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/haotianteng/BoostNano
  tools:
  - BoostNano
  - Guppy
  - Guppy (v4.0.14)
  - PyTorch
  - RISER
  - minimap2
  - minimap2 (v2.17)
  - samtools
  - samtools (v1.10)
  edam_operation: http://edamontology.org/operation_0362
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9285714285714286
n_claims: 14
---

# globin-gene-read-annotation

## Summary

Assigns ground-truth gene and biotype labels to whole-blood direct RNA sequencing (DRS) reads by basecalling with Guppy, mapping to the GENCODE transcriptome (release 34, GRCh38.p13) with minimap2, and filtering with samtools, so reads from HBA1, HBA2, and HBB form the globin positive class and all other reads (mtRNA, lncRNA, other biotypes) form the negative class for evaluating the RISER globin model.

## When to use

Use when whole-blood nanopore DRS reads must be labeled by gene/biotype to benchmark a globin classifier — e.g., building the reserved 84,831-signal test set with HBA1/HBA2/HBB positives against non-globin negatives.

## When NOT to use

- Input reads are not human whole-blood DRS data or cannot be mapped against GENCODE GRCh38.p13 — the HBA1/HBA2/HBB positive-class definition is specific to human globin transcripts
- The goal is real-time in-pore rejection via the ReadUntil API rather than post-hoc labeling of a reserved read set for accuracy evaluation
- Reads have not been trimmed of sequencing adapter and poly(A) tail with BoostNano, so signal-level evaluation would be confounded rather than isolating classification performance

## Inputs

- Raw nanopore DRS FAST5 files from whole-blood RNA002 (FLO-MIN106) sequencing
- GENCODE reference transcriptome release 34 (assembly GRCh38.p13)
- RISER target-class probability outputs per read (threshold T = 0.9)

## Outputs

- BAM of high-confidence mapped reads (reverse-strand, secondary, supplementary, and unmapped reads removed)
- Per-read gene and biotype labels (globin vs non-globin ground truth)
- globin_biotype_accuracy.csv with per-biotype accuracy for HBA1, HBA2, HBB, mtRNA, lncRNA, and other

## How to apply

Basecall FAST5 files with Guppy (v4.0.14, --flowcell FLO-MIN106 --kit SQK-RNA002) and map reads to the GENCODE release 34 reference transcriptome (GRCh38.p13) with minimap2 (v2.17) using '-ax map-ont --secondary=no'. Retain only high-confidence mappings by removing reverse-strand, secondary, supplementary, and unmapped alignments with samtools (v1.10, option -F 2324). Assign each read its gene and biotype from the GENCODE-based mappings: reads from globin genes HBA1, HBA2, and HBB constitute the positive class, while non-globin reads (including mtRNA and lncRNA biotypes, balanced one read at a time per gene) form the negative class. Tabulate per-transcript and per-biotype proportions of correct, incorrect, and undecided calls (classification threshold T = 0.9 on RISER target-class probabilities) and write per-biotype accuracies for HBA1, HBA2, HBB, mtRNA, lncRNA, and aggregated 'other' to globin_biotype_accuracy.csv.

## Related tools

- **Guppy (v4.0.14)** (Basecalls FAST5 DRS files with --flowcell FLO-MIN106 --kit SQK-RNA002 to obtain sequences for mapping)
- **minimap2 (v2.17)** (Maps basecalled reads to the GENCODE release 34 (GRCh38.p13) transcriptome with '-ax map-ont --secondary=no' to derive gene/biotype labels)
- **samtools (v1.10)** (Filters mappings with option -F 2324 to remove reverse-strand, secondary/supplementary, and unmapped reads, retaining only high-confidence assignments)
- **BoostNano** (Trims sequencing adapter and poly(A) tail from raw signals so annotation-based evaluation isolates model performance) — https://github.com/haotianteng/BoostNano

## Examples

```
minimap2 -ax map-ont --secondary=no GRCh38.p13.fa reads.fastq | samtools view -F 2324 -b -o globin_test_mapped.bam
```

## Evaluation signals

- globin_biotype_accuracy.csv contains a per-biotype accuracy table computed over the 84,831-signal reserved whole-blood test set
- Globin-gene accuracies (HBA1, HBA2, HBB) are consistent with the paper's >90% correct globin classification/depletion performance
- All non-globin biotypes (mtRNA, lncRNA, other) are classified as non-globin with >99% accuracy
- Mapping output contains no reverse-strand, secondary, supplementary, or unmapped alignments (samtools -F 2324 filter applied)
- Every read carries a gene and biotype label traceable to GENCODE release 34 GRCh38.p13 mappings, with negatives balanced one read at a time per gene

## Limitations

- Bioinformatic annotation and discarding of globin reads post-sequencing is reported as insufficient to counteract the problem of limited coverage of non-globin RNAs, unlike real-time RISER depletion
- Accurate lncRNA identification from the 3′ end signal remains challenging, and cell type-specific lncRNA expression makes model generalization to new samples harder
- Undecided reads (no confident decision at threshold T = 0.9) must be tracked separately from correct/incorrect classifications

## Evidence

- [methods] Fast5 files were basecalled with Guppy (v4.0.14) using options –flowcell FLO-MIN106 –kit SQK-RNA002: "Fast5 files were basecalled with Guppy (v4.0.14) using options –flowcell FLO-MIN106 –kit SQK-RNA002"
- [methods] minimap2 mapping to GENCODE: "mapped to the GENCODE reference transcriptome (release 34, assembly GRCh38.p13) with minimap233 (v2.17) using options -ax map-ont --secondary=no"
- [methods] samtools high-confidence filtering: "unmapped reads were removed using samtools34 (v1.10) with option -F 2324"
- [discussion] globin mRNA depletion application in blood: "we applied it to the depletion of globin mRNA from blood samples"
- [readme] README globin model support: "RISER provides models to target the following RNA classes, which are typically highly abundant in various cell lines"
