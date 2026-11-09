---
name: long-read-barcode-demultiplexing
description: Use when demultiplexing PacBio HiFi/FLNC long reads (e.g., HIT-scISOseq
  concatenated cDNA) into per-cell reads, or when benchmarking barcode assignment
  accuracy against a ground-truth spike-in such as SIRV-Set 4.
when_to_use_negative:
- Input is short-read single-cell data already handled by CellRanger's mkfastq/count
  barcode processing, not concatenated long reads
- Barcodes do not come from a 10x Genomics-style 3'-tagged capture chemistry with
  a matching whitelist; posterior-probability correction depends on the 3M-february-2018
  whitelist
- Reads are raw CCS BAM without prior primer mapping, cell barcode/UMI extraction,
  and polyA trimming (FLNC classification must precede demultiplexing)
edam_operation: http://edamontology.org/operation_3680
edam_topics:
- http://edamontology.org/topic_0080
- http://edamontology.org/topic_3170
tools:
- name: scISA-Tools
  role: 'Core demultiplexing pipeline: splits HIT-scISOseq concatenated reads into
    single-cell cDNA reads, corrects cell barcodes against the whitelist (cellBC_UMI_corrector),
    and builds gene/isoform count matrices'
  repo: https://github.com/shizhuoxing/scISA-Tools
- name: CellRanger
  role: Source of the 10x Genomics cell barcode and UMI correction function that scISA-Tools
    wraps as the cellBC_UMI_corrector module; CellRanger 3.1.0 referenced in README
  repo: https://www.10xgenomics.com/support/software/cell-ranger/downloads/previous-versions
- name: 10x Genomics
  role: Single-cell capture chemistry and barcode whitelist (3M-february-2018.txt)
    against which cellBC posterior-probability correction is performed
- name: PacBio Sequel II
  role: Sequencing platform producing the CCS reads (SMRT Cell 8M) from which FLNC
    reads are generated before demultiplexing
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-023-38324-9
    title: High-throughput and high-accuracy single-cell RNA isoform analysis using
      PacBio circular consensus sequencing
  source_content_hash: sha256:546ce28b21dc9ca6c6013240b707b547d29c15d6854e1669bc01cbdb94df3d8b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/shizhuoxing/scISA-Tools
  tools:
  - 10x Genomics
  - CellRanger
  - PacBio Sequel II
  - scISA-Tools
  edam_operation: http://edamontology.org/operation_3680
  edam_topics:
  - http://edamontology.org/topic_0080
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 5
---

# long-read-barcode-demultiplexing

## Summary

Demultiplex concatenated long-read (HIT-scISOseq) sequences into single-cell cDNA reads by correcting 10x Genomics cell barcodes against a whitelist and applying UMI quality filters, achieving >99.99% assignment accuracy and specificity as validated on spiked-in barcoded SIRV-Set 4 FLNC reads.

## When to use

Use when demultiplexing PacBio HiFi/FLNC long reads (e.g., HIT-scISOseq concatenated cDNA) into per-cell reads, or when benchmarking barcode assignment accuracy against a ground-truth spike-in such as SIRV-Set 4.

## When NOT to use

- Input is short-read single-cell data already handled by CellRanger's mkfastq/count barcode processing, not concatenated long reads
- Barcodes do not come from a 10x Genomics-style 3'-tagged capture chemistry with a matching whitelist; posterior-probability correction depends on the 3M-february-2018 whitelist
- Reads are raw CCS BAM without prior primer mapping, cell barcode/UMI extraction, and polyA trimming (FLNC classification must precede demultiplexing)

## Inputs

- FLNC reads (FASTQ) with extracted cell barcode and UMI, e.g. from spiked-in barcoded SIRV-Set 4 condition
- 10x Genomics cell barcode whitelist 3M-february-2018.txt (or 3M-february-2018.txt.gz)
- Known SIRV-Set 4 spike-in isoform reference for ground-truth assignment

## Outputs

- Demultiplexed single-cell cDNA reads (per-cell BarcodeUMI assignments)
- SIRV isoform assignment confusion matrix (sirv_demux_confusion_matrix.csv)
- Demultiplexing accuracy and specificity metrics (demux_metrics.csv)

