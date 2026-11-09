---
name: transcript-abundance-quantification-benchmarking
description: Use when DRS (direct RNA) reads aligned to a transcriptome must be assigned
  to quantification tools, and ground-truth spike-in abundances (SIRV mixes E0/E2)
  are available to judge which quantifier is most accurate.
when_to_use_negative:
- Input reads are short-read RNA-seq rather than nanopore direct RNA sequencing —
  Salmon's error model should not be disabled and DRS-specific 3'-end filtering does
  not apply
- No ground-truth abundance data (e.g. spike-ins with known concentrations) exist,
  so quantification accuracy cannot be evaluated against expected values
- The goal is gene-level rather than transcript/isoform-level abundance estimation,
  which these isoform-resolving quantifiers are not designed to benchmark
edam_operation: http://edamontology.org/operation_3800
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0622
tools:
- name: minimap2
  role: Aligns SIRV direct-RNA reads to the transcriptome with -ax map-ont -N 10 to
    retain secondary mappings needed for EM assignment
  repo: https://github.com/lh3/minimap2
- name: samtools
  role: Converts minimap2 SAM output to BAM for the quantifiers
- name: NanoCount
  role: DRS-specific quantifier under test; applies 3'-end <=50nt and secondary AS
    >=95% filtering before EM
  repo: https://github.com/a-slide/NanoCount
- name: Salmon
  role: Competing quantifier run with --noErrorModel on the same alignments
- name: StringTie2
  role: Competing quantifier run in default mode
- name: FLAIR
  role: Competing long-read quantifier run in default mode
- name: LIQA
  role: Competing DRS-aware quantifier run in default mode
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/nar/gkab1129
    title: Accurate expression quantification from nanopore direct RNA sequencing
      with NanoCount
  source_content_hash: sha256:bed74c86c80e88c4db420f84658a77e33e81ea881284987e0844bc27660f6581
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
  edam_operation: http://edamontology.org/operation_3800
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8461538461538461
n_claims: 13
---

# transcript-abundance-quantification-benchmarking

## Summary

Benchmark transcript quantification tools for nanopore direct RNA sequencing (DRS) on SIRV spike-in data with known isoform concentrations, comparing NanoCount against Salmon, StringTie2, FLAIR and LIQA using coefficient of variance (equal-mix E0) and Spearman rho (varied-mix E2).

## When to use

Use when DRS (direct RNA) reads aligned to a transcriptome must be assigned to quantification tools, and ground-truth spike-in abundances (SIRV mixes E0/E2) are available to judge which quantifier is most accurate.

## When NOT to use

- Input reads are short-read RNA-seq rather than nanopore direct RNA sequencing — Salmon's error model should not be disabled and DRS-specific 3'-end filtering does not apply
- No ground-truth abundance data (e.g. spike-ins with known concentrations) exist, so quantification accuracy cannot be evaluated against expected values
- The goal is gene-level rather than transcript/isoform-level abundance estimation, which these isoform-resolving quantifiers are not designed to benchmark

## Inputs

- SIRV spike-in direct RNA sequencing FASTQ reads (e.g. ENA accession SRR6058583 for mixes E0 and E2)
- SIRV transcriptome reference FASTA
- SIRV annotation variants: complete (C), insufficient (I) and over-complete (O) GTF/annotation sets
- Known SIRV isoform concentrations for mix E0 (equal) and mix E2 (varied)

## Outputs

- Per-tool, per-annotation-variant transcript abundance estimates (TSV/quant files from NanoCount, Salmon, StringTie2, FLAIR, LIQA)
- Benchmarking table (TSV) of coefficient of variance for mix E0 and Spearman rho for mix E2 per quantifier, comparable to Supplementary Table S1

## How to apply

