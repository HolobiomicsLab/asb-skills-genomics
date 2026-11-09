---
name: mas-seq-multimer-read-processing
description: Use when input is PacBio MAS-seq or concatenated multimer FASTQ/BAM data
  (e.g., 15-mer arrays) and multiple 10x-barcoded transcripts must be extracted per
  read before alignment and UMI/cell-barcode processing.
when_to_use_negative:
- Input is standard single-transcript ONT or PacBio single-molecule reads that are
  not concatenated multimers — deconcatenation is unnecessary.
- Starting data is an already-processed feature-barcode matrix or post-alignment BAM
  without multimer structure.
- Short-read Illumina data are supplied — nanoranger's multimer workflow targets long-read
  PacBio/ONT inputs only.
edam_operation: http://edamontology.org/operation_3216
edam_topics:
- http://edamontology.org/topic_0654
- http://edamontology.org/topic_3170
tools:
- name: nanoranger
  role: Runs the 5p10XGEX pipeline that deconcatenates multimer reads into subreads
    and assigns 10x cell barcodes/UMIs
  repo: https://github.com/mehdiborji/nanoranger
- name: Pacific Biosciences
  role: Long-read sequencing platform that generates the MAS-seq multimer arrays being
    deconcatenated
- name: minimap2
  role: Aligns deconcatenated subreads against mitochondrial/transcript references
    for mapping statistics
  repo: https://github.com/lh3/minimap2
- name: 10x Genomics
  role: Source of the 5'-anchored whole-transcriptome cDNA libraries whose barcoded
    transcripts populate each multimer read
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-023-44137-7
    title: Integrative genotyping of cancer and immune phenotypes by long-read sequencing
  source_content_hash: sha256:3c210fd76dfe8c7e57468b37634a6658f47f1e7d4d02dc3e8133a7e7fc1fba74
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mehdiborji/nanoranger
  tools:
  - 10x Genomics
  - Pacific Biosciences
  - minimap2
  - nanoranger
  edam_operation: http://edamontology.org/operation_3216
  edam_topics:
  - http://edamontology.org/topic_0654
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# mas-seq-multimer-read-processing

## Summary

Deconcatenation of PacBio MAS-seq / MAS-ISO-seq multimer reads into individual subread transcripts using nanoranger, enabling recovery of multiple barcoded single-cell transcripts from a single long read (e.g., 15-mer arrays).

## When to use

Use when input is PacBio MAS-seq or concatenated multimer FASTQ/BAM data (e.g., 15-mer arrays) and multiple 10x-barcoded transcripts must be extracted per read before alignment and UMI/cell-barcode processing.

## When NOT to use

- Input is standard single-transcript ONT or PacBio single-molecule reads that are not concatenated multimers — deconcatenation is unnecessary.
- Starting data is an already-processed feature-barcode matrix or post-alignment BAM without multimer structure.
- Short-read Illumina data are supplied — nanoranger's multimer workflow targets long-read PacBio/ONT inputs only.

## Inputs

- PacBio MAS-seq / MAS-ISO-seq multimer reads in FASTQ or BAM format (e.g., 1019_mtDNA.fastq.gz 15-mer arrays)
- Reference transcript/chromosome sequences (e.g., mitochondrial genome/transcriptome FASTA)

## Outputs

- Deconcatenated subreads with 10x cell barcode (CB) and UMI tags
- Mapped reads (e.g., via minimap2 against mitochondrial references)
- subread_deconcatenation_stats.tsv (subread counts, mapped reads, max transcripts per read)

## How to apply

Obtain the multimer dataset (e.g., 1019_mtDNA.fastq.gz, PacBio 15-mer MAS-seq arrays from github.com/mehdiborji/nanoranger) and build a reference of the target transcripts (here, mitochondrial transcript and chromosome sequences). Run nanoranger in 5p10XGEX mode against these references so that each multimer read is split into subreads carrying their 10x cell barcode and UMI. Inspect the deconcatenation log to confirm multiple subreads per multimer read; use the paper's 15-mer artificial benchmark (10.7M deconcatenated vs 8.5M mapped reads via longbow) as the expected recovery ratio. Summarize deconcatenated subread counts, mapped reads, and maximum transcripts per read (paper reports up to 478 transcripts from a single read across 13 amplicon libraries) into subread_deconcatenation_stats.tsv.

## Related tools

- **nanoranger** (Runs the 5p10XGEX pipeline that deconcatenates multimer reads into subreads and assigns 10x cell barcodes/UMIs) — https://github.com/mehdiborji/nanoranger
- **Pacific Biosciences** (Long-read sequencing platform that generates the MAS-seq multimer arrays being deconcatenated)
- **minimap2** (Aligns deconcatenated subreads against mitochondrial/transcript references for mapping statistics) — https://github.com/lh3/minimap2
- **10x Genomics** (Source of the 5'-anchored whole-transcriptome cDNA libraries whose barcoded transcripts populate each multimer read)

## Examples

```
nanoranger run --mode 5p10XGEX --input 1019_mtDNA.fastq.gz --reference mt_transcripts.fa --output deconcatenated/
```

## Evaluation signals

- Deconcatenation recovers substantially more reads than the mapped-read count, consistent with the 10.7M vs 8.5M ratio reported for the 15-mer benchmark.
- Deconcatenation log shows multiple subreads extracted per multimer input read.
- Coverage distributions of deconcatenated subreads are similar between nanoranger and longbow benchmarks.
- subread_deconcatenation_stats.tsv reports a plausible maximum transcripts-per-read value (paper reports up to 478 transcripts from a single read across 13 amplicon libraries).
- Extracted subreads carry valid 10x cell barcodes and UMIs assignable via the 5p10XGEX mode.

## Limitations

- Recovery depends on multimer length (15-mer arrays benchmarked); shorter arrays yield proportionally fewer transcripts per read.
- Maximum transcripts per read observed (478 across 13 amplicon libraries) bounds expected yields; substantially lower recovery suggests failure.
- Genotyping ceiling is limited by low, incomplete transcript representation and cannot be merely overcome by amplicon optimization or deeper sequencing.

## Evidence

- [other] on artificially generated 15-mer MAS-ISO-seq arrays it extracted 10.7 vs 8.5 million mapped reads with longbow: [evidence span withheld — anchor status: not_found]
- [other] workflow description from source card: [evidence span withheld — anchor status: not_found]
- [methods] nanoranger input data description: "The input data can be obtained through sequencing of 10x Genomics whole-transcriptome cDNA libraries or amplicons obtained through targeted amplification"
- [abstract] nanoranger devise statement: "we devise a flexible long-read sequencing workflow and analysis pipeline, termed nanoranger, that starts from intermediate single-cell cDNA libraries"
- [readme] platform description from README: "with Oxford Nanopore Technologies (ONT) or Pacific Biosciences devices. The schematic of our workflow is demonstrated below."
