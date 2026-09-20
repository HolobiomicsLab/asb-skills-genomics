---
name: nanopore-amplicon-pipeline-execution
description: Use when an ONT- (or PacBio-) sequenced 10x Genomics 5' cDNA or targeted
  TCR amplicon FASTQ must be converted into per-cell-barcode TCR calls and a TCR-positive
  cell count comparable to Illumina-based 10x V(DJ) results.
when_to_use_negative:
- Input is Illumina short-read data — nanoranger is built for long reads from ONT
  or Pacific Biosciences devices, and short reads cannot resolve the full-length targeted
  amplicons
- Input is a raw 10x whole-transcriptome FASTQ with no targeted amplification of the
  locus of interest — nanoranger defines optimal targets as loci near the 5' end of
  highly expressed genes with transcripts shorter than 4 kB; unsuitable loci hit the
  genotyping ceiling
- The analysis goal is standard gene-expression quantification (count matrix / clustering)
  rather than lineage-defining features such as TCR sequences
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_3304
- http://edamontology.org/topic_0102
tools:
- name: nanoranger
  role: 'Primary pipeline: aligns long reads from the targeted TCR amplicon and calls
    TCR sequences per 10x cell barcode in 5p10X_XCR mode'
  repo: https://github.com/mehdiborji/nanoranger
- name: nanoranger.R
  role: R package for downstream processing of nanoranger output; collapses UMIs with
    starcode within Levenshtein distance 3 and reads out variants/fusions/CAR-TCR
    sequences
  repo: liviuspenter/nanoranger.R
- name: minimap2
  role: Long-read alignment engine supporting noisy ONT cDNA reads (e.g. minimap2
    -ax map-ont), underlying the read-to-locus assignment step
  repo: https://github.com/lh3/minimap2
- name: MiXCR
  role: Alternative/complementary universal software for TCR repertoire analysis of
    single-cell 10x data
  repo: milaboratory/mixcr
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-023-44137-7
    title: Integrative genotyping of cancer and immune phenotypes by long-read sequencing
  source_content_hash: sha256:8ac396abaf13d4b4c1b91f2e286ccddc180eebd4c0e57766c76bea3e86e52bae
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mehdiborji/nanoranger
  tools:
  - 10x Genomics
  - MiXCR
  - Oxford Nanopore Technologies (ONT)
  - minimap2
  - nanoranger
  - nanoranger.R
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_3304
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# nanopore-amplicon-pipeline-execution

## Summary

Run the nanoranger long-read workflow in 5p10X_XCR mode to align ONT-sequenced 10x Genomics TCR amplicon reads and call TCR sequences per cell barcode, then aggregate per-cell calls to a total TCR-positive cell count. This converts targeted-amplification long-read FASTQ into a quantitative, Illumina-comparable per-cell TCR detection result.

## When to use

Use when an ONT- (or PacBio-) sequenced 10x Genomics 5' cDNA or targeted TCR amplicon FASTQ must be converted into per-cell-barcode TCR calls and a TCR-positive cell count comparable to Illumina-based 10x V(DJ) results.

## When NOT to use

- Input is Illumina short-read data — nanoranger is built for long reads from ONT or Pacific Biosciences devices, and short reads cannot resolve the full-length targeted amplicons
- Input is a raw 10x whole-transcriptome FASTQ with no targeted amplification of the locus of interest — nanoranger defines optimal targets as loci near the 5' end of highly expressed genes with transcripts shorter than 4 kB; unsuitable loci hit the genotyping ceiling
- The analysis goal is standard gene-expression quantification (count matrix / clustering) rather than lineage-defining features such as TCR sequences

## Inputs

- Sample FASTQ of a 10x Genomics TCR cDNA library (targeted amplicon, e.g. melanoma tumor-infiltrating T cell V(DJ) library) sequenced on an Oxford Nanopore Technologies device
- nanoranger workflow in 5p10X_XCR mode (pipeline configuration bundled with the mehdiborji/nanoranger repository)

## Outputs

- Per-cell TCR calls assigning detected TCR sequences to 10x cell barcodes
- tcr_cell_counts.csv containing the number of cells with a detectable TCR (expected 5653 for the reference melanoma ONT library)

