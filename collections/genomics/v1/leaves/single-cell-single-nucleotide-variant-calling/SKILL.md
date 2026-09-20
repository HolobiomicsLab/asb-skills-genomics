---
name: single-cell-single-nucleotide-variant-calling
description: Use when you need per-cell SNV or RNA-editing quantification from full-length
  Nanopore scRNA-seq data, e.g. computing edited vs total UMI molecule counts per
  cell cluster for a known SNP position.
when_to_use_negative:
- You only need bulk SNV calling from genomic data — Sicelore SNPMatrix operates on
  single-cell transcriptome molecules with 10x barcodes/UMIs, not whole-genome reads
- Short-read scRNA-seq without full-length transcript coverage — the editing site
  may not be captured near read ends after Illumina short-read sequencing
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_2885
- http://edamontology.org/topic_3308
tools:
- name: Sicelore
  role: Runs the SNPMatrix pipeline (Step 5) for SNV calling cell by cell from molecule
    BAMs
  repo: https://github.com/ucagenomix/sicelore
- name: samtools
  role: Required dependency for BAM handling in the Sicelore workflow
  repo: https://github.com/samtools/samtools
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-020-17800-6
    title: High throughput error corrected Nanopore single cell transcriptome sequencing
  source_content_hash: sha256:28f11941104ae27763558930940aae4399ced83c0fe0302b69dac8dbc0acbdff
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ucagenomix/sicelore
  tools:
  - Java
  - Sicelore
  - samtools
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_2885
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# single-cell-single-nucleotide-variant-calling

## Summary

Call single nucleotide variants and RNA editing events (e.g. Gria2 R/G site A-to-I editing) cell by cell from Nanopore single-cell transcriptomes using the Sicelore SNPMatrix pipeline on UMI error-corrected molecule BAMs.

## When to use

Use when you need per-cell SNV or RNA-editing quantification from full-length Nanopore scRNA-seq data, e.g. computing edited vs total UMI molecule counts per cell cluster for a known SNP position.

## When NOT to use

- You only need bulk SNV calling from genomic data — Sicelore SNPMatrix operates on single-cell transcriptome molecules with 10x barcodes/UMIs, not whole-genome reads
- Short-read scRNA-seq without full-length transcript coverage — the editing site may not be captured near read ends after Illumina short-read sequencing

## Inputs

- UMI consensus molecule BAM (molecules.GE.tags.bam from Sicelore step 4.b.5)
- SNP descriptor file listing positions of interest (e.g. Gria2 R/G editing site)
- Valid cell barcode list (ValidBarcodes.csv) from SelectValidCellBarcode
- Cell-to-cluster annotation (t-SNE cluster assignments per cell barcode)

## Outputs

- PREFIX_matrix.txt: [Cell] x [SNP position - base] molecule-count matrix
- PREFIX_molinfos.txt: per-molecule SNP information
- editing_fraction_per_cluster.tsv: per-cluster edited/total molecule fractions

## How to apply

Run the Sicelore SNPMatrix pipeline (Java >= 13, samtools) on the UMI consensus molecule BAM (molecules.GE.tags.bam produced in step 4.b.5), supplying a SNP descriptor file and a valid barcode list (ValidBarcodes.csv). Set MINRN=0 and MINQV=0 to retain all molecules when computing editing fractions. Parse the PREFIX_matrix.txt output, a [Cell] x [SNP position - base] molecule-count matrix, together with PREFIX_molinfos.txt for per-molecule SNP information. Map each cell barcode to its cluster annotation (e.g. t-SNE clusters: progenitor 1,2,5; immature 3,6; mature neurons 4,7), sum edited-base versus total molecule counts per position, and compute editing fraction as edited molecules divided by total molecules covering the position.

## Related tools

- **Sicelore** (Runs the SNPMatrix pipeline (Step 5) for SNV calling cell by cell from molecule BAMs) — https://github.com/ucagenomix/sicelore
- **samtools** (Required dependency for BAM handling in the Sicelore workflow) — https://github.com/samtools/samtools

## Examples

```
java -jar Sicelore.jar SNPMatrix -i molecules.GE.tags.bam --snps SNPdescriptor.txt --cellBCs ValidBarcodes.csv --MINRN 0 --MINQV 0
```

## Evaluation signals

- Editing fractions show the expected maturation trend: ~9.3% in progenitor clusters (1,2,5), ~14.3% immature GABAergic (6), ~16.4% immature glutamatergic (3), ~26.7% and ~26% in mature neuron clusters (7, 4)
- Matrix is a [Cell] x [SNP position - base] molecule-count table keyed by the barcodes in ValidBarcodes.csv
- MINRN and MINQV are explicitly set to 0 so no molecules are silently filtered from the count matrix
- Every edited count can be traced back to molecule records in PREFIX_molinfos.txt

## Limitations

- Nanopore sequencing errors require UMI-guided consensus correction; at least one-third of UMIs were reported mis-assigned without such error correction
- SNP calling (Step 5) is an optional step and default MINRN filtering will drop molecules below the read-number threshold unless overridden
- Editing fraction denominators depend on molecules actually covering the SNP position, so lowly expressed loci give noisy fractions per cell

## Evidence

- [methods] MINRN (required): Minimum read number to keep the molecule for SNP calling: "MINRN (required): Minimum read number to keep the molecule for SNP calling"
- [abstract] ScNaUmi-seq allows defining splicing and SNVs (RNA editing) at a single cell level: "ScNaUmi-seq allows defining splicing and SNVs (RNA editing) at a single cell level"
- [readme] Step 5 (optional) - Single Nucleotide Variant calling cell by cell: "Step 5 (optional) - [Single Nucleotide Variant calling cell by cell](#snp-calling)"
- [readme] production of isoforms / junctions / SNPs x cells count matrices for new modalities: [evidence span withheld — anchor status: external_doc]
