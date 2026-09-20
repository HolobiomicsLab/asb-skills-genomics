---
name: sirv-spike-in-isoform-assignment
description: Use when a demultiplexing or cell-barcode correction method must be quantitatively
  validated with a spike-in whose true isoform and barcode identities are known, e.g.
  barcoded SIRV-Set 4 FLNC reads from PacBio CCS data.
when_to_use_negative:
- Data contain no spike-in controls with known barcode/isoform ground truth — accuracy
  cannot be computed without expected SIRV assignments
- Reads are short-read (e.g. Illumina NextSeq) single-end gene-expression data that
  never undergo HIT-scISOseq concatenation, so the concatenated-read demultiplexing
  benchmark does not apply
edam_operation: http://edamontology.org/operation_2928
edam_topics:
- http://edamontology.org/topic_0080
- http://edamontology.org/topic_3170
tools:
- name: scISA-Tools
  role: Demultiplexes HIT-scISOseq concatenated reads into single-cell cDNA reads
    and performs cell barcode/UMI correction (cellBC_UMI_corrector wrapping CellRanger
    correction) against the whitelist
  repo: https://github.com/shizhuoxing/scISA-Tools
- name: CellRanger
  role: Provides the cell barcode and UMI correction function that scISA-Tools wraps
    as the cellBC_UMI_corrector module
  repo: https://www.10xgenomics.com/support/software/cell-ranger/downloads/previous-versions
- name: 10x Genomics
  role: Source of the 3M-february-2018.txt barcode whitelist used as the reference
    set for barcode correction during SIRV benchmarking
- name: PacBio Sequel II
  role: Generates the circular consensus (CCS) reads from the SMRT Cell 8M that underlie
    the SIRV FLNC reads being benchmarked
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-023-38324-9
    title: High-throughput and high-accuracy single-cell RNA isoform analysis using
      PacBio circular consensus sequencing
  source_content_hash: sha256:35534d84ee3b824b924a9387030a8debbc1e50abe1ee4ab0e84a86d9a041de59
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/shizhuoxing/scISA-Tools
  tools:
  - 10x Genomics
  - CellRanger
  - PacBio Sequel II
  - scISA-Tools
  edam_operation: http://edamontology.org/operation_2928
  edam_topics:
  - http://edamontology.org/topic_0080
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8461538461538461
n_claims: 13
---

# sirv-spike-in-isoform-assignment

## Summary

Benchmark demultiplexing accuracy of scISA-Tools on HIT-scISOseq concatenated reads using spiked-in barcoded SIRV-Set 4 (Lexogen) FLNC reads, computing barcode assignment accuracy/specificity and a SIRV isoform confusion matrix against known ground truth.

## When to use

Use when a demultiplexing or cell-barcode correction method must be quantitatively validated with a spike-in whose true isoform and barcode identities are known, e.g. barcoded SIRV-Set 4 FLNC reads from PacBio CCS data.

## When NOT to use

- Data contain no spike-in controls with known barcode/isoform ground truth — accuracy cannot be computed without expected SIRV assignments
- Reads are short-read (e.g. Illumina NextSeq) single-end gene-expression data that never undergo HIT-scISOseq concatenation, so the concatenated-read demultiplexing benchmark does not apply

## Inputs

- Barcoded SIRV-Set 4 (Lexogen) FLNC reads (BarcodeUMI fastq) from the spiked-in HIT-scISOseq condition
- 10x Genomics cell barcode whitelist 3M-february-2018.txt (available in the scISA-Tools example_data folder or CellRanger path)
- Known SIRV-Set 4 spike-in isoform reference (ground-truth isoform identities)

## Outputs

- SIRV isoform assignment confusion matrix (sirv_demux_confusion_matrix.csv) matching Fig. 2b-d
- Demultiplexing accuracy and specificity metrics (demux_metrics.csv)

## How to apply

