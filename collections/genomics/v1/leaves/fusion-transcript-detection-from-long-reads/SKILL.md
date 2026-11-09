---
name: fusion-transcript-detection-from-long-reads
description: Use when ONT long-read FASTQ from a 5'-anchored 10x Genomics scRNA-seq
  cDNA library must be screened for chimeric fusion-junction reads and fusion-positive
  cells counted per cell barcode.
when_to_use_negative:
- Fusion breakpoints lie far from the 5' end or target transcripts exceed ~4 kB and
  are lowly expressed — nanoranger detection sensitivity drops substantially
- Input is short-read sequencing data that cannot resolve structural transcriptomic
  variants such as gene fusions
- The goal is bulk (non-single-cell) fusion detection without cell barcode/UMI information
edam_operation: http://edamontology.org/operation_3563
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3512
tools:
- name: nanoranger
  role: Primary long-read processing pipeline run in 5p10XGEX fusion mode to align
    reads and detect chimeric fusion-junction reads with cell barcodes and UMIs
  repo: https://github.com/mehdiborji/nanoranger
- name: nanoranger.R
  role: R downstream package to read out fusion genes/CAR sequences from nanoranger
    output, collapsing UMIs with starcode within a Levenshtein distance of 3
  repo: https://github.com/liviuspenter/nanoranger.R
- name: 10x Genomics
  role: 'Source chemistry: 5''-anchored whole-transcriptome scRNA-seq cDNA libraries
    supplying cell barcodes and UMIs'
- name: Oxford Nanopore Technologies (ONT)
  role: Long-read sequencing platform generating the input FASTQ reads
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-023-44137-7
    title: Integrative genotyping of cancer and immune phenotypes by long-read sequencing
  source_content_hash: sha256:143c15291e60072e81ae1e7efdfcb52616b8dcf49f0af2243ca7dae0841b3c29
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mehdiborji/nanoranger
  tools:
  - 10x Genomics
  - Oxford Nanopore Technologies (ONT)
  - nanoranger
  - nanoranger.R
  edam_operation: http://edamontology.org/operation_3563
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3512
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# fusion-transcript-detection-from-long-reads

## Summary

Detects fusion transcripts (e.g., BCR::ABL1, RUNX1::RUNX1T1) at single-cell resolution by aligning ONT long reads from 5'-anchored 10x Genomics cDNA libraries with nanoranger in 5p10XGEX fusion mode, then tabulating fusion-positive cells via the fusion_gene.py downstream script.

## When to use

Use when ONT long-read FASTQ from a 5'-anchored 10x Genomics scRNA-seq cDNA library must be screened for chimeric fusion-junction reads and fusion-positive cells counted per cell barcode.

## When NOT to use

- Fusion breakpoints lie far from the 5' end or target transcripts exceed ~4 kB and are lowly expressed — nanoranger detection sensitivity drops substantially
- Input is short-read sequencing data that cannot resolve structural transcriptomic variants such as gene fusions
- The goal is bulk (non-single-cell) fusion detection without cell barcode/UMI information

## Inputs

- Gzipped FASTQ of ONT long reads from a 10x Genomics 5' whole-transcriptome cDNA library (e.g., K562_Kasumi1_BCRABL1_RUNX1_RUNX1T1.fastq.gz)
- Reference transcript sequences spanning the target fusion junctions (BCR-ABL1, RUNX1-RUNX1T1)

## Outputs

- fusion_calls.tsv table with BCR::ABL1 and RUNX1::RUNX1T1 entries including cell barcode and UMI fields
- Per-cell fusion-positive counts and fractions (e.g., 124/2.7% K562 cells BCR::ABL1+, 88/3.9% Kasumi-1 cells RUNX1::RUNX1T1+)

## How to apply

