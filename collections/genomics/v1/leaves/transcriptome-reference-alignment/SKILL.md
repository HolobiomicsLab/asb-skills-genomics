---
name: transcriptome-reference-alignment
description: Use when long-read single-cell cDNA or amplicon reads (ONT or PacBio)
  must be mapped to a reference transcriptome to call isoforms, fusions, SNVs, or
  per-gene/per-cell molecule counts.
when_to_use_negative:
- Input is already an aligned BAM/feature count matrix from an upstream pipeline (e.g.,
  cellranger output) — proceed directly to nanoranger.R downstream processing instead
- Analysis target is bulk (non-single-cell) transcriptomics where cell barcode/UMI
  demultiplexing is not needed
- The reference transcriptome does not cover the target locus (e.g., novel fusion
  partners or unannotated transcripts), where de novo or genome-guided approaches
  are preferable
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0080
tools:
- name: nanoranger
  role: End-to-end long-read single-cell processing workflow that takes transcriptome-aligned
    reads and produces SNV, fusion, isoform, and CAR/TCR calls
  repo: github.com/mehdiborji/nanoranger
- name: minimap2
  role: Splice-aware alignment of long RNA reads (PacBio Iso-Seq/Kinnex, ONT cDNA/direct
    RNA) against the reference transcriptome using its splice presets
  repo: https://github.com/lh3/minimap2
- name: STAR
  role: Alternative splice-aware transcriptome aligner used within the nanoranger
    ecosystem for reference-guided mapping
  repo: https://github.com/alexdobin/STAR
- name: nanoranger.R
  role: R downstream package that reads nanoranger output and extracts SNVs, indels,
    fusions/CARs, and isoforms from aligned amplicons; collapses UMIs with starcode
    within Levenshtein distance 3
  repo: liviuspenter/nanoranger.R
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-023-44137-7
    title: Integrative genotyping of cancer and immune phenotypes by long-read sequencing
  source_content_hash: sha256:269202e2f3197e7a0832c37de196ffef905887383761670187c8762e036b3dc7
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Pacific Biosciences
  - STAR
  - minimap2
  - nanoranger
  - nanoranger.R
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# transcriptome-reference-alignment

## Summary

Aligns long-read single-cell cDNA reads (from 10x Genomics whole-transcriptome libraries sequenced on ONT or Pacific Biosciences devices) against a transcriptome reference such as GENCODE v44, using splice-aware aligners like minimap2 or STAR, as done in the nanoranger pipeline.

## When to use

Use when long-read single-cell cDNA or amplicon reads (ONT or PacBio) must be mapped to a reference transcriptome to call isoforms, fusions, SNVs, or per-gene/per-cell molecule counts.

## When NOT to use

- Input is already an aligned BAM/feature count matrix from an upstream pipeline (e.g., cellranger output) — proceed directly to nanoranger.R downstream processing instead
- Analysis target is bulk (non-single-cell) transcriptomics where cell barcode/UMI demultiplexing is not needed
- The reference transcriptome does not cover the target locus (e.g., novel fusion partners or unannotated transcripts), where de novo or genome-guided approaches are preferable

## Inputs

- Long-read single-cell cDNA reads (FASTQ/BAM) from ONT or PacBio sequencing of 10x Genomics whole-transcriptome cDNA libraries
- Amplicon reads from targeted amplification of genes of interest
- Reference transcriptome annotation (e.g., GENCODE v44 FASTA/GTF)

## Outputs

- Aligned BAM/SAM (or PAF) files with 10x CB/UMI tags preserved
- Per-read gene/isoform assignments
- Per-gene and per-cell molecule count matrices for downstream nanoranger.R analysis

## How to apply

