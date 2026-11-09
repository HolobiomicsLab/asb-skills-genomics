---
name: live-run-depletion-efficiency-analysis
description: Use when quantifying depletion or enrichment efficiency of a live RISER
  run from basecalled, mapped reads split between deplete (even-numbered channels)
  and control channels, expressed as percent change in read and nt counts per RNA
  biotype.
when_to_use_negative:
- Input is a non-RISER control run with no decisions CSV — condition assignment must
  instead come from randomly split channels for the null comparison, not RISER channel
  labels
- Analysis targets globin mRNA depletion in whole blood, which is validated against
  a different benchmark (>90% globin decrease, ~16% non-globin increase) rather than
  the mRNA/mtRNA HEK293 metrics
- Goal is RISER model training/testing or real-time signal classification, not post-sequencing
  quantification of depletion efficiency
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: Guppy (v4.0.14)
  role: Basecalls the live-run FAST5 files using options --flowcell FLO-MIN106 --kit
    SQK-RNA002 before mapping
- name: minimap2 (v2.17)
  role: Maps basecalled reads to the GENCODE transcriptome (GRCh38.p13) with '-ax
    map-ont --secondary=no' to obtain per-transcript assignments
- name: samtools (v1.10)
  role: Removes reverse-strand, secondary/supplementary and unmapped alignments with
    '-F 2324' to keep only high-confidence mappings
- name: RISER
  role: Produces the decisions CSV that assigns reads to deplete (even-numbered channels)
    versus control conditions based on live read-until rejections
  repo: https://github.com/comprna/riser
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-024-48673-8
    title: Biochemical-free enrichment or depletion of RNA classes in real-time during
      direct RNA sequencing with RISER
  source_content_hash: sha256:b3db08c3f11944ad9691edf12075ed5a5753b586ee43709c699a81eb66a6ce1c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/comprna/riser
  tools:
  - Guppy (v4.0.14)
  - RISER
  - minimap2 (v2.17)
  - samtools (v1.10)
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 6
---

# live-run-depletion-efficiency-analysis

## Summary

Quantifies how much live RISER read-until depletion of mRNA and mtRNA changes per-transcript read and nucleotide counts in a split-flow-cell MinION direct RNA sequencing run, and validates the aggregate depletion against reported benchmarks (~86%/85% read depletion).

## When to use

Use when quantifying depletion or enrichment efficiency of a live RISER run from basecalled, mapped reads split between deplete (even-numbered channels) and control channels, expressed as percent change in read and nt counts per RNA biotype.

## When NOT to use

- Input is a non-RISER control run with no decisions CSV — condition assignment must instead come from randomly split channels for the null comparison, not RISER channel labels
- Analysis targets globin mRNA depletion in whole blood, which is validated against a different benchmark (>90% globin decrease, ~16% non-globin increase) rather than the mRNA/mtRNA HEK293 metrics
- Goal is RISER model training/testing or real-time signal classification, not post-sequencing quantification of depletion efficiency

## Inputs

- Live-run FAST5 signal files from a RISER split-flow-cell MinION Mk1B direct RNA sequencing experiment (FLO-MIN106, SQK-RNA002)
- RISER decisions CSV mapping reads to channels (even-numbered = deplete, remaining = control)
- GENCODE reference transcriptome (release 43, assembly GRCh38.p13) and matching GTF for biotype and exon-overlap annotation

## Outputs

- Per-transcript percent change in read counts and nucleotide counts between RISER-deplete and control conditions, by biotype
- Aggregate summary table of mRNA, mtRNA, and lncRNA read-count and nt-count percent changes
- Wilcoxon signed-rank test statistics comparing RISER percent changes to random channel splits of a non-RISER control run

## How to apply

Basecall the live-run FAST5 files with Guppy (v4.0.14, --flowcell FLO-MIN106 --kit SQK-RNA002) and map to the GENCODE transcriptome (release 43, GRCh38.p13) with minimap2 v2.17 using '-ax map-ont --secondary=no', discarding reverse-strand, secondary/supplementary and unmapped alignments via samtools v1.10 with '-F 2324'. Load the RISER decisions CSV and assign each read to the deplete condition (even-numbered channels) or the control condition (remaining channels). Compute per-transcript percent changes in read counts and nucleotide counts for each biotype, then filter: keep only transcripts occupying >=95% of control-condition reads, exclude transcripts with fewer than 30 read counts in both conditions, and drop lncRNAs whose exons overlap protein-coding exons in the reference annotation. Aggregate the filtered percent changes into a summary table and validate against the expected magnitudes (mRNA ~86%, mtRNA ~85% read depletion; ~89%/90% nt reductions; ~47%/49% lncRNA read/nt increase), confirming significance with a paired Wilcoxon signed-rank test against percent changes from randomly split channels of a non-RISER control run.

