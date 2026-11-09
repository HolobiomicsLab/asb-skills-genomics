---
name: split-flowcell-condition-read-count-quantification
description: Use when a live DRS run splits channels into RISER-deplete (even-numbered)
  and control conditions and you need per-transcript percent change in read/nt counts
  per biotype.
when_to_use_negative:
- The run was not a split-flow-cell design (e.g., the entire flow cell was targeted
  by RISER or no control channels exist), so no within-run control condition is available
  for percent-change computation
- Reads are from cDNA sequencing or non-ONT data rather than nanopore direct RNA sequencing
  of native RNA
- The question is about real-time rejection behavior or model classification accuracy
  rather than post-sequencing quantification of depletion/enrichment effects
edam_topics:
- http://edamontology.org/topic_0102
- http://edamontology.org/topic_3308
tools:
- name: Guppy (v4.0.14)
  role: Basecalls fast5 files from the live run using –flowcell FLO-MIN106 –kit SQK-RNA002
    before mapping
- name: minimap2 (v2.17)
  role: Maps basecalled reads to the GENCODE transcriptome (release 43, GRCh38.p13)
    with -ax map-ont --secondary=no
- name: samtools (v1.10)
  role: Filters alignments with option -F 2324 to remove reverse-strand, secondary/supplementary
    and unmapped reads
- name: RISER
  role: Supplies the decisions CSV that assigns each read to the deplete (even-numbered)
    or control channels
  repo: https://github.com/comprna/riser
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-024-48673-8
    title: Biochemical-free enrichment or depletion of RNA classes in real-time during
      direct RNA sequencing with RISER
  source_content_hash: sha256:096c3052d84c4abcdfdad9d01c1bfc8fca1facac8fc3b8847a74db45e932bbf1
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/comprna/riser
  tools:
  - Guppy (v4.0.14)
  - RISER
  - minimap2 (v2.17)
  - samtools (v1.10)
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0102
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# split-flowcell-condition-read-count-quantification

## Summary

Quantifies percent changes in read and nucleotide counts per transcript between the RISER-depleted and control channels of a split-flow-cell live MinION direct RNA sequencing run, to measure RISER-mediated mRNA/mtRNA depletion and lncRNA enrichment.

## When to use

Use when a live DRS run splits channels into RISER-deplete (even-numbered) and control conditions and you need per-transcript percent change in read/nt counts per biotype.

## When NOT to use

- The run was not a split-flow-cell design (e.g., the entire flow cell was targeted by RISER or no control channels exist), so no within-run control condition is available for percent-change computation
- Reads are from cDNA sequencing or non-ONT data rather than nanopore direct RNA sequencing of native RNA
- The question is about real-time rejection behavior or model classification accuracy rather than post-sequencing quantification of depletion/enrichment effects

## Inputs

- Live MinION Mk1B split-flow-cell direct RNA sequencing fast5 files (FLO-MIN106, SQK-RNA002)
- RISER decisions CSV assigning reads to even-numbered (deplete) vs remaining (control) channels
- GENCODE reference transcriptome release 43 (assembly GRCh38.p13)
- GENCODE reference annotation for lncRNA/protein-coding exon overlap filtering

## Outputs

- Per-transcript percent change in read counts and nucleotide counts between RISER and control conditions, per biotype
- Filtered per-transcript percent-change summary table (≥95% control read occupancy, ≥30 reads in both conditions, no protein-coding-overlapping lncRNAs)
- Aggregate mRNA, mtRNA and lncRNA read-count and nt-count depletion/enrichment percentages with paired Wilcoxon signed rank test p-values

## How to apply

