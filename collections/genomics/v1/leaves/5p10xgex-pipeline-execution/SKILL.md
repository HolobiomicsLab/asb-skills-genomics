---
name: 5p10xgex-pipeline-execution
description: Use when FASTQ files from long-read (ONT/PacBio) sequencing of 10x Genomics
  scRNA-seq cDNA libraries or targeted amplicons need alignment, variant calling,
  and per-cell genotype calls keyed by 10x cell barcodes.
when_to_use_negative:
- Input data are Illumina short-read libraries, since short-read sequencing is suboptimal
  for long fragments and cannot resolve structural transcriptomic variants such as
  fusions or isoforms
- The goal is full-transcriptome quantification rather than targeted genotyping of
  specific loci; nanoranger targets loci close to the 5' end of highly expressed genes
  with transcript lengths shorter than 4 kB
- Cells lack 5'-anchored 10x Genomics cDNA structure (e.g. 3'-only chemistries), since
  the pipeline depends on 5' cell barcodes and UMIs
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0625
tools:
- name: nanoranger
  role: 'Core pipeline: aligns long-read amplicon/cDNA reads in 5p10XGEX mode and
    calls single-nucleotide variants from 10x barcoded libraries'
  repo: https://github.com/mehdiborji/nanoranger
- name: nanoranger.R
  role: 'R downstream processing: reads nanoranger output to extract SNVs, indels,
    fusion genes/CAR sequences and isoforms; collapses UMIs with starcode within a
    Levenshtein distance of 3'
  repo: https://github.com/liviuspenter/nanoranger.R
- name: Oxford Nanopore Technologies (ONT)
  role: Long-read sequencing platform producing the input reads for the 5p10XGEX workflow
- name: 10x Genomics
  role: Source of the 5'-anchored whole-transcriptome cDNA libraries that supply cell
    barcodes and UMIs
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-023-44137-7
    title: Integrative genotyping of cancer and immune phenotypes by long-read sequencing
  source_content_hash: sha256:1a430666af53dc850cab101e4d59a89ad154c117f2c3de7d975bcaf5c7fcd2b5
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mehdiborji/nanoranger
  tools:
  - 10x Genomics
  - Oxford Nanopore Technologies (ONT)
  - nanoranger
  - nanoranger.R
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0625
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9230769230769231
n_claims: 13
---

# 5p10xgex-pipeline-execution

## Summary

Executes the nanoranger long-read single-cell analysis pipeline in 5p10XGEX mode, aligning amplicon reads from 5'-anchored 10x Genomics whole-transcriptome cDNA libraries and calling SNVs to build per-cell genotyping matrices for somatic mutations.

## When to use

Use when FASTQ files from long-read (ONT/PacBio) sequencing of 10x Genomics scRNA-seq cDNA libraries or targeted amplicons need alignment, variant calling, and per-cell genotype calls keyed by 10x cell barcodes.

## When NOT to use

- Input data are Illumina short-read libraries, since short-read sequencing is suboptimal for long fragments and cannot resolve structural transcriptomic variants such as fusions or isoforms
- The goal is full-transcriptome quantification rather than targeted genotyping of specific loci; nanoranger targets loci close to the 5' end of highly expressed genes with transcript lengths shorter than 4 kB
- Cells lack 5'-anchored 10x Genomics cDNA structure (e.g. 3'-only chemistries), since the pipeline depends on 5' cell barcodes and UMIs

## Inputs

- FASTQ file from ONT long-read sequencing of 10x Genomics whole-transcriptome cDNA libraries or targeted amplicons (e.g. 1022_DNMT3A_RUNX1_SF3B1.fastq.gz)
- Reference genome for alignment
- Target variant definitions (e.g. DNMT3AR882H, RUNX1I177S, SF3B1K700E)

## Outputs

- Aligned reads (BAM/SAM) from 5p10XGEX-mode alignment to the reference genome
- Single-nucleotide variant calls from the nanoranger variant-calling module
- Per-cell genotyping matrix keyed by 10x cell barcodes
- genotyping_summary.csv with per-cell genotype calls and per-mutation rate columns

## How to apply