## Related tools

- **Guppy (v4.0.14)** (Basecalls the live-run FAST5 files using options --flowcell FLO-MIN106 --kit SQK-RNA002 before mapping)
- **minimap2 (v2.17)** (Maps basecalled reads to the GENCODE transcriptome (GRCh38.p13) with '-ax map-ont --secondary=no' to obtain per-transcript assignments)
- **samtools (v1.10)** (Removes reverse-strand, secondary/supplementary and unmapped alignments with '-F 2324' to keep only high-confidence mappings)
- **RISER** (Produces the decisions CSV that assigns reads to deplete (even-numbered channels) versus control conditions based on live read-until rejections) — https://github.com/comprna/riser

## Examples

```
minimap2 -ax map-ont --secondary=no GENCODE_GRCh38.p13_transcriptome.fa guppy_basecalled_reads.fastq | samtools view -F 2324 -b > riser_live_mapped.bam
```

## Evaluation signals

- Recomputed aggregate mRNA read-count depletion averages ~86% (n = 873 transcripts) and mtRNA ~85% (n = 14), i.e. more than 85% for both target classes
- Nucleotide-count reductions for mRNA and mtRNA are approximately 89% and 90%
- lncRNA read and nt counts increase by approximately 47% and 49% on average in the filtered summary table
- Samtools '-F 2324' filtering leaves only forward-strand, primary, mapped alignments; no secondary/supplementary records remain in the BAM
- Paired Wilcoxon signed-rank test against random channel splits of a non-RISER control run is significant (Suppl. Table 8), confirming the depletion is not explained by channel stochasticity

## Limitations

- The lag between the read-until API receiving a reject command and the voltage reversal, and the reversal efficiency itself, are beyond RISER's control and vary across platforms, affecting achievable depletion levels
- RNA secondary structures on the trans side of the pore can impede or completely inhibit reverse translocation by voltage reversal
- Accurate lncRNA identification from the 3' end signal remains challenging, so lncRNA enrichment estimates depend on filtering out lncRNAs whose exons overlap protein-coding exons
- Percent changes are only computed for transcripts with >=30 read counts in both conditions and >=95% control-read occupancy, restricting inference to well-covered transcripts

## Evidence

- [abstract] By targeting the dominant messenger and mitochondrial RNA classes for depletion, RISER reduces their respective read counts by more than 85%: "By targeting the dominant messenger and mitochondrial RNA classes for depletion, RISER reduces their respective read counts by more than 85%"
- [methods] Fast5 files were basecalled with Guppy (v4.0.14) using options –flowcell FLO-MIN106 –kit SQK-RNA002: "Fast5 files were basecalled with Guppy (v4.0.14) using options –flowcell FLO-MIN106 –kit SQK-RNA002"
- [methods] mapped to the GENCODE reference transcriptome (release 34, assembly GRCh38.p13) with minimap2 (v2.17) using options -ax map-ont --secondary=no: "mapped to the GENCODE reference transcriptome (release 34, assembly GRCh38.p13) with minimap233 (v2.17) using options -ax map-ont --secondary=no"
- [methods] unmapped reads were removed using samtools (v1.10) with option -F 2324: "unmapped reads were removed using samtools34 (v1.10) with option -F 2324"
- [discussion] We illustrated RISER’s usability during live runs by targeting the highly abundant mRNA and mtRNA classes for depletion: "We illustrated RISER’s usability during live runs by targeting the highly abundant mRNA and mtRNA classes for depletion"
- [discussion] the lag between the read-until API receiving a reject command from RISER and requesting a voltage reversal in the hardware: "the lag between the read-until API receiving a reject command from RISER and requesting a voltage reversal in the hardware"
- [readme] RISER accurately classifies RNA molecules live during sequencing, directly from the first ~70-280nt-worth of raw nanopore signals: "RISER accurately classifies RNA molecules live during sequencing, directly from the first ~70-280nt-worth of raw nanopore signals"