Basecall the live-run reads with Guppy (v4.0.14, –flowcell FLO-MIN106 –kit SQK-RNA002) and map them to the GENCODE reference transcriptome (release 43, GRCh38.p13) with minimap2 (v2.17) using -ax map-ont --secondary=no, then retain only high-confidence alignments by removing reverse-strand, secondary/supplementary and unmapped reads with samtools (v1.10, -F 2324). Load the RISER decisions CSV and assign each read to the deplete condition (even-numbered channels) or the control condition (remaining channels). Compute percent change in read counts and nucleotide counts between RISER and control conditions per transcript per biotype, then restrict to transcripts occupying at least 95% of control-condition reads, exclude transcripts with fewer than 30 read counts in both conditions, and exclude lncRNAs whose exons overlap protein-coding exons in the reference annotation. Assemble the filtered per-transcript percent changes into a summary of aggregate mRNA, mtRNA and lncRNA read- and nt-count changes, and validate significance against randomly split channels of a non-RISER control run with a paired Wilcoxon signed rank test (Suppl. Table 8).

## Related tools

- **Guppy (v4.0.14)** (Basecalls fast5 files from the live run using –flowcell FLO-MIN106 –kit SQK-RNA002 before mapping)
- **minimap2 (v2.17)** (Maps basecalled reads to the GENCODE transcriptome (release 43, GRCh38.p13) with -ax map-ont --secondary=no)
- **samtools (v1.10)** (Filters alignments with option -F 2324 to remove reverse-strand, secondary/supplementary and unmapped reads)
- **RISER** (Supplies the decisions CSV that assigns each read to the deplete (even-numbered) or control channels) — https://github.com/comprna/riser

## Examples

```
minimap2 -ax map-ont --secondary=no gencode.v43.transcripts.fa reads.fa | samtools view -F 2324 -b > live_highconf.bam
```

## Evaluation signals

- Aggregate recomputed mRNA read-count depletion is ~86% (n = 873) and mtRNA ~85% (n = 14), exceeding 85% as reported
- Nucleotide-count reductions are ~89% for mRNA and ~90% for mtRNA
- lncRNA read- and nt-count changes show increases of ~47% and ~49%, respectively
- A paired Wilcoxon signed rank test against percent changes from randomly split channels of a non-RISER control run (Suppl. Table 8) confirms significance of the depletion
- All retained transcripts pass the filters: ≥95% of control-condition reads, ≥30 read counts in both conditions, and no lncRNA with exons overlapping protein-coding exons

## Limitations

- Depletion efficiency is bounded by the lag between RISER's reject command and hardware voltage reversal, which varies across platforms and is beyond RISER's control
- RNA secondary structures on the trans side of the pore can impede or fully inhibit reverse translocation by voltage reversal
- Accurate lncRNA identification from 3′ end signal remains challenging, and cell type-specific lncRNA expression makes models harder to generalize, so lncRNA enrichment estimates require overlap filtering

## Evidence

- [results] In the live HEK293 split-flow-cell MinION Mk1B run, RISER reduced read counts for transcripts in the mRNA and mtRNA target classes by an average of 86% (n = 873) and 85% (n = 14), respectively.: [evidence span withheld — anchor status: not_found]
- [methods] mapped to the GENCODE reference transcriptome (release 34, assembly GRCh38.p13) with minimap233 (v2.17) using options -ax map-ont --secondary=no: "mapped to the GENCODE reference transcriptome (release 34, assembly GRCh38.p13) with minimap233 (v2.17) using options -ax map-ont --secondary=no"
- [methods] unmapped reads were removed using samtools34 (v1.10) with option -F 2324: "unmapped reads were removed using samtools34 (v1.10) with option -F 2324"
- [abstract] By targeting the dominant messenger and mitochondrial RNA classes for depletion, RISER reduces their respective read counts by more than 85%: "By targeting the dominant messenger and mitochondrial RNA classes for depletion, RISER reduces their respective read counts by more than 85%"
- [abstract] resulting in an increase in sequencing depth of 47% on average for long non-coding RNAs: "resulting in an increase in sequencing depth of 47% on average for long non-coding RNAs"