## How to apply

Obtain the sample FASTQ generated from 10x Genomics cDNA (here a melanoma tumor-infiltrating T cell TCR library from the 10x Genomics V(DJ) kit) by targeted amplification and ONT sequencing. Execute the nanoranger workflow in 5p10X_XCR mode: it aligns the long reads (leveraging minimap2-style long-read mapping against the TCR target locus) and assigns TCR sequences to individual 10x cell barcodes carried in the read IDs. Aggregate the per-cell TCR calls so that each cell barcode contributes at most a detected/undetected TCR status, then count TCR-positive cells. Write the resulting count to tcr_cell_counts.csv and verify it against the paper-reported reference of 5653 ONT cells with a detected TCR (Illumina reference: 5767 cells, ONT–Illumina concordance r=0.8 and r=0.9). For downstream UMI-level correction, nanoranger.R collapses UMIs corrupted by sequencing error into groups within a Levenshtein distance of 3 using starcode, since 10x UMIs lack a whitelist for error correction.

## Related tools

- **nanoranger** (Primary pipeline: aligns long reads from the targeted TCR amplicon and calls TCR sequences per 10x cell barcode in 5p10X_XCR mode) — https://github.com/mehdiborji/nanoranger
- **nanoranger.R** (R package for downstream processing of nanoranger output; collapses UMIs with starcode within Levenshtein distance 3 and reads out variants/fusions/CAR-TCR sequences) — liviuspenter/nanoranger.R
- **minimap2** (Long-read alignment engine supporting noisy ONT cDNA reads (e.g. minimap2 -ax map-ont), underlying the read-to-locus assignment step) — https://github.com/lh3/minimap2
- **MiXCR** (Alternative/complementary universal software for TCR repertoire analysis of single-cell 10x data) — milaboratory/mixcr

## Evaluation signals

- tcr_cell_counts.csv exists and contains 5653 TCR-positive cells for the reference melanoma ONT sample
- Per-cell TCR detection is highly concordant with the Illumina-sequenced reference (5767 TCR-positive cells; reported concordance r=0.8 and r=0.9)
- Every TCR call maps to a valid 10x cell barcode with associated UMI, confirming correct barcode/UMI handling from the read IDs
- UMI groups collapse within Levenshtein distance 3 (starcode), consistent with ONT sequencing-error correction rather than genuine receptor diversity
- Detected TCR sequences are validated full-length V(DJ) rearrangements, not spurious partial alignments to the target locus

## Limitations

- Genotyping ceiling is set by the low, incomplete representation of transcripts in the starting cDNA library and cannot be overcome merely by amplicon optimization or deeper sequencing
- Optimal nanoranger targets are restricted: loci close to the 5' end of highly expressed genes with transcript lengths shorter than 4 kB
- ONT data analysis for single-cell applications must handle the platform's higher sequencing error, particularly for cell barcode and UMI correction (no UMI whitelist exists, hence the Levenshtein-distance-3 starcode collapse)

## Evidence

- [abstract] we devise a flexible long-read sequencing workflow and analysis pipeline, termed nanoranger, that starts from intermediate single-cell cDNA libraries: "we devise a flexible long-read sequencing workflow and analysis pipeline, termed nanoranger, that starts from intermediate single-cell cDNA libraries"
- [abstract] we define the optimal targets for nanoranger, namely those loci close to the 5' end of highly expressed genes with transcript lengths shorter than 4 kB: "we define the optimal targets for nanoranger, namely those loci close to the 5' end of highly expressed genes with transcript lengths shorter than 4 kB"
- [readme] The input data can be obtained through sequencing of 10x Genomics whole-transcriptome cDNA libraries or amplicons obtained through targeted amplification: "The input data can be obtained through sequencing of 10x Genomics whole-transcriptome cDNA libraries or amplicons obtained through targeted amplification"
- [discussion] the ceiling for any genotyping approach is currently determined by the low, incomplete representation of many transcripts including absence of longer variants: "the ceiling for any genotyping approach is currently determined by the low, incomplete representation of many transcripts including absence of longer variants"
