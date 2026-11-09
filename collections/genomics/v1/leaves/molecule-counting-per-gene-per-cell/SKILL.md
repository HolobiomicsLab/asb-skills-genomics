---
name: molecule-counting-per-gene-per-cell
description: Use when processing PacBio MAS-Seq (MAS-ISO-seq) single-cell cDNA reads
  to deconcatenate concatenated transcripts and quantify molecules per gene and per
  cell, especially when comparing against skera.
when_to_use_negative:
- Input is standard 10x Genomics short-read scRNA-seq FASTQ rather than concatenated
  MAS-Seq long reads — deconcatenation/skera comparison does not apply
- Data are ONT amplicon reads for targeted genotyping (SNVs, fusions, CAR/TCR) — use
  the nanoranger.R downstream pipeline instead of MAS-ISO-seq molecule counting
- Only bulk long-read Iso-Seq data without cell barcodes/UMIs — per-cell molecule
  counting is impossible without single-cell barcode structure
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0622
tools:
- name: nanoranger
  role: End-to-end pipeline for deconcatenating MAS-Seq reads and quantifying molecules
    per gene and per cell from long-read single-cell data
  repo: github.com/mehdiborji/nanoranger
- name: Pacific Biosciences
  role: Sequencing platform and data source (Revio MAS-Seq run; skera is PacBio's
    competing deconcatenation tool)
- name: minimap2
  role: Spliced long-read aligner available for mapping Iso-Seq/cDNA reads against
    the transcriptome reference within long-read single-cell workflows
  repo: https://github.com/lh3/minimap2
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-023-44137-7
    title: Integrative genotyping of cancer and immune phenotypes by long-read sequencing
  source_content_hash: sha256:e3550f764d93a2ebcb5925d676e9940d858aa82aecb6e8d9ea4d62fdd1f58c49
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Pacific Biosciences
  - minimap2
  - nanoranger
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# molecule-counting-per-gene-per-cell

## Summary

Deconcatenates long-read single-cell MAS-ISO-seq reads into segments and counts molecules per gene and per cell using nanoranger, benchmarking against PacBio's skera on PBMC data (DATA-Revio-PBMC-2).

## When to use

Use when processing PacBio MAS-Seq (MAS-ISO-seq) single-cell cDNA reads to deconcatenate concatenated transcripts and quantify molecules per gene and per cell, especially when comparing against skera.

## When NOT to use

- Input is standard 10x Genomics short-read scRNA-seq FASTQ rather than concatenated MAS-Seq long reads — deconcatenation/skera comparison does not apply
- Data are ONT amplicon reads for targeted genotyping (SNVs, fusions, CAR/TCR) — use the nanoranger.R downstream pipeline instead of MAS-ISO-seq molecule counting
- Only bulk long-read Iso-Seq data without cell barcodes/UMIs — per-cell molecule counting is impossible without single-cell barcode structure

## Inputs

- PacBio Revio MAS-Seq (MAS-ISO-seq) long-read single-cell cDNA FASTQ/BAM files (e.g. DATA-Revio-PBMC-2 from downloads.pacbcloud.com)
- GENCODE v44 transcriptome reference
- skera benchmark values (median 15 segments/read; 55 molecules/gene) for comparison

## Outputs

- Per-read segment counts from the nanoranger output
- Per-gene and per-cell molecule count matrices (mean 141 vs 55 molecules/gene; 2580 vs 1650 molecules/cell)
- Side-by-side benchmark table nanoranger_vs_skera_comparison.csv (segments-per-read and molecules-per-gene rows with skera reference columns)

## How to apply

Download the public PacBio Revio MAS-Seq run (e.g. DATA-Revio-PBMC-2) from downloads.pacbcloud.com and run the nanoranger workflow (github.com/mehdiborji/nanoranger) end-to-end, supplying GENCODE v44 as the transcriptome alignment reference and configuring the TruSeq-adapter barcode search as described in the Methods. Extract per-read segment counts and per-gene/per-cell molecule counts from the nanoranger output. Compute median segments per read and mean molecules per gene/cell, then compare against skera-based reference values (median 12 vs 15 segments/read; 141 vs 55 molecules/gene; r = 0.88 per gene and r = 0.97 per cell). Expect nanoranger to find fewer segments per read (median 12; range 0–37) than skera (median 15; range 0–16) while reporting consistently higher molecule counts that remain highly correlated, indicating the counts are comparable despite differing deconcatenation stringency.

## Related tools

- **nanoranger** (End-to-end pipeline for deconcatenating MAS-Seq reads and quantifying molecules per gene and per cell from long-read single-cell data) — github.com/mehdiborji/nanoranger
- **Pacific Biosciences** (Sequencing platform and data source (Revio MAS-Seq run; skera is PacBio's competing deconcatenation tool))
- **minimap2** (Spliced long-read aligner available for mapping Iso-Seq/cDNA reads against the transcriptome reference within long-read single-cell workflows) — https://github.com/lh3/minimap2

## Examples

```
git clone https://github.com/mehdiborji/nanoranger && run nanoranger workflow on DATA-Revio-PBMC-2 MAS-Seq reads with GENCODE v44 reference, then write nanoranger_vs_skera_comparison.csv
```

## Evaluation signals

- nanoranger_vs_skera_comparison.csv exists and is well-formed CSV with rows for both metrics (segments-per-read and molecules-per-gene)
- Reported skera reference values (12 vs 15 segments/read; 141 vs 55 molecules/gene) are present as benchmark columns
- Nanoranger median segments per read is expected near 12 (range 0–37) on PBMC MAS-Seq data, below skera's 15
- Per-gene molecule counts vs skera correlate strongly (r = 0.88) and per-cell counts even more strongly (r = 0.97), with nanoranger counts consistently higher (141 vs 55 per gene; 2580 vs 1650 per cell)

## Limitations

- Nanoranger detects consistently fewer segments per read (median 12 vs skera's 15) with a wider range (0–37 vs 0–16), so absolute segment counts are not directly comparable to skera's
- Nanoranger reports higher molecule counts per gene/cell than skera despite fewer segments, reflecting differing deconcatenation/quantification stringency rather than more input molecules
- Optimal nanoranger targets are loci near the 5' end of highly expressed genes with transcript lengths shorter than 4 kb; longer or lowly expressed transcripts are underrepresented

## Evidence

- [results] nanoranger found consistently fewer segments per read (median 12; range 0–37) than skera (median 15; range 0–16): [evidence span withheld — anchor status: not_found]
- [results] detected mean molecules per gene (141 vs. 55) and per cell (2580 vs. 1650) that were highly correlated: [evidence span withheld — anchor status: not_found]
- [methods] The input data can be obtained through sequencing of 10x Genomics whole-transcriptome cDNA libraries: "The input data can be obtained through sequencing of 10x Genomics whole-transcriptome cDNA libraries or amplicons obtained through targeted amplification"
- [readme] nanoranger is a processing tool for long-read single-cell transcriptomics: "*nanoranger* is a processing tool for long-read single-cell transcriptomics as described in our [Nature Communications](https://www.nature.com/articles/s41467-023-44137-7) paper"
- [abstract] we define the optimal targets for nanoranger, namely those loci close to the 5' end of highly expressed genes with transcript lengths shorter than 4 kB: "we define the optimal targets for nanoranger, namely those loci close to the 5' end of highly expressed genes with transcript lengths shorter than 4 kB"
