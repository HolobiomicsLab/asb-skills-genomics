---
name: noncoding-rna-exon-overlap-filtering
description: Use when computing per-transcript read/nt percent changes from a RISER
  read-until depletion run and the biotype-level lncRNA summary would otherwise include
  transcripts whose exons overlap protein-coding exons in the reference annotation.
when_to_use_negative:
- Reads are mapped to a genome or transcriptome without exon-level annotation, so
  lncRNA/protein-coding exon overlap cannot be assessed
- The analysis goal is not biotype-level enrichment/depletion quantification — e.g.
  real-time RISER classification or model retraining, where annotation-based lncRNA
  filtering is irrelevant
- Transcripts already have unambiguous, mutually exclusive biotype assignments (e.g.
  an explicit curated lncRNA list with no protein-coding exon overlap)
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: minimap2 (v2.17)
  role: Maps basecalled reads to the GENCODE reference transcriptome (-ax map-ont
    --secondary=no) so per-transcript counts and exon coordinates can be joined
- name: samtools (v1.10)
  role: Retains only high-confidence alignments (option -F 2324) that feed the per-transcript
    count computation prior to lncRNA overlap filtering
- name: RISER
  role: Supplies the decisions CSV assigning each read to the RISER-deplete (even-numbered
    channels) or control condition used to compute per-transcript percent changes
  repo: https://github.com/comprna/riser
- name: GENCODE reference transcriptome release 43 (GRCh38.p13)
  role: Reference annotation providing exon coordinates and transcript biotypes for
    identifying lncRNAs whose exons overlap protein-coding exons
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-024-48673-8
    title: Biochemical-free enrichment or depletion of RNA classes in real-time during
      direct RNA sequencing with RISER
  source_content_hash: sha256:94737b5c3f995429c6e1e1e3bfc1468f730bbe167f967a2383c2064ccdad35bb
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/comprna/riser
  tools:
  - GENCODE reference transcriptome release 43 (GRCh38.p13)
  - Guppy (v4.0.14)
  - RISER
  - minimap2 (v2.17)
  - samtools (v1.10)
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8461538461538461
n_claims: 13
---

# Noncoding RNA exon overlap filtering

## Summary

A refinement step in the RISER live-depletion analysis that removes lncRNA transcripts whose exons overlap protein-coding exons in the GENCODE (release 43, GRCh38.p13) reference annotation, so that percent-change summaries for the lncRNA class are not contaminated by ambiguously annotated protein-coding transcripts.

## When to use

Use when computing per-transcript read/nt percent changes from a RISER read-until depletion run and the biotype-level lncRNA summary would otherwise include transcripts whose exons overlap protein-coding exons in the reference annotation.

## When NOT to use

- Reads are mapped to a genome or transcriptome without exon-level annotation, so lncRNA/protein-coding exon overlap cannot be assessed
- The analysis goal is not biotype-level enrichment/depletion quantification — e.g. real-time RISER classification or model retraining, where annotation-based lncRNA filtering is irrelevant
- Transcripts already have unambiguous, mutually exclusive biotype assignments (e.g. an explicit curated lncRNA list with no protein-coding exon overlap)

## Inputs

- BAM alignments of basecalled (Guppy v4.0.14) direct RNA reads mapped to the GENCODE release 43 transcriptome (GRCh38.p13) and filtered with samtools -F 2324
- RISER decisions CSV mapping read IDs to even-numbered (deplete) vs remaining (control) channels
- GENCODE release 43 (GRCh38.p13) GTF/GFF reference annotation with exon coordinates and transcript_type labels
- Per-transcript read-count and nucleotide-count tables for RISER and control conditions

## Outputs

- Filtered per-transcript percent-change table (read counts and nt counts, RISER vs control) with overlapping-exon lncRNAs removed
- Aggregate summary of mRNA, mtRNA, and lncRNA read-count and nt-count percent changes

## How to apply

After mapping reads to the GENCODE transcriptome with minimap2 (-ax map-ont --secondary=no) and cleaning alignments with samtools (-F 2324), assign each read to the RISER-deplete (even-numbered channels) or control condition using the RISER decisions CSV. Compute percent changes in read and nucleotide counts per transcript per biotype, then restrict to transcripts occupying at least 95% of control-condition reads and having at least 30 read counts in both conditions. Next, intersect lncRNA exon coordinates with protein-coding exon coordinates from the reference annotation and drop any lncRNA with overlapping exons; the rationale is that such overlaps make class assignment ambiguous, biasing the lncRNA enrichment estimate. Assemble the remaining per-transcript percent changes into a summary table aggregating mRNA, mtRNA, and lncRNA read-count and nt-count changes.

## Related tools

- **minimap2 (v2.17)** (Maps basecalled reads to the GENCODE reference transcriptome (-ax map-ont --secondary=no) so per-transcript counts and exon coordinates can be joined)
- **samtools (v1.10)** (Retains only high-confidence alignments (option -F 2324) that feed the per-transcript count computation prior to lncRNA overlap filtering)
- **RISER** (Supplies the decisions CSV assigning each read to the RISER-deplete (even-numbered channels) or control condition used to compute per-transcript percent changes) — https://github.com/comprna/riser
- **GENCODE reference transcriptome release 43 (GRCh38.p13)** (Reference annotation providing exon coordinates and transcript biotypes for identifying lncRNAs whose exons overlap protein-coding exons)

## Evaluation signals

- Aggregate recomputed mRNA and mtRNA read-count depletion exceeds 85% (about 86% and 85%), with ~89% and 90% nt-count reductions, matching Suppl. Table 8
- lncRNA class shows a ~47% increase in read counts and ~49% increase in nt counts after exclusion of exon-overlapping lncRNAs
- Filtered transcript set retains only transcripts with ≥95% of control-condition reads and ≥30 read counts in both conditions
- Paired Wilcoxon signed-rank test against percent changes from randomly split channels of a non-RISER control run confirms significance

## Limitations

- Accurate lncRNA identification from the 3′ end signal remains challenging, and cell type-specific lncRNA expression makes predictive models harder to generalize to new samples
- Exon-overlap exclusion removes genuinely annotated lncRNAs that share exons with protein-coding genes, so the lncRNA summary covers only unambiguously non-overlapping transcripts
- Lag between RISER's reject command and hardware voltage reversal varies across platforms and is beyond RISER's control, affecting depletion magnitude

## Evidence

- [supplementary] Exclude transcripts with fewer than 30 read counts in both conditions, and exclude lncRNAs whose exons overlap protein-coding exons in the reference annotation.: [evidence span withheld — anchor status: not_found]
- [methods] minimap2 mapping to GENCODE transcriptome: "mapped to the GENCODE reference transcriptome (release 34, assembly GRCh38.p13) with minimap233 (v2.17) using options -ax map-ont --secondary=no"
- [supplementary] Read count and nt percent change computation: [evidence span withheld — anchor status: not_found]
- [readme] RISER live classification from raw signal: "RISER accurately classifies RNA molecules live during sequencing, directly from the first ~70-280nt-worth of raw nanopore signals with a convolutional neural network"
