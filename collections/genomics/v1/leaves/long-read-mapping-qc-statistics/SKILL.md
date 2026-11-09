---
name: long-read-mapping-qc-statistics
description: Use when a nanoranger (5p10XGEX) run processes multimeric long reads
  (e.g., PacBio 15-mer MAS-seq arrays or natural ONT multimers) and you must confirm
  deconcatenation succeeded before downstream genotyping.
when_to_use_negative:
- Input reads are already single-transcript (non-multimer) long reads, so deconcatenation
  ratios are not a meaningful QC signal
- You are evaluating short-read (Illumina) single-cell data, where the paper notes
  short reads cannot resolve long fragments or structural variants
- Downstream analysis has already moved past mapping to genotyping/isoform calling;
  use nanoranger.R-style variant/UMI-collapse QC instead
tools:
- name: nanoranger
  role: 'Primary pipeline: 5p10XGEX mode deconcatenates multimer reads into subreads
    and maps them to the provided references; its log supplies the QC statistics'
  repo: https://github.com/mehdiborji/nanoranger
- name: longbow
  role: External deconcatenation benchmark used for comparison (8.5M mapped reads
    vs nanoranger's 10.7M on 15-mer MAS-ISO-seq arrays)
- name: Pacific Biosciences
  role: Sequencing platform that generated the 15-mer MAS-seq multimer arrays used
    as the deconcatenation benchmark
- name: 10x Genomics
  role: Source of the 5'-anchored whole-transcriptome scRNA-seq cDNA libraries that
    are the upstream input to nanoranger
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-023-44137-7
    title: Integrative genotyping of cancer and immune phenotypes by long-read sequencing
  source_content_hash: sha256:96744ebc5d5aefe926b35978de98ee39ec22bdbb7a797ce528f5db2bbf051636
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mehdiborji/nanoranger
  tools:
  - 10x Genomics
  - Pacific Biosciences
  - longbow
  - nanoranger
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 10
---

# long-read-mapping-qc-statistics

## Summary

Quality-control and statistics workflow for nanoranger long-read single-cell runs: verify that multimer-read deconcatenation recovers substantially more subreads than the final mapped-read count, and summarize deconcatenation/mapping yields into a stats table.

## When to use

Use when a nanoranger (5p10XGEX) run processes multimeric long reads (e.g., PacBio 15-mer MAS-seq arrays or natural ONT multimers) and you must confirm deconcatenation succeeded before downstream genotyping.

## When NOT to use

- Input reads are already single-transcript (non-multimer) long reads, so deconcatenation ratios are not a meaningful QC signal
- You are evaluating short-read (Illumina) single-cell data, where the paper notes short reads cannot resolve long fragments or structural variants
- Downstream analysis has already moved past mapping to genotyping/isoform calling; use nanoranger.R-style variant/UMI-collapse QC instead

## Inputs

- Long-read FASTQ from PacBio MAS-seq arrays or ONT sequencing of 10x Genomics whole-transcriptome cDNA libraries or targeted amplicons (e.g., 1019_mtDNA.fastq.gz)
- Reference sequences (e.g., mitochondrial transcript and chromosome FASTA) for alignment
- nanoranger deconcatenation log files

## Outputs

- subread_deconcatenation_stats.tsv with deconcatenated subread counts, mapped-read counts, and maximum transcripts per read
- Deconcatenated subread FASTQ/BAM used for downstream mapping
- Mapped alignment records (e.g., BAM) of subreads against the reference

## How to apply

Run nanoranger in 5p10XGEX mode on the input FASTQ (e.g., 1019_mtDNA.fastq.gz, PacBio 15-mer MAS-seq arrays) against the target references only, enabling multimer-read deconcatenation into subreads. Read the deconcatenation log and confirm that multiple subreads are extracted per multimer read. Compare the number of deconcatenated subreads to the mapped-read count: validation accepts the run when deconcatenation recovers substantially more reads than mapped, consistent with the paper's 15-mer artificial benchmark (10.7M deconcatenated vs 8.5M mapped reads, also comparable to the longbow tool). Record the deconcatenated subread count, mapped-read count, and maximum transcripts recovered per read (paper reports up to 478 transcripts from a single read across 13 amplicon libraries) into a tab-separated summary such as subread_deconcatenation_stats.tsv.

## Related tools

- **nanoranger** (Primary pipeline: 5p10XGEX mode deconcatenates multimer reads into subreads and maps them to the provided references; its log supplies the QC statistics) — https://github.com/mehdiborji/nanoranger
- **longbow** (External deconcatenation benchmark used for comparison (8.5M mapped reads vs nanoranger's 10.7M on 15-mer MAS-ISO-seq arrays))
- **Pacific Biosciences** (Sequencing platform that generated the 15-mer MAS-seq multimer arrays used as the deconcatenation benchmark)
- **10x Genomics** (Source of the 5'-anchored whole-transcriptome scRNA-seq cDNA libraries that are the upstream input to nanoranger)

## Examples

```
nanoranger 5p10XGEX --fastq 1019_mtDNA.fastq.gz --reference mtDNA_transcripts.fa --output nanoranger_out; cat nanoranger_out/logs/deconcatenation.log > subread_deconcatenation_stats.tsv
```

## Evaluation signals

- Deconcatenated subread count substantially exceeds the mapped-read count (benchmark: 10.7M vs 8.5M for 15-mer MAS-ISO-seq arrays), confirming multimer splitting
- Deconcatenation log shows multiple subreads extracted per multimer read
- Coverage distributions of nanoranger-deconcatenated reads are similar to the longbow benchmark
- subread_deconcatenation_stats.tsv reports a plausible maximum transcripts-per-read value (paper: up to 478 transcripts from a single read)
- Mapped reads align only against the supplied references (e.g., mitochondrial transcript/chromosome sequences)

## Limitations

- Genotyping sensitivity is bounded by transcript representation: the ceiling is set by low, incomplete representation of many transcripts and cannot be overcome merely by deeper sequencing or amplicon optimization
- Optimal nanoranger targets are loci near the 5' end of highly expressed genes with transcript lengths shorter than 4 kB; targets outside this regime give sparse coverage
- Validation ratio (10.7M vs 8.5M) was established on artificially generated 15-mer MAS-ISO-seq arrays; naturally occurring multimers may show different deconcatenation yields

## Evidence

- [abstract] nanoranger as flexible long-read workflow: "we devise a flexible long-read sequencing workflow and analysis pipeline, termed nanoranger, that starts from intermediate single-cell cDNA libraries"
- [abstract] optimal target definition: "we define the optimal targets for nanoranger, namely those loci close to the 5' end of highly expressed genes with transcript lengths shorter than 4 kB"
- [readme] input data sources: "The input data can be obtained through sequencing of 10x Genomics whole-transcriptome cDNA libraries or amplicons obtained through targeted amplification, with Oxford Nanopore Technologies (ONT) or"
- [intro] long-read platforms: "PacBio and Oxford Nanopore Technologies (ONT) are two established long-read sequencing platforms."