Run nanoranger in 5p10XGEX fusion mode on the sample FASTQ (e.g., K562_Kasumi1_BCRABL1_RUNX1_RUNX1T1.fastq.gz), aligning long reads against reference transcripts spanning the BCR-ABL1 and RUNX1-RUNX1T1 fusion junctions. Parse the aligned records to retain chimeric reads spanning either fusion breakpoint together with their 10x cell barcode and UMI sequences. Execute the downstream fusion_gene.py script to deduplicate barcode-UMI pairs per cell and tabulate fusion-supporting reads for each fusion gene. Compare the resulting fusion-positive cell counts and fractions against the benchmark of 124 (2.7%) BCR::ABL1+ K562 cells and 88 (3.9%) RUNX1::RUNX1T1+ Kasumi-1 cells; low fractions are expected because fusion breakpoints distant from the 5' end reduce capture efficiency.

## Related tools

- **nanoranger** (Primary long-read processing pipeline run in 5p10XGEX fusion mode to align reads and detect chimeric fusion-junction reads with cell barcodes and UMIs) — https://github.com/mehdiborji/nanoranger
- **nanoranger.R** (R downstream package to read out fusion genes/CAR sequences from nanoranger output, collapsing UMIs with starcode within a Levenshtein distance of 3) — https://github.com/liviuspenter/nanoranger.R
- **10x Genomics** (Source chemistry: 5'-anchored whole-transcriptome scRNA-seq cDNA libraries supplying cell barcodes and UMIs)
- **Oxford Nanopore Technologies (ONT)** (Long-read sequencing platform generating the input FASTQ reads)

## Examples

```
nanoranger --mode 5p10XGEX --fusion --input K562_Kasumi1_BCRABL1_RUNX1_RUNX1T1.fastq.gz && python fusion_gene.py --calls fusion_calls.tsv
```

## Evaluation signals

- fusion_calls.tsv exists and contains BCR::ABL1 and RUNX1::RUNX1T1 entries with populated cell barcode and UMI fields
- Fraction of fusion-positive K562 cells falls near the reported 2.7% (124 cells) benchmark
- Fraction of fusion-positive Kasumi-1 cells falls near the reported 3.9% (88 cells) benchmark
- Barcode-UMI deduplication is applied (UMIs collapsed, e.g., within Levenshtein distance of 3 via starcode) before counting fusion-positive cells
- Fusion-positive fractions are low single-digit percentages, consistent with breakpoints distant from the 5' end

## Limitations

- Fusion breakpoints distant from the 5' end yield low detection rates (only 2.7% of K562 and 3.9% of Kasumi-1 cells fusion-positive in the mixing benchmark)
- Optimal nanoranger targets are loci close to the 5' end of highly expressed genes with transcript lengths shorter than 4 kB; fusions outside this regime are under-detected
- The genotyping ceiling is set by low, incomplete transcript representation and cannot be merely overcome through amplicon optimization or deeper sequencing

## Evidence

- [other] Nanoranger's fusion detection workflow on the K562/Kasumi-1 mixing sample identified fusion transcripts in only 124 (2.7%) K562 cells (BCR::ABL1) and 88 (3.9%) Kasumi-1 cells (RUNX1::RUNX1T1): [evidence span withheld — anchor status: not_found]
- [abstract] we define the optimal targets for nanoranger, namely those loci close to the 5' end of highly expressed genes with transcript lengths shorter than 4 kB: "we define the optimal targets for nanoranger, namely those loci close to the 5' end of highly expressed genes with transcript lengths shorter than 4 kB"
- [intro] we provide a long-read based pipeline for the ONT platform using limited primer sets to amplify target genes from 5′-anchored 10x Genomics scRNA-seq whole-transcriptome cDNA libraries: "amplify target genes from 5′-anchored 10x Genomics scRNA-seq whole-transcriptome cDNA libraries"
- [abstract] starts from intermediate single-cell cDNA libraries to detect cell lineage-defining features, including single-nucleotide variants, fusion genes, isoforms: "starts from intermediate single-cell cDNA libraries to detect cell lineage-defining features, including single-nucleotide variants, fusion genes"
- [readme] UMIs that arise due to sequencing error are collapsed in groups within a Levenshtein distance of 3: [evidence span withheld — anchor status: external_doc]
