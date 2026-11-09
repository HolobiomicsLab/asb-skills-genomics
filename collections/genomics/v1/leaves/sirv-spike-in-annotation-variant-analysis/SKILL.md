---
name: sirv-spike-in-annotation-variant-analysis
description: Use when you have DRS reads aligned to a spike-in transcriptome (e.g.
  SIRV mixes E0/E2 with known concentrations) and need to decide which transcript
  quantification tool is most accurate under complete, insufficient or over annotation.
when_to_use_negative:
- Your goal is differential expression or isoform switching analysis in a complex
  human transcriptome rather than quantification accuracy benchmarking on spike-ins
  with known concentrations
- Your data are short-read RNA-seq rather than Oxford Nanopore direct RNA reads —
  the DRS-specific 3'-end and AS filtering (50nt / 95% threshold) is designed for
  DRS alignments
- There is no ground-truth concentration reference (spike-in mixes), so coefficient
  of variance and Spearman rho versus known abundance cannot be computed
edam_operation: http://edamontology.org/operation_3680
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: minimap2
  role: Aligns DRS reads to the SIRV transcriptome reference with -ax map-ont and
    retained secondary mappings (e.g. -N 10) to supply multi-mapping alignments for
    quantification
  repo: https://github.com/lh3/minimap2
- name: samtools
  role: Converts minimap2 SAM output to BAM for downstream quantifiers
- name: NanoCount
  role: Primary quantifier under test; applies DRS-specific filtering (3' end <=50nt,
    secondary AS >=95% of best) and expectation-maximisation for transcript abundance
  repo: https://github.com/a-slide/NanoCount
- name: Salmon
  role: Competing quantifier run with --noErrorModel on the same alignments
- name: StringTie2
  role: Competing quantifier run in default mode on the same alignments
- name: FLAIR
  role: Competing quantifier run in default mode on the same alignments
- name: LIQA
  role: Competing quantifier run in default mode on the same alignments
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/nar/gkab1129
    title: Accurate expression quantification from nanopore direct RNA sequencing
      with NanoCount
  source_content_hash: sha256:eabca0afa33cf19f473584490ea893a3d91d311f2ea6f8ec76acc02eb3e3a269
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/lh3/minimap2
  tools:
  - FLAIR
  - LIQA
  - NanoCount
  - Salmon
  - StringTie2
  - minimap2
  - samtools
  edam_operation: http://edamontology.org/operation_3680
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# sirv-spike-in-annotation-variant-analysis

## Summary

A benchmarking skill that compares DRS-compatible transcript quantification tools (NanoCount, Salmon --noErrorModel, StringTie2, FLAIR, LIQA) on SIRV spike-in direct RNA sequencing data (mixes E0/E2) under complete, insufficient and over annotation variants, scoring each tool by coefficient of variance (E0) and Spearman rho (E2).

## When to use

Use when you have DRS reads aligned to a spike-in transcriptome (e.g. SIRV mixes E0/E2 with known concentrations) and need to decide which transcript quantification tool is most accurate under complete, insufficient or over annotation.

## When NOT to use

- Your goal is differential expression or isoform switching analysis in a complex human transcriptome rather than quantification accuracy benchmarking on spike-ins with known concentrations
- Your data are short-read RNA-seq rather than Oxford Nanopore direct RNA reads — the DRS-specific 3'-end and AS filtering (50nt / 95% threshold) is designed for DRS alignments
- There is no ground-truth concentration reference (spike-in mixes), so coefficient of variance and Spearman rho versus known abundance cannot be computed

## Inputs

- SIRV spike-in direct RNA sequencing FASTQ/FASTA reads (e.g. ENA accession SRR6058583, mixes E0 and E2)
- SIRV transcriptome reference FASTA for minimap2 indexing/alignment
- SIRV annotation variants: complete (C), insufficient (I), over annotation (O)
- BAM alignments produced with minimap2 retaining secondary mappings
- Known SIRV isoform concentrations for E0 (equal) and E2 (varied)

## Outputs