## How to apply

Run scISA-Tools demultiplexing on the FLNC reads (BarcodeUMI fastq from classify_by_primer with -min_primerlen 16 -min_seqlen 50) using the 10x Genomics whitelist 3M-february-2018.txt; the cellBC_UMI_corrector module wraps CellRanger's barcode/UMI correction, replacing an observed barcode with the whitelist barcode of highest posterior probability only when that probability exceeds 0.975. Apply UMI quality filters in parallel: reject homopolymer UMIs (e.g., AAAAAAAAAA), UMIs containing N, and UMIs with any base quality <10. For accuracy benchmarking, assign each demultiplexed read to its expected SIRV-Set 4 isoform identity using the known spike-in reference, build the isoform confusion matrix (Fig. 2b-d), and compute overall accuracy and specificity from it. Correct application is validated by reproducing 99.997% accuracy, 99.998% specificity, and a confusion rate as low as 0.1066%.

## Related tools

- **scISA-Tools** (Core demultiplexing pipeline: splits HIT-scISOseq concatenated reads into single-cell cDNA reads, corrects cell barcodes against the whitelist (cellBC_UMI_corrector), and builds gene/isoform count matrices) — https://github.com/shizhuoxing/scISA-Tools
- **CellRanger** (Source of the 10x Genomics cell barcode and UMI correction function that scISA-Tools wraps as the cellBC_UMI_corrector module; CellRanger 3.1.0 referenced in README) — https://www.10xgenomics.com/support/software/cell-ranger/downloads/previous-versions
- **10x Genomics** (Single-cell capture chemistry and barcode whitelist (3M-february-2018.txt) against which cellBC posterior-probability correction is performed)
- **PacBio Sequel II** (Sequencing platform producing the CCS reads (SMRT Cell 8M) from which FLNC reads are generated before demultiplexing)

## Examples

```
python scISA-Tools/demux.py --flnc sirv_flnc.fq --whitelist 3M-february-2018.txt --correct --prob-threshold 0.975 --outdir ./
```

## Evaluation signals

- Demultiplexing accuracy on barcoded SIRV-Set 4 FLNC reads reproduces 99.997% (reported >99.99%)
- Specificity reproduces 99.998% (reported >99.99%)
- SIRV isoform assignment confusion rate is as low as 0.1066% and the confusion matrix matches Fig. 2b-d
- All corrected cell barcodes trace to whitelist entries with posterior probability >0.975
- Rejected UMIs include no homopolymers, no N bases, and no bases with quality <10

## Limitations

- Nanopore-based long-read demultiplexing suffers low cell-barcode demultiplex efficiency due to relatively high read error rate; the >99.99% figures are specific to PacBio HiFi/CCS data
- The 10x Genomics ScISOr-Seq preparation pipeline introduces a high proportion (~50%) of undesirable cell-barcode-free reads, mostly TSO artifacts, which must be removed upstream
- Compatibility with single-cell platforms other than 10x Genomics is claimed but not demonstrated in this study

## Evidence

- [abstract] scISA-Tools demultiplexes HIT-scISOseq concatenated reads into single-cell cDNA reads with >99.99% accuracy and specificity: "demultiplex HIT-scISOseq concatenated reads into single-cell cDNA reads with >99.99% accuracy and specificity"
- [discussion] Barcode correction threshold and mechanism: [evidence span withheld — anchor status: not_found]
- [discussion] UMI quality filter criteria: "Must not be a homopolymer, e.g. AAAAAAAAAA;Must not contain N;Must not contain bases with base quality <10"
- [discussion] FLNC read generation parameters preceding demultiplexing: [evidence span withheld — anchor status: not_found]
- [methods] cellBC_UMI_corrector wraps CellRanger correction: "we have warps `CellRanger` cell barcode and UMI correction function as a module in our pipeline, named cellBC_UMI_corrector"
- [methods] Whitelist file location documented in repository: "you can find `3M-february-2018.txt.gz` file in example_data folder in this repo or in your cellranger path"
- [intro] Nanopore demultiplex efficiency limitation: "its cell-barcode demultiplex efficiency is low due to the relatively high read error rate"
