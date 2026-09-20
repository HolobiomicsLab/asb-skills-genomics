---
name: long-read-subread-deconcatenation
description: Use when input FASTQ/BAM contains multimer or concatenated long reads
  (e.g., PacBio MAS-seq 15-mer arrays or ONT multimers) and multiple transcript-level
  records must be recovered from a single read.
when_to_use_negative:
- Input reads are already single-transcript subreads or a deconcatenated FASTQ/BAM,
  so no multimer splitting is needed
- The analysis goal is gene expression from standard 10x Genomics short-read libraries
  rather than long-read multimer recovery
- Task requires only barcode/UMI processing without reference-guided deconcatenation
  and mapping
tools:
- name: nanoranger
  role: Pipeline that deconcatenates multimer reads into subreads (run in 5p10XGEX
    mode with mitochondrial references)
  repo: https://github.com/mehdiborji/nanoranger
- name: minimap2
  role: Long-read aligner used to map deconcatenated subreads to the reference sequences
  repo: https://github.com/lh3/minimap2
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-023-44137-7
    title: Integrative genotyping of cancer and immune phenotypes by long-read sequencing
  source_content_hash: sha256:ff5ccefec9b8d20399e493ffd68890cbc8d03ca031eb5b30c2ad112ddc7861ff
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mehdiborji/nanoranger
  tools:
  - 10x Genomics
  - Pacific Biosciences
  - minimap2
  - nanoranger
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# long-read-subread-deconcatenation

## Summary

Split multimer long reads (naturally occurring ONT multimers or artificially concatenated PacBio MAS-seq 15-mer arrays) into individual transcript subreads using nanoranger, then map the extracted subreads to references to quantify how many transcripts were recovered per original read.

## When to use

Use when input FASTQ/BAM contains multimer or concatenated long reads (e.g., PacBio MAS-seq 15-mer arrays or ONT multimers) and multiple transcript-level records must be recovered from a single read.

## When NOT to use

- Input reads are already single-transcript subreads or a deconcatenated FASTQ/BAM, so no multimer splitting is needed
- The analysis goal is gene expression from standard 10x Genomics short-read libraries rather than long-read multimer recovery
- Task requires only barcode/UMI processing without reference-guided deconcatenation and mapping

## Inputs

- Multimer/concatenated long-read FASTQ (e.g., 1019_mtDNA.fastq.gz, PacBio 15-mer MAS-seq arrays)
- Reference transcript/chromosome sequences (e.g., mitochondrial chromosome FASTA)

## Outputs

- Deconcatenated subreads per multimer read
- Mapped subread alignments against the reference
- subread_deconcatenation_stats.tsv with subread counts, mapped reads, and max transcripts per read
- Deconcatenation log confirming multiple subreads per multimer read

## How to apply

Obtain the multimer input (e.g., the repository sample 1019_mtDNA.fastq.gz of PacBio 15-mer MAS-seq arrays from github.com/mehdiborji/nanoranger) and build mitochondrial transcript/chromosome reference sequences. Run nanoranger in 5p10XGEX mode against the mitochondrial references only, which enables multimer-read deconcatenation into subreads. Inspect the deconcatenation log to confirm that multiple subreads are extracted per multimer read, using the paper's 15-mer artificial benchmark (10.7M deconcatenated vs 8.5M mapped reads, comparable to longbow) as the reference ratio. Map the deconcatenated subreads (e.g., with minimap2, which supports '-ax map-pb' for PacBio CLR and '-ax splice:hq' for PacBio Kinnex/Iso-seq) and tabulate the number of deconcatenated subreads, mapped reads, and maximum transcripts recovered per read (the paper reports up to 478 transcripts from a single read across 13 amplicon libraries) into subread_deconcatenation_stats.tsv.

## Related tools

- **nanoranger** (Pipeline that deconcatenates multimer reads into subreads (run in 5p10XGEX mode with mitochondrial references)) — https://github.com/mehdiborji/nanoranger
- **minimap2** (Long-read aligner used to map deconcatenated subreads to the reference sequences) — https://github.com/lh3/minimap2

## Examples

```
nanoranger 5p10XGEX --genome MT.fa --fastq 1019_mtDNA.fastq.gz --output deconcat_out
```

## Evaluation signals

- Deconcatenation recovers substantially more reads than the mapped-read count, consistent with the reported 10.7M vs 8.5M ratio for the 15-mer benchmark
- Coverage distributions of nanoranger-deconcatenated reads are similar to those from longbow on the artificial 15-mer MAS-ISO-seq arrays
- The deconcatenation log shows multiple subreads extracted per multimer read
- subread_deconcatenation_stats.tsv contains subread counts, mapped-read counts, and a maximum transcripts-per-read value (paper maximum: 478 transcripts from a single read)

## Limitations

- Genotyping ceiling is determined by low, incomplete transcript representation and cannot be overcome merely by amplicon optimization or deeper sequencing
- Maximum transcripts recoverable per read and total throughput depend on long-read flow cell yields (5–50 million reads per flow cell)
- Deconcatenation results should be benchmarked against an alternative splitter (e.g., longbow) since absolute counts depend on the algorithm and 15-mer arrays are artificial benchmarks

## Evidence

- [abstract] we devise a flexible long-read sequencing workflow and analysis pipeline, termed nanoranger: "we devise a flexible long-read sequencing workflow and analysis pipeline, termed nanoranger, that starts from intermediate single-cell cDNA libraries"
- [readme] Input data are 10x cDNA libraries or targeted amplicons sequenced on ONT or PacBio: "The input data can be obtained through sequencing of 10x Genomics whole-transcriptome cDNA libraries or amplicons obtained through targeted amplification, with Oxford Nanopore Technologies (ONT) or"
- [abstract] nanoranger detects lineage-defining features from single-cell cDNA libraries: "starts from intermediate single-cell cDNA libraries to detect cell lineage-defining features, including single-nucleotide variants, fusion genes, isoforms, sequences of chimeric antigen and TCRs"
- [readme] PacBio Kinnex/Iso-seq splice-aware mapping preset in minimap2: [evidence span withheld — anchor status: not_found]
- [discussion] Enrichment for genes of interest augments long-read structural variant detection: "enriching for genes of interest augments the inherent ability of long-read sequencing to detect structural transcriptomic variants"
