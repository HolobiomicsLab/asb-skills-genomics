---
name: car-versus-wildtype-cd28-genotyping
description: Use when a 10x Genomics 5'-anchored scRNA-seq cDNA library has been targeted-amplified
  for the CAR/CD28 locus and long reads must assign each cell barcode to CAR transgene
  versus wildtype CD28.
when_to_use_negative:
- Only unenriched Illumina short-read data are available — short reads cannot resolve
  the CAR transgene structure without prior enrichment
- The target transcript is longer than ~4 kb or not close to the 5' end — outside
  the loci the paper defines as optimal for nanoranger
- No CAR_CD28 reference exists; reads cannot be assigned to CAR versus wildtype CD28
  transcript classes
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3308
tools:
- name: nanoranger
  role: 'Core pipeline run in 5p10XGEX mode: aligns long reads to the CAR_CD28 reference,
    extracts cell barcodes/UMIs, and assigns reads to CAR or wildtype CD28 classes'
  repo: https://github.com/mehdiborji/nanoranger
- name: nanoranger.R
  role: R downstream package that reads out fusion genes/CAR sequences from nanoranger
    amplicon output and collapses error-prone UMIs via starcode
  repo: https://github.com/liviuspenter/nanoranger.R
- name: Oxford Nanopore Technologies (ONT)
  role: Long-read sequencing device used to sequence the targeted CAR/CD28 amplicon
    from the intermediate cDNA library
- name: 10x Genomics
  role: Source of the 5'-anchored whole-transcriptome cDNA library from which amplicons
    are obtained by targeted amplification
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-023-44137-7
    title: Integrative genotyping of cancer and immune phenotypes by long-read sequencing
  source_content_hash: sha256:e56aa4e984f9099faac8560e1bcf5c28ed349015374bb7ea8cee1dce50d59781
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
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 11
---

# CAR-versus-wildtype-CD28-genotyping

## Summary

Nanoranger-based long-read genotyping that discriminates CD19 CAR transgene transcripts from endogenous wildtype CD28 using a CD28-specific primer amplicon sequenced on ONT, recovering >75% CAR+ T cells in the infusion product.

## When to use

Use when a 10x Genomics 5'-anchored scRNA-seq cDNA library has been targeted-amplified for the CAR/CD28 locus and long reads must assign each cell barcode to CAR transgene versus wildtype CD28.

## When NOT to use

- Only unenriched Illumina short-read data are available — short reads cannot resolve the CAR transgene structure without prior enrichment
- The target transcript is longer than ~4 kb or not close to the 5' end — outside the loci the paper defines as optimal for nanoranger
- No CAR_CD28 reference exists; reads cannot be assigned to CAR versus wildtype CD28 transcript classes

## Inputs

- FASTQ of targeted amplicons sequenced on ONT from an intermediate 10x Genomics 5'-anchored whole-transcriptome cDNA library (e.g. 97_6_CAR.fastq.gz)
- CAR_CD28 combined reference covering the CAR construct and endogenous CD28 locus

## Outputs

- Per-cell quantification table car_wt_quantification.csv with cell barcode, CAR and wildtype CD28 UMI counts, and transcript class
- Overall fraction of CAR+ cells

## How to apply

Start from an intermediate 10x Genomics 5'-anchored whole-transcriptome cDNA library, amplify the target with a CD28-specific primer, and sequence the amplicon on an Oxford Nanopore device to produce FASTQ (e.g. 97_6_CAR.fastq.gz). Run nanoranger in 5p10XGEX mode with a CAR_CD28 reference so reads align to both the chimeric antigen receptor construct and the endogenous CD28 locus. Nanoranger extracts 10x cell barcodes and UMIs from the long reads and assigns each read to the CAR or wildtype CD28 transcript class. Collapse UMI counts per cell barcode — downstream via nanoranger.R/starcode, UMIs with sequencing error are collapsed within a Levenshtein distance of 3 because, unlike cell barcodes, 10x UMIs have no whitelist — and classify each cell as CAR+ or wildtype CD28 by the dominant transcript class. Summarize the overall CAR+ cell fraction and export the per-cell table as car_wt_quantification.csv.

## Related tools

- **nanoranger** (Core pipeline run in 5p10XGEX mode: aligns long reads to the CAR_CD28 reference, extracts cell barcodes/UMIs, and assigns reads to CAR or wildtype CD28 classes) — https://github.com/mehdiborji/nanoranger
- **nanoranger.R** (R downstream package that reads out fusion genes/CAR sequences from nanoranger amplicon output and collapses error-prone UMIs via starcode) — https://github.com/liviuspenter/nanoranger.R
- **Oxford Nanopore Technologies (ONT)** (Long-read sequencing device used to sequence the targeted CAR/CD28 amplicon from the intermediate cDNA library)
- **10x Genomics** (Source of the 5'-anchored whole-transcriptome cDNA library from which amplicons are obtained by targeted amplification)

## Examples

```
nanoranger --mode 5p10XGEX --fastq 97_6_CAR.fastq.gz --reference CAR_CD28; R -e "nanorangerR::export_quantification(out='car_wt_quantification.csv')"
```

## Evaluation signals

- Computed fraction of CAR+ cells is above 0.75, matching the reported >75% CAR+ T cell detection
- CAR+ cell set includes all CAR+ cells found by unenriched Illumina scRNA-seq plus ~15% additional cells
- car_wt_quantification.csv contains one row per cell barcode with CAR and wildtype CD28 UMI counts and a dominant-class assignment
- Per-read assignments resolve to exactly two classes (CAR vs wildtype CD28) with no ambiguous dominating class in classified cells
- UMI collapse (starcode) groups erroneous UMIs within Levenshtein distance 3 before counting, avoiding inflated per-cell transcript counts

## Limitations

- Optimal nanoranger targets are loci close to the 5' end of highly expressed genes with transcript lengths shorter than 4 kB
- The genotyping ceiling is determined by the low, incomplete representation of transcripts in the intermediate cDNA library and cannot be overcome merely by amplicon optimization or deeper sequencing
- 10x UMIs have no whitelist for correcting sequencing error, so UMI collapse must rely on distance-based clustering rather than exact matching

## Evidence

- [abstract] Nanoranger detects lineage-defining features from single-cell cDNA libraries: "starts from intermediate single-cell cDNA libraries to detect cell lineage-defining features, including single-nucleotide variants, fusion genes, isoforms, sequences of chimeric antigen and TCRs"
- [abstract] Nanoranger workflow origin: "we devise a flexible long-read sequencing workflow and analysis pipeline, termed nanoranger, that starts from intermediate single-cell cDNA libraries"
- [readme] Input data generation: "The input data can be obtained through sequencing of 10x Genomics whole-transcriptome cDNA libraries or amplicons obtained through targeted amplification"
- [readme] Downstream readout includes CAR sequences: [evidence span withheld — anchor status: external_doc]
- [readme] UMI error collapse rationale: [evidence span withheld — anchor status: external_doc]