Align the SIRV E0/E2 direct-RNA reads (e.g. SRR6058583) to the SIRV transcriptome reference with minimap2 using -ax map-ont and -N 10 so at least 10 secondary mappings are retained, converting SAM to BAM with samtools; secondary alignments are essential because EM-based quantifiers resolve multi-mapping reads. Run NanoCount v1.0.0 with its default DRS-specific filtering (discard alignments with 3' ends >50 nt from the transcript 3' end and secondary alignments with AS <95% of the best AS) and its expectation-maximization algorithm. Run the competitors on the same BAM: Salmon with --noErrorModel, StringTie2 and FLAIR in default modes, and LIQA in default mode. For each tool and each SIRV annotation variant (complete C, insufficient I, over-annotation O), compute the coefficient of variance across transcripts for mix E0 (equal concentrations, so CV should be low) and Spearman's rank correlation rho between known and measured abundance for mix E2 (varied concentrations). Assemble the per-tool metrics into a TSV table comparable to Supplementary Table S1, where NanoCount is expected to show the least variability for E0 and the highest Spearman rho for E2.

## Related tools

- **minimap2** (Aligns SIRV direct-RNA reads to the transcriptome with -ax map-ont -N 10 to retain secondary mappings needed for EM assignment) — https://github.com/lh3/minimap2
- **samtools** (Converts minimap2 SAM output to BAM for the quantifiers)
- **NanoCount** (DRS-specific quantifier under test; applies 3'-end <=50nt and secondary AS >=95% filtering before EM) — https://github.com/a-slide/NanoCount
- **Salmon** (Competing quantifier run with --noErrorModel on the same alignments)
- **StringTie2** (Competing quantifier run in default mode)
- **FLAIR** (Competing long-read quantifier run in default mode)
- **LIQA** (Competing DRS-aware quantifier run in default mode)

## Examples

```
minimap2 -ax map-ont -N 10 SIRV_transcriptome.fasta SRR6058583.fastq | samtools view -bh > aligned_reads.bam && nanocount --reads aligned_reads.bam --count_out abundance.tsv
```

## Evaluation signals

- For mix E0 (equal concentrations), the coefficient of variance across transcripts should be lowest for the best quantifier (NanoCount expected)
- For mix E2 (varied concentrations), Spearman's rank correlation rho between known and measured abundance should be highest for NanoCount among the tested quantifiers
- The assembled TSV metrics should match the corresponding entries of Supplementary Table S1 (output_matches_reference / expert_review)
- minimap2 must be run with -N 10 so secondary mappings are retained; running without it will starve EM-based multi-mapping resolution
- Each quantifier must consume identical input alignments to ensure a fair comparison across tools

## Limitations

- NanoCount relies on polyA+ DRS reads; incomplete reads largely confined to 3'UTRs cannot specify the expressed isoform, limiting quantification of lowly expressed transcripts
- Modest sequencing depth makes detection and quantification of lowly expressed genes and isoforms challenging
- Benchmark conclusions from spike-ins should be extrapolated with caution because the spike-in transcriptome is less complex than the human one
- Results depend on annotation completeness; complete (C), insufficient (I) and over-annotation (O) variants change relative tool performance

## Evidence

- [results] NanoCount showed the best quantification performance on both SIRV mixes: [evidence span withheld — anchor status: not_found]
- [results] DRS-specific filtering thresholds: "discard read alignments with 3′ ends >50nt away from the transcript 3′ end and secondary alignments with an AS <95% that of the best alignment"
- [results] EM-based quantification: "NanoCount performs DRS specific filtering steps to improve transcript assignment prior to quantifying DRS reads using an expectation-maximisation algorithm"
- [results] Competing tools: "We tested the performance of NanoCount against other transcript quantification programs, including Salmon, StringTie2, FLAIR and LIQA"
- [results] Minimap2 alignment to transcriptome: "Reads from each sample were aligned using minimap2 (29) to the human and sequin reference genome and transcriptome."
- [readme] NanoCount EM on multi-mapping DRS reads: "NanoCount estimates transcripts abundance from Oxford Nanopore *direct-RNA sequencing* datasets, using an expectation-maximization approach like RSEM, Kallisto, salmon, etc to handle the uncertainty"