Start from intermediate single-cell cDNA libraries produced by sequencing 10x Genomics whole-transcriptome libraries or targeted-amplicon PCR products on Oxford Nanopore or Pacific Biosciences devices. Supply a curated transcriptome reference (e.g., GENCODE v44) as the alignment target, choosing a splice-aware aligner appropriate to read type — minimap2's splice presets for long noisy RNA reads (e.g., map-hifi/splice:hq for PacBio HiFi/Kinnex, map-ont for noisy ONT) or STAR for higher-accuracy reads. Keep cell barcodes and UMIs (10x Genomics CB/UMI tags) intact through alignment so downstream nanoranger.R can collapse UMIs with starcode (Levenshtein distance ≤ 3) and assign molecules to cells. Rationale: alignment against the transcriptome — rather than a de novo assembly — anchors reads to annotated genes/isoforms, enabling detection of lineage-defining features such as SNVs, fusion genes, isoforms, and CAR/TCR sequences.

## Related tools

- **nanoranger** (End-to-end long-read single-cell processing workflow that takes transcriptome-aligned reads and produces SNV, fusion, isoform, and CAR/TCR calls) — github.com/mehdiborji/nanoranger
- **minimap2** (Splice-aware alignment of long RNA reads (PacBio Iso-Seq/Kinnex, ONT cDNA/direct RNA) against the reference transcriptome using its splice presets) — https://github.com/lh3/minimap2
- **STAR** (Alternative splice-aware transcriptome aligner used within the nanoranger ecosystem for reference-guided mapping) — https://github.com/alexdobin/STAR
- **nanoranger.R** (R downstream package that reads nanoranger output and extracts SNVs, indels, fusions/CARs, and isoforms from aligned amplicons; collapses UMIs with starcode within Levenshtein distance 3) — liviuspenter/nanoranger.R

## Examples

```
./minimap2 -ax splice:hq -uf gencode_v44_transcripts.fa pbmc_masiseq.fq.gz > aln.sam
```

## Evaluation signals

- Alignment rate against the GENCODE reference is high and reads retain 10x CB and UMI tags in the BAM (e.g., via samtools stats)
- Splice-aware mapping produces canonical splice junctions consistent with the annotation; unexpected junctions are flagged for fusion/isoform review
- UMI collapsing with starcode (Levenshtein distance ≤ 3) yields a knee plot with a clear cell-barcode cutoff and plausible per-cell molecule counts
- Molecule counts per gene and per cell are highly correlated with an orthogonal short-read quantification (in the MAS-ISO-seq benchmark, nanoranger vs skera gave r = 0.88 per gene and r = 0.97 per cell)
- Called isoforms/fusions are reproducible across replicates and consistent with known annotation (e.g., CD45RA isoforms that CITE-seq antibodies failed to resolve)

## Limitations

- The genotyping ceiling is set by low, incomplete transcript representation: long or lowly expressed transcripts may be absent and cannot be recovered merely by amplicon optimization or deeper sequencing
- Current high-throughput single-cell chemistries do not provide truly full-length cDNA, limiting complete isoform coverage at the 3' end
- Alignment accuracy depends on the platform error profile — minimap2 presets must match the input type (map-ont vs map-hifi/splice:hq), and ONT error can obscure SNV calling without UMI consensus collapsing

## Evidence

- [abstract] nanoranger workflow from intermediate cDNA libraries: "we devise a flexible long-read sequencing workflow and analysis pipeline, termed nanoranger, that starts from intermediate single-cell cDNA libraries"
- [intro] target genes from 10x scRNA-seq cDNA libraries: "amplify target genes from 5′-anchored 10x Genomics scRNA-seq whole-transcriptome cDNA libraries"
- [readme] nanoranger input data modalities: "The input data can be obtained through sequencing of 10x Genomics whole-transcriptome cDNA libraries or amplicons obtained through targeted amplification, with Oxford Nanopore Technologies (ONT) or"
- [readme] minimap2 PacBio Iso-Seq splice alignment preset: [evidence span withheld — anchor status: not_found]
- [readme] UMI collapsing strategy for 10x libraries: [evidence span withheld — anchor status: external_doc]