Load the sample FASTQ (e.g. 1022_DNMT3A_RUNX1_SF3B1.fastq.gz, derived from a 10x Genomics whole-transcriptome cDNA library) as input to nanoranger and run it in 5p10XGEX mode so amplicon reads targeting genes of interest (e.g. DNMT3A, RUNX1, SF3B1) are aligned to the reference genome. Call single-nucleotide variants from the aligned reads using nanoranger's variant-calling module, then collapse variant calls into a per-cell genotyping matrix keyed by 10x cell barcodes. Compute per-mutation genotyping rates and compare them against expected values — for AML1022 relapse, 4.8–19.2% for DNMT3AR882H/SF3B1K700E and ~39.2% for RUNX1I177S. Nanoranger achieves genotyping rates comparable to the Illumina-based GoT protocol (5.5–19.1% for the same AML1022 mutations), so cross-platform agreement is a validation benchmark.

## Related tools

- **nanoranger** (Core pipeline: aligns long-read amplicon/cDNA reads in 5p10XGEX mode and calls single-nucleotide variants from 10x barcoded libraries) — https://github.com/mehdiborji/nanoranger
- **nanoranger.R** (R downstream processing: reads nanoranger output to extract SNVs, indels, fusion genes/CAR sequences and isoforms; collapses UMIs with starcode within a Levenshtein distance of 3) — https://github.com/liviuspenter/nanoranger.R
- **Oxford Nanopore Technologies (ONT)** (Long-read sequencing platform producing the input reads for the 5p10XGEX workflow)
- **10x Genomics** (Source of the 5'-anchored whole-transcriptome cDNA libraries that supply cell barcodes and UMIs)

## Examples

```
nanoranger align --mode 5p10XGEX --fastq 1022_DNMT3A_RUNX1_SF3B1.fastq.gz --reference GRCh38.fa --outdir nanoranger_out
```

## Evaluation signals

- Computed per-mutation genotyping rates fall within reported ranges: 4.8–19.2% for DNMT3AR882H and SF3B1K700E and ~39.2% for RUNX1I177S in AML1022 relapse
- genotyping_summary.csv exists and contains per-cell genotype calls plus per-mutation rate columns
- Per-cell genotyping matrix is keyed by valid 10x cell barcodes
- Genotyping rates are concordant with the Illumina-based GoT protocol benchmark (5.5–19.1% for DNMT3AR882H/SF3B1K700E)
- Alignment runs complete in 5p10XGEX mode without errors on the input FASTQ

## Limitations

- Optimal targets are loci close to the 5' end of highly expressed genes with transcript lengths shorter than 4 kB; loci outside this regime genotype poorly
- The ceiling for any genotyping approach is determined by the low, incomplete representation of many transcripts, including absence of longer variants, and cannot be overcome through amplicon optimization or deeper sequencing
- Current long-read flow cells generate only 5–50 million reads, constraining multiplexing depth

## Evidence

- [results] Nanoranger and GoT sequenced with Illumina achieved similar genotyping rates for DNMT3AR882H and SF3B1K700E: 4.8–19.2% for nanoranger versus 5.5–19.1% for GoT.: [evidence span withheld — anchor status: not_found]
- [abstract] we devise a flexible long-read sequencing workflow and analysis pipeline, termed nanoranger, that starts from intermediate single-cell cDNA libraries: "we devise a flexible long-read sequencing workflow and analysis pipeline, termed nanoranger, that starts from intermediate single-cell cDNA libraries"
- [readme] The input data can be obtained through sequencing of 10x Genomics whole-transcriptome cDNA libraries or amplicons obtained through targeted amplification, with Oxford Nanopore Technologies (ONT) or Pacific Biosciences devices.: "The input data can be obtained through sequencing of 10x Genomics whole-transcriptome cDNA libraries or amplicons obtained through targeted amplification, with Oxford Nanopore Technologies (ONT) or"
- [abstract] we define the optimal targets for nanoranger, namely those loci close to the 5' end of highly expressed genes with transcript lengths shorter than 4 kB: "we define the optimal targets for nanoranger, namely those loci close to the 5' end of highly expressed genes with transcript lengths shorter than 4 kB"
- [discussion] the ceiling for any genotyping approach is currently determined by the low, incomplete representation of many transcripts including absence of longer variants: "the ceiling for any genotyping approach is currently determined by the low, incomplete representation of many transcripts including absence of longer variants"