Load the barcoded SIRV-Set 4 FLNC reads (BarcodeUMI fastq from the spiked-in condition, generated after CCS with '--min-passes 0 --min-length 50 --max-length 21000 --min-rq 0.75' and classify_by_primer with '-min_primerlen 16 -min_seqlen 50'). Run scISA-Tools demultiplexing to split HIT-scISOseq concatenated reads into single-cell cDNA reads, correcting cell barcodes against the 10x Genomics whitelist 3M-february-2018.txt using the CellRanger-derived cellBC_UMI_corrector with a posterior probability threshold >0.975, and applying UMI filters (no homopolymers, no N bases, no base quality <10). Assign each demultiplexed read to its expected SIRV-Set 4 isoform identity using the known SIRV spike-in reference, then build the isoform assignment confusion matrix (as in Fig. 2b-d) comparing demultiplexed assignments against ground truth. Compute overall accuracy and specificity from the confusion matrix; the rationale is that SIRV isoforms are known a priori, so any misassignment reflects a method error rather than transcript-inference uncertainty. Validation requires recomputed metrics to reproduce the reported 99.997% accuracy, 99.998% specificity, and 0.1066% isoform confusion rate.

## Related tools

- **scISA-Tools** (Demultiplexes HIT-scISOseq concatenated reads into single-cell cDNA reads and performs cell barcode/UMI correction (cellBC_UMI_corrector wrapping CellRanger correction) against the whitelist) — https://github.com/shizhuoxing/scISA-Tools
- **CellRanger** (Provides the cell barcode and UMI correction function that scISA-Tools wraps as the cellBC_UMI_corrector module) — https://www.10xgenomics.com/support/software/cell-ranger/downloads/previous-versions
- **10x Genomics** (Source of the 3M-february-2018.txt barcode whitelist used as the reference set for barcode correction during SIRV benchmarking)
- **PacBio Sequel II** (Generates the circular consensus (CCS) reads from the SMRT Cell 8M that underlie the SIRV FLNC reads being benchmarked)

## Examples

```
scISA-Tools demux --input sirv_flnc.fq --whitelist 3M-february-2018.txt --posterior 0.975 --umi-filter homopolymer,N,bq10
```

## Evaluation signals

- Recomputed barcode assignment accuracy reproduces the reported 99.997% on barcoded SIRV FLNC reads
- Recomputed specificity reproduces the reported 99.998%
- SIRV isoform assignment confusion rate is as low as 0.1066%
- Confusion matrix is nearly diagonal and matches the published Fig. 2b-d layout
- All retained UMIs pass the quality filters: no homopolymers, no N bases, no base quality <10; corrected cell barcodes have posterior probability >0.975

## Limitations

- The benchmark is only demonstrated on the 10x Genomics single-cell platform; compatibility with other platforms is claimed but not demonstrated
- Accuracy estimates depend on the SIRV spike-in reference being exact; real transcriptomes lack such complete ground truth, so metrics may not transfer to endogenous isoforms
- CellBC correction is limited to whitelist barcodes (3M-february-2018.txt) with posterior probability >0.975, so non-whitelist true barcodes would be dropped or misassigned

## Evidence

- [abstract] Demultiplexing accuracy metric: "We also report the development of scISA-Tools that demultiplex HIT-scISOseq concatenated reads into single-cell cDNA reads with >99.99% accuracy and specificity."
- [discussion] Barcode correction threshold: [evidence span withheld — anchor status: not_found]
- [discussion] UMI quality filters: "Must not be a homopolymer, e.g. AAAAAAAAAA;Must not contain N;Must not contain bases with base quality <10"
- [methods] Whitelist source: "you can find `3M-february-2018.txt.gz` file in example_data folder in this repo or in your cellranger path"
- [methods] CellRanger correction wrapper: "we have warps `CellRanger` cell barcode and UMI correction function as a module in our pipeline, named cellBC_UMI_corrector"