- Per-tool, per-annotation-variant transcript abundance tables (TSV) from NanoCount, Salmon, StringTie2, FLAIR and LIQA
- Coefficient of variance across transcripts for mix E0 per tool/annotation variant
- Spearman's rank correlation rho between known and measured abundance for mix E2 per tool/annotation variant
- Benchmarking TSV table comparable to Supplementary Table S1

## How to apply

Align the SIRV E0/E2 direct-RNA reads to the SIRV transcriptome with minimap2 (-ax map-ont, retaining secondary mappings, e.g. -N 10) and convert to BAM with samtools, since multi-mapping read handling is central to isoform quantification. Run NanoCount v1.0.0 with its DRS-specific filtering (discard alignments with 3' ends >50nt from the transcript 3' end and secondary alignments with AS <95% of the best alignment) followed by expectation-maximisation; run Salmon with --noErrorModel, StringTie2, FLAIR and LIQA in default modes on the same alignments. For each quantifier and each SIRV annotation variant (complete C, insufficient I, over O), compute the coefficient of variance across transcripts for mix E0 (equal concentrations) and Spearman's rank correlation rho between known and measured abundance for mix E2 (varied concentrations). Assemble all metrics into a benchmarking table comparable to Supplementary Table S1 and write it as a TSV file. Expect NanoCount to show the least variability for E0 and the highest Spearman rho for E2 among the tested quantifiers.

## Related tools

- **minimap2** (Aligns DRS reads to the SIRV transcriptome reference with -ax map-ont and retained secondary mappings (e.g. -N 10) to supply multi-mapping alignments for quantification) — https://github.com/lh3/minimap2
- **samtools** (Converts minimap2 SAM output to BAM for downstream quantifiers)
- **NanoCount** (Primary quantifier under test; applies DRS-specific filtering (3' end <=50nt, secondary AS >=95% of best) and expectation-maximisation for transcript abundance) — https://github.com/a-slide/NanoCount
- **Salmon** (Competing quantifier run with --noErrorModel on the same alignments)
- **StringTie2** (Competing quantifier run in default mode on the same alignments)
- **FLAIR** (Competing quantifier run in default mode on the same alignments)
- **LIQA** (Competing quantifier run in default mode on the same alignments)

## Examples

```
minimap2 -ax map-ont -N 10 SIRV_transcriptome.fa SRR6058583_reads.fastq | samtools view -bh > sirv_aligned_reads.bam
```

## Evaluation signals

- Per-tool CV (mix E0) and Spearman rho (mix E2) values match the corresponding entries of Supplementary Table S1 (output_matches_reference / expert_review)
- NanoCount exhibits the least variability in measured isoform abundances for mix E0 (equal concentrations)
- NanoCount achieves the highest Spearman rho for mix E2 among NanoCount, Salmon, StringTie2, FLAIR and LIQA
- All five quantifiers are run on the same minimap2 BAM alignments, differing only in annotation variant (C, I, O)
- Coverage fraction computed as alignment length divided by original known transcript length for each assigned isoform

## Limitations

- Caution is needed extrapolating spike-in validation results due to the lower complexity of the spike-in transcriptome compared to the human one
- Many incomplete DRS reads are largely consigned to 3'UTRs and cannot specify the expressed isoform, affecting quantification accuracy
- Performance conclusions depend on the annotation variant used (complete, insufficient or over annotation); results vary across these scenarios

## Evidence

- [results] We tested the performance of NanoCount against other transcript quantification programs, including Salmon, StringTie2, FLAIR and LIQA: "We tested the performance of NanoCount against other transcript quantification programs, including Salmon, StringTie2, FLAIR and LIQA"
- [results] NanoCount 3'-end and AS filters: "discard read alignments with 3′ ends >50nt away from the transcript 3′ end"
- [results] Coverage fraction definition: "We defined coverage fraction as the observed transcript length (alignment length) divided by the original known transcript length."
- [results] SIRV benchmark data source: "using SIRV RNA spike-in DRS data (Materials and Methods)."
- [readme] BamSlam alignment command: [evidence span withheld — anchor status: external_doc]
