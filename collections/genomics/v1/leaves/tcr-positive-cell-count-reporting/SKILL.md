---
name: tcr-positive-cell-count-reporting
description: Use when an ONT- or PacBio-sequenced 10x Genomics 5'-anchored TCR/cDNA
  amplicon FASTQ has been processed with nanoranger in 5p10X_XCR mode and you need
  to count cell barcodes with a detected TCR and benchmark against Illumina-derived
  counts.
when_to_use_negative:
- The input is raw ONT/PacBio long reads that have not yet been processed by nanoranger
  — run the 5p10X_XCR workflow first; this skill only aggregates and counts per-cell
  calls.
- The library is a whole-transcriptome scRNA-seq or SNV/fusion/isoform nanoranger
  run rather than a TCR-targeted 10x amplicon — per-cell TCR counting does not apply.
- You need clone-level clonotype annotation or V(D)J diversity metrics rather than
  a simple count of TCR-positive cells; a dedicated repertoire tool (e.g. MiXCR) is
  the wrong scope here but a different analysis is needed.
edam_topics:
- http://edamontology.org/topic_2830
tools:
- name: nanoranger
  role: Processes the ONT-sequenced 10x TCR amplicon FASTQ in 5p10X_XCR mode to call
    TCR sequences per 10x cell barcode; its per-cell calls are what this skill aggregates
    and counts.
  repo: github.com/mehdiborji/nanoranger
- name: Oxford Nanopore Technologies (ONT)
  role: Long-read sequencing platform that generates the TCR amplicon FASTQ input;
    ONT counts are benchmarked against Illumina sequencing of the same library.
- name: 10x Genomics
  role: Provides the 5'-anchored scRNA-seq cell barcodes and V(DJ) cDNA library chemistry
    from which the TCR amplicons were targeted and amplified.
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-023-44137-7
    title: Integrative genotyping of cancer and immune phenotypes by long-read sequencing
  source_content_hash: sha256:b7bc85a63314ae5cc2a367636e6b0f4e4d684eb6302dc004b932f5c71ca047d7
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - 10x Genomics
  - Oxford Nanopore Technologies (ONT)
  - nanoranger
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_2830
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# TCR-positive cell count reporting

## Summary

Aggregates per-cell TCR calls produced by nanoranger in 5p10X_XCR mode from ONT-sequenced 10x Genomics TCR cDNA amplicons into a single count of cells with a detectable TCR, reportable as tcr_cell_counts.csv and checkable against the paper's reference value of 5653 ONT cells (vs. 5767 Illumina cells).

## When to use

Use when an ONT- or PacBio-sequenced 10x Genomics 5'-anchored TCR/cDNA amplicon FASTQ has been processed with nanoranger in 5p10X_XCR mode and you need to count cell barcodes with a detected TCR and benchmark against Illumina-derived counts.

## When NOT to use

- The input is raw ONT/PacBio long reads that have not yet been processed by nanoranger — run the 5p10X_XCR workflow first; this skill only aggregates and counts per-cell calls.
- The library is a whole-transcriptome scRNA-seq or SNV/fusion/isoform nanoranger run rather than a TCR-targeted 10x amplicon — per-cell TCR counting does not apply.
- You need clone-level clonotype annotation or V(D)J diversity metrics rather than a simple count of TCR-positive cells; a dedicated repertoire tool (e.g. MiXCR) is the wrong scope here but a different analysis is needed.

## Inputs

- ONT-sequenced FASTQ of a melanoma tumor-infiltrating T cell TCR cDNA library (10x Genomics V(DJ) kit, targeted amplification), provided with the nanoranger repository
- nanoranger per-cell TCR call table keyed by 10x cell barcode (5p10X_XCR mode output)

## Outputs

- tcr_cell_counts.csv containing the number of cells with a detected TCR sequence (reference value: 5653 for the ONT sample)
- Per-cell TCR-positive/TCR-negative assignment by 10x cell barcode for downstream concordance analysis

## How to apply

Obtain the sample FASTQ of the 10x Genomics-derived TCR cDNA library from the nanoranger repository and run the nanoranger workflow in 5p10X_XCR mode, which aligns the long reads (using the minimap2/STAR-family alignment strategy in the pipeline) and assigns TCR sequences to 10x cell barcodes. Aggregate the per-cell TCR calls over all cell barcodes and count each barcode only once as TCR-positive. Write the count to tcr_cell_counts.csv and verify it against the paper-reported reference of 5653 ONT TCR-positive cells, which the study shows is highly concordant with the 5767 TCR-positive cells called from Illumina sequencing of the same library (correlations of r=0.8 and r=0.9).

## Related tools

- **nanoranger** (Processes the ONT-sequenced 10x TCR amplicon FASTQ in 5p10X_XCR mode to call TCR sequences per 10x cell barcode; its per-cell calls are what this skill aggregates and counts.) — github.com/mehdiborji/nanoranger
- **Oxford Nanopore Technologies (ONT)** (Long-read sequencing platform that generates the TCR amplicon FASTQ input; ONT counts are benchmarked against Illumina sequencing of the same library.)
- **10x Genomics** (Provides the 5'-anchored scRNA-seq cell barcodes and V(DJ) cDNA library chemistry from which the TCR amplicons were targeted and amplified.)

## Evaluation signals

- The count written to tcr_cell_counts.csv matches the paper-reported reference of 5653 ONT cells with a detected TCR for the melanoma sample FASTQ.
- Per-cell TCR calls show high concordance with the Illumina-derived call set (5767 TCR-positive cells), with reported correlations of r=0.8 and r=0.9.
- Every 10x cell barcode contributes at most one TCR-positive assignment (no duplicated barcodes in the aggregation).
- The output file tcr_cell_counts.csv exists and contains a single well-formed integer count.
- The ONT count is within a few percent of the Illumina count (5653 vs. 5767, ~2% difference), consistent with the reported high concordance.

## Limitations

- Detection sensitivity is bounded by transcript representation: the study notes the ceiling for genotyping is set by low, incomplete transcript representation and cannot be overcome by amplicon optimization or deeper sequencing.
- Counts are library- and platform-specific; the 5653 reference applies only to the ONT-sequenced melanoma sample FASTQ distributed with nanoranger.
- UMIs in 10x Genomics libraries have no whitelist, so downstream processing (e.g. nanoranger.R) collapses UMI sequencing error within a Levenshtein distance of 3 using starcode; uncorrected error can affect per-cell assignments.

## Evidence

- [abstract] nanoranger flexible long-read workflow: "we devise a flexible long-read sequencing workflow and analysis pipeline, termed nanoranger, that starts from intermediate single-cell cDNA libraries"
- [abstract] TCR detection from cDNA libraries: "sequences of chimeric antigen and TCRs"
- [readme] input data definition: "The input data can be obtained through sequencing of 10x Genomics whole-transcriptome cDNA libraries or amplicons obtained through targeted amplification, with Oxford Nanopore Technologies (ONT) or"
- [intro] ONT-targeted amplification pipeline rationale: "we provide a long-read based pipeline for the ONT platform using limited primer sets to amplify target genes from 5′-anchored 10x Genomics scRNA-seq whole-transcriptome cDNA libraries"
- [readme] UMI error correction in downstream processing: [evidence span withheld — anchor status: external_doc]
