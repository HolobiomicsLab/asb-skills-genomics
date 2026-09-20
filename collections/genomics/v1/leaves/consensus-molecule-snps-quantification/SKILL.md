---
name: consensus-molecule-snps-quantification
description: Use when you need cell- or cluster-level SNV/RNA-editing fractions (e.g.
  Gria2 R/G site) from ScNaUmi-seq UMI consensus molecules rather than error-prone
  raw Nanopore reads.
when_to_use_negative:
- When the long reads have not been UMI-consensus corrected (step 4.b not run) — raw
  Nanopore error rates would corrupt edit calls
- When cell barcodes have not been validated (no ValidBarcodes.csv / SelectValidCellBarcode
  MINUMI/ED0ED1RATIO filtering applied)
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3512
tools:
- name: Java
  role: Runtime for executing Sicelore's SNPMatrix jar (requires Java >= 13)
- name: samtools
  role: BAM handling for the UMI consensus molecule file fed into SNPMatrix
  repo: https://github.com/samtools/samtools
- name: Sicelore (SNPMatrix)
  role: Runs SNP calling cell by cell on UMI consensus molecules, producing the cell
    x position-base count matrix
  repo: https://github.com/ucagenomix/sicelore-2.1
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-020-17800-6
    title: High throughput error corrected Nanopore single cell transcriptome sequencing
  source_content_hash: sha256:fe3b1b0294051fcfab9521c9700456cdd92f81cea2410cab48f14ba5f41062e6
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/samtools/samtools
  tools:
  - Java
  - Sicelore (SNPMatrix)
  - samtools
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3512
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# consensus-molecule-snps-quantification

## Summary

Quantify single-nucleotide variants (e.g. A-to-I RNA editing) per cell cluster from Sicelore UMI-consensus molecule BAMs by running the SNPMatrix pipeline and computing edited-molecule fractions per position. This turns long-read Nanopore scRNA-seq data into per-cluster editing/SNV fraction estimates.

## When to use

Use when you need cell- or cluster-level SNV/RNA-editing fractions (e.g. Gria2 R/G site) from ScNaUmi-seq UMI consensus molecules rather than error-prone raw Nanopore reads.

## When NOT to use

- When the long reads have not been UMI-consensus corrected (step 4.b not run) — raw Nanopore error rates would corrupt edit calls
- When cell barcodes have not been validated (no ValidBarcodes.csv / SelectValidCellBarcode MINUMI/ED0ED1RATIO filtering applied)

## Inputs

- UMI consensus molecule BAM (molecules.GE.tags.bam) from Sicelore step 4.b.5
- SNP descriptor file (positions/strands of variants to quantify)
- ValidBarcodes.csv list of accepted cell barcodes
- Cell barcode to cluster annotation mapping (e.g. t-SNE cluster assignments)

## Outputs

- PREFIX_matrix.txt ([Cell] x [SNP position - base] molecule-count matrix)
- PREFIX_molinfos.txt (per-molecule SNP information)
- editing_fraction_per_cluster.tsv (edited/total molecule fraction per cluster group per SNP position)

## How to apply

Run Sicelore's Step 5 SNPMatrix on the UMI consensus molecule BAM (molecules.GE.tags.bam produced in workflow step 4.b.5), supplying the SNP descriptor file, the valid barcode list (ValidBarcodes.csv), and setting MINRN=0 and MINQV=0 so every molecule is retained regardless of read count or consensus quality. Parse the PREFIX_matrix.txt output, a [Cell] x [SNP position - base] molecule-count matrix, together with PREFIX_molinfos.txt for per-molecule SNP information. Map each cell barcode to its cluster annotation (e.g. t-SNE groups), sum edited-base versus total molecule counts per SNP position within each group, and compute editing fraction = edited molecules / total molecules covering the position. Rationale: UMI-guided consensus error correction removes Nanopore sequencing errors, so molecule-level counts reflect true RNA edits rather than read noise.

## Related tools

- **Java** (Runtime for executing Sicelore's SNPMatrix jar (requires Java >= 13))
- **samtools** (BAM handling for the UMI consensus molecule file fed into SNPMatrix) — https://github.com/samtools/samtools
- **Sicelore (SNPMatrix)** (Runs SNP calling cell by cell on UMI consensus molecules, producing the cell x position-base count matrix) — https://github.com/ucagenomix/sicelore-2.1

## Examples

```
java -jar Sicelore.jar -method 5 -b molecules.GE.tags.bam -snp SNPdescriptor.txt -cb ValidBarcodes.csv -MINRN 0 -MINQV 0
```

## Evaluation signals

- Editing fractions follow the reported maturation trend: ~9.3% in progenitor clusters (1,2,5), 14.3% in immature GABAergic (6), 16.4% in immature glutamatergic (3), 26.7% in mature GABAergic (7), and 26% in mature glutamatergic (4) neurons
- Editing fraction = edited molecules / total molecules covering the position is recomputable from PREFIX_matrix.txt and PREFIX_molinfos.txt sums per cluster group
- Every valid cell barcode in the matrix maps to a known cluster group (no unassigned barcodes when joining to the t-SNE annotation)
- Setting MINRN=0 and MINQV=0 yields non-empty molecule counts even for low-coverage positions, confirming all molecules were retained

## Limitations

- Lowly covered SNV positions produce noisy fractions even with MINRN=0/MINQV=0, since few molecules cover the site
- Variant calling is restricted to positions specified in the SNP descriptor file; de novo SNV discovery is not performed by this skill
- Interpretation depends on external cluster annotations (e.g. t-SNE groups) that are not produced by the Sicelore pipeline itself

## Evidence

- [methods] MINRN / MINQV molecule filter for SNP calling: "MINRN (required): Minimum read number to keep the molecule for SNP calling"
- [readme] SNP calling workflow step: "Step 5 (optional) - [Single Nucleotide Variant calling cell by cell](#snp-calling)"
- [abstract] Rationale for UMI consensus: "UMI guided error correction allows to generate high accuracy full length sequence information"
- [abstract] Editing at single-cell level: "ScNaUmi-seq allows defining splicing and SNVs (RNA editing) at a single cell level"
