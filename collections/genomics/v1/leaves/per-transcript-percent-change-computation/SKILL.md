---
name: per-transcript-percent-change-computation
description: Use when you have a split-flow-cell live RISER run (deplete vs control
  channels) with basecalled, mapped reads and a RISER decisions CSV, and you need
  per-transcript read/nt count percent changes per biotype.
when_to_use_negative:
- Data comes from a non-RISER run with randomly split channels rather than a live
  RISER deplete/control split — such splits serve as the negative-control background
  in the Wilcoxon test, not the effect being quantified.
- Transcripts fail the filters (not in the top 95% of control-condition reads, or
  fewer than 30 read counts in both conditions), or are lncRNAs overlapping protein-coding
  exons — their percent changes are excluded from the summary.
edam_operation: http://edamontology.org/operation_3800
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0080
tools:
- name: Guppy (v4.0.14)
  role: Basecalls fast5 files from the live run using --flowcell FLO-MIN106 --kit
    SQK-RNA002 before mapping
- name: minimap2 (v2.17)
  role: Maps basecalled reads to the GENCODE transcriptome with -ax map-ont --secondary=no
    to assign reads to transcripts
- name: samtools (v1.10)
  role: Filters alignments with -F 2324 to keep only high-confidence mappings
- name: RISER
  role: Produces the decisions CSV that assigns reads to deplete (even-numbered channels)
    or control conditions in the live run
  repo: https://github.com/comprna/riser
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-024-48673-8
    title: Biochemical-free enrichment or depletion of RNA classes in real-time during
      direct RNA sequencing with RISER
  source_content_hash: sha256:bcb8520021c7e4b5c6c6257dbee28f72eeeddf7bab5241c700ed514499d9e82a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/comprna/riser
  tools:
  - Guppy (v4.0.14)
  - RISER
  - minimap2 (v2.17)
  - samtools (v1.10)
  edam_operation: http://edamontology.org/operation_3800
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# Per-transcript percent change computation

## Summary

Computes per-transcript percent changes in read counts and nucleotide (nt) counts between RISER-deplete (even-numbered channels) and control (remaining channels) conditions from a live HEK293 MinION direct RNA sequencing run, to quantify RISER-mediated mRNA/mtRNA depletion and lncRNA enrichment.

## When to use

Use when you have a split-flow-cell live RISER run (deplete vs control channels) with basecalled, mapped reads and a RISER decisions CSV, and you need per-transcript read/nt count percent changes per biotype.

## When NOT to use

- Data comes from a non-RISER run with randomly split channels rather than a live RISER deplete/control split — such splits serve as the negative-control background in the Wilcoxon test, not the effect being quantified.
- Transcripts fail the filters (not in the top 95% of control-condition reads, or fewer than 30 read counts in both conditions), or are lncRNAs overlapping protein-coding exons — their percent changes are excluded from the summary.

## Inputs

- Basecalled reads from the live split-flow-cell MinION Mk1B run (fast5 basecalled with Guppy v4.0.14)
- BAM alignments to GENCODE release 43 transcriptome (GRCh38.p13) filtered with samtools -F 2324
- RISER decisions CSV mapping reads to channel IDs for deplete/control assignment
- GENCODE reference annotation (GTF) with biotype labels and exon coordinates

## Outputs

- Per-transcript percent change table for read counts and nt counts by biotype
- Summary table reporting aggregate mRNA, mtRNA, and lncRNA read-count and nt-count percent changes

## How to apply

Basecall fast5 files with Guppy (v4.0.14, --flowcell FLO-MIN106 --kit SQK-RNA002), map to the GENCODE reference transcriptome (release 43, GRCh38.p13) with minimap2 (v2.17) using -ax map-ont --secondary=no, and retain only high-confidence alignments with samtools (v1.10) using -F 2324 (removing reverse-strand, secondary/supplementary, and unmapped reads). Load the RISER decisions CSV and assign each read to the deplete condition (even-numbered channels) or control condition (remaining channels). For each transcript in each biotype, compute the percent change in read counts and nt counts between RISER and control conditions. Restrict to transcripts occupying at least 95% of control-condition reads, exclude transcripts with fewer than 30 read counts in both conditions, and exclude lncRNAs whose exons overlap protein-coding exons in the reference annotation; then aggregate the filtered per-transcript percent changes into a summary table.

## Related tools

- **Guppy (v4.0.14)** (Basecalls fast5 files from the live run using --flowcell FLO-MIN106 --kit SQK-RNA002 before mapping)
- **minimap2 (v2.17)** (Maps basecalled reads to the GENCODE transcriptome with -ax map-ont --secondary=no to assign reads to transcripts)
- **samtools (v1.10)** (Filters alignments with -F 2324 to keep only high-confidence mappings)
- **RISER** (Produces the decisions CSV that assigns reads to deplete (even-numbered channels) or control conditions in the live run) — https://github.com/comprna/riser

## Examples

```
minimap2 -ax map-ont --secondary=no gencode.v43.transcripts.fa reads.fq | samtools view -b -F 2324 - > mapped.bam
```

## Evaluation signals

- Aggregate mRNA and mtRNA read-count reductions should be about 86% (n = 873) and 85% (n = 14), with nt-count reductions of about 89% and 90%.
- lncRNA read and nt counts should increase by about 47% and 49% respectively after filtering.
- A paired Wilcoxon signed rank test against percent changes from randomly split channels of a non-RISER control run (Suppl. Table 8) should confirm significance of the depletion.
- Every transcript in the summary must satisfy the filters: top 95% of control-condition reads, ≥30 read counts in both conditions, and no lncRNA–protein-coding exon overlap.

## Limitations

- Lag between the read-until API receiving a reject command and voltage reversal, and voltage-reversal efficiency, vary across platforms and are beyond RISER's control, affecting realized depletion per channel.
- Accurate lncRNA identification from the 3′ end signal remains challenging; cell type-specific lncRNA expression makes models harder to generalize, so lncRNA percent changes carry extra uncertainty.
- Larger training datasets are needed for lowly expressed RNA classes, so sensitivity to rare RNAs is limited.

## Evidence

- [methods] Guppy basecalling and minimap2/samtools filtering: "mapped to the GENCODE reference transcriptome (release 34, assembly GRCh38.p13) with minimap233 (v2.17) using options -ax map-ont --secondary=no"
- [methods] High-confidence alignment filtering: "unmapped reads were removed using samtools34 (v1.10) with option -F 2324"
- [abstract] Reported depletion magnitudes used to validate the computed percent changes: "By targeting the dominant messenger and mitochondrial RNA classes for depletion, RISER reduces their respective read counts by more than 85%"
- [abstract] Expected lncRNA increase used as an evaluation signal: "resulting in an increase in sequencing depth of 47% on average for long non-coding RNAs"
