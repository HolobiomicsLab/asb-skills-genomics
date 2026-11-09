---
name: nanopore-read-alignment-and-biotype-assignment
description: Use when you have nanopore DRS reads from a RISER live run (even-numbered
  channels = deplete, others = control) and need per-transcript or per-biotype read-count
  and nucleotide-count changes to evaluate depletion/enrichment efficiency.
when_to_use_negative:
- Sequencing reads were produced by basecalling-plus-mapping adaptive sampling rather
  than RISER raw-signal classification
- You need live reject decisions during the run — this skill is post-sequencing quantification,
  not real-time depletion
- Transcripts of interest have fewer than 30 read counts in both conditions, where
  percent-change estimates are excluded as unreliable
edam_operation: http://edamontology.org/operation_3198
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3170
tools:
- name: Guppy (v4.0.14)
  role: Basecalls fast5 signals with --flowcell FLO-MIN106 --kit SQK-RNA002
- name: minimap2 (v2.17)
  role: Splice-aware ONT transcriptome alignment (-ax map-ont --secondary=no) against
    GENCODE
- name: samtools (v1.10)
  role: Filters low-confidence alignments using option -F 2324
- name: RISER
  role: Provides the decisions CSV that defines even-numbered (deplete) vs remaining
    (control) channels
  repo: https://github.com/comprna/riser
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-024-48673-8
    title: Biochemical-free enrichment or depletion of RNA classes in real-time during
      direct RNA sequencing with RISER
  source_content_hash: sha256:02329dc2eb956240f7a06d0711d9a2b5f27dcf2fcd06f90943cba0f556492c77
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/comprna/riser
  tools:
  - Guppy (v4.0.14)
  - RISER
  - minimap2 (v2.17)
  - samtools (v1.10)
  edam_operation: http://edamontology.org/operation_3198
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# nanopore-read-alignment-and-biotype-assignment

## Summary

Basecalls nanopore direct RNA reads, maps them to a GENCODE transcriptome, and assigns each read to a biotype so per-biotype read/nucleotide count changes between RISER-depleted and control channels can be quantified.

## When to use

Use when you have nanopore DRS reads from a RISER live run (even-numbered channels = deplete, others = control) and need per-transcript or per-biotype read-count and nucleotide-count changes to evaluate depletion/enrichment efficiency.

## When NOT to use

- Sequencing reads were produced by basecalling-plus-mapping adaptive sampling rather than RISER raw-signal classification
- You need live reject decisions during the run — this skill is post-sequencing quantification, not real-time depletion
- Transcripts of interest have fewer than 30 read counts in both conditions, where percent-change estimates are excluded as unreliable

## Inputs

- fast5 raw signal files from a MinION Mk1B direct RNA sequencing run
- GENCODE reference transcriptome (release 43, assembly GRCh38.p13)
- RISER decisions CSV mapping read/channel to deplete or control condition

## Outputs

- BAM of high-confidence transcript alignments (samtools -F 2324 filtered)
- Per-transcript percent change in read counts and nucleotide counts by biotype
- Summary table of aggregate mRNA, mtRNA, and lncRNA read-count and nt-count changes

## How to apply

Basecall fast5 files with Guppy (v4.0.14, --flowcell FLO-MIN106 --kit SQK-RNA002) and map to the GENCODE reference transcriptome (release 43, GRCh38.p13) with minimap2 (v2.17) using -ax map-ont --secondary=no. Retain only high-confidence alignments by removing reverse-strand, secondary, supplementary, and unmapped reads with samtools (v1.10) -F 2324. Load the RISER decisions CSV and assign reads to the deplete condition (even-numbered channels) or control condition (remaining channels), then compute percent changes in read counts and nucleotide counts per transcript per biotype. Restrict to transcripts occupying at least 95% of control-condition reads, drop transcripts with fewer than 30 read counts in both conditions, and exclude lncRNAs whose exons overlap protein-coding exons in the reference annotation to avoid misclassification.

## Related tools

- **Guppy (v4.0.14)** (Basecalls fast5 signals with --flowcell FLO-MIN106 --kit SQK-RNA002)
- **minimap2 (v2.17)** (Splice-aware ONT transcriptome alignment (-ax map-ont --secondary=no) against GENCODE)
- **samtools (v1.10)** (Filters low-confidence alignments using option -F 2324)
- **RISER** (Provides the decisions CSV that defines even-numbered (deplete) vs remaining (control) channels) — https://github.com/comprna/riser

## Examples

```
minimap2 -ax map-ont --secondary=no GENCODE.v43.transcripts.fa reads.fastq | samtools view -b -F 2324 > aligned.filtered.bam
```

## Evaluation signals

- Aggregate mRNA and mtRNA read-count depletion should exceed 85% (~86% and ~85%), with ~89% and ~90% nucleotide-count reductions
- lncRNA read/nt counts should increase ~47%/~49% when mRNA/mtRNA are depleted
- Significance should be confirmed by a paired Wilcoxon signed rank test against percent changes computed from randomly split channels of a non-RISER control run (Suppl. Table 8)
- All retained alignments should be forward-strand, primary, and mapped (samtools flag filter -F 2324 removes 2324-flagged reads)
- Retained transcripts should occupy at least 95% of control-condition reads after filtering

## Limitations

- Accurate lncRNA identification from 3'-end signal remains challenging; lncRNAs whose exons overlap protein-coding exons must be excluded from the reference annotation
- Lag between the read-until API receiving a reject command and voltage reversal, and voltage-reversal efficiency, vary across platforms and limit achievable depletion
- Transcripts with fewer than 30 read counts in both conditions are excluded, limiting power for lowly expressed RNAs

## Evidence

- [methods] Basecall and map live-run reads: "Fast5 files were basecalled with Guppy (v4.0.14) using options –flowcell FLO-MIN106 –kit SQK-RNA002"
- [methods] Alignment parameters: "mapped to the GENCODE reference transcriptome (release 34, assembly GRCh38.p13) with minimap233 (v2.17) using options -ax map-ont --secondary=no"
- [methods] High-confidence filtering: "unmapped reads were removed using samtools34 (v1.10) with option -F 2324"
- [abstract] Depletion outcome: "By targeting the dominant messenger and mitochondrial RNA classes for depletion, RISER reduces their respective read counts by more than 85%"
- [readme] RISER channel assignment basis: "RISER then either allows the molecule to complete sequencing or sends a reject command to the sequencing platform via Oxford Nanopore Technologies' ReadUntil API"
