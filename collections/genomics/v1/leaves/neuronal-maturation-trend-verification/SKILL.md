---
name: neuronal-maturation-trend-verification
description: Use when recomputing or validating RNA-editing (SNV) fractions per cell
  cluster from a long-read single-cell dataset, e.g. testing whether Gria2 R/G editing
  rises from progenitor to mature neuron clusters.
when_to_use_negative:
- Input is bulk or short-read scRNA-seq data without full-length long-read molecules,
  so UMI-consensus molecule counts at the editing site are unavailable
- The goal is de novo SNV discovery rather than verifying an already-reported per-cluster
  editing fraction trend from existing cluster assignments
- Cell barcodes have not been assigned to t-SNE clusters, so grouping into progenitor/immature/mature
  populations is not possible
edam_operation: http://edamontology.org/operation_3229
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: Sicelore
  role: SNPMatrix pipeline producing the per-cell molecule-count matrix at SNV/editing
    positions from the UMI consensus BAM
  repo: https://github.com/ucagenomix/sicelore-2.1
- name: samtools
  role: BAM manipulation and indexing of molecules.GE.tags.bam prior to SNPMatrix
  repo: https://github.com/samtools/samtools
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-020-17800-6
    title: High throughput error corrected Nanopore single cell transcriptome sequencing
  source_content_hash: sha256:c3f443514dcc6f868755f91a80a94a80442d3e3b316a5bbe03d3eedc16fa2d61
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ucagenomix/sicelore-2.1
  tools:
  - Java
  - Sicelore
  - samtools
  edam_operation: http://edamontology.org/operation_3229
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# neuronal-maturation-trend-verification

## Summary

Verifies the reported increase of A-to-I RNA editing (Gria2 R/G site) across neuronal maturation stages by recomputing per-cluster editing fractions from ScNaUmi-seq molecule-level SNP counts, using the Sicelore SNPMatrix output [Cell] x [SNP position - base] molecule-count matrix.

## When to use

Use when recomputing or validating RNA-editing (SNV) fractions per cell cluster from a long-read single-cell dataset, e.g. testing whether Gria2 R/G editing rises from progenitor to mature neuron clusters.

## When NOT to use

- Input is bulk or short-read scRNA-seq data without full-length long-read molecules, so UMI-consensus molecule counts at the editing site are unavailable
- The goal is de novo SNV discovery rather than verifying an already-reported per-cluster editing fraction trend from existing cluster assignments
- Cell barcodes have not been assigned to t-SNE clusters, so grouping into progenitor/immature/mature populations is not possible

## Inputs

- molecules.GE.tags.bam — UMI consensus molecule BAM produced by Sicelore step 4.b.5
- SNP descriptor file listing the Gria2 R/G editing position
- ValidBarcodes.csv — list of valid cell barcodes
- t-SNE cluster assignment per cell barcode (clusters 1,2,3,4,5,6,7)

## Outputs

- PREFIX_matrix.txt — [Cell] x [SNP position - base] molecule-count matrix
- PREFIX_molinfos.txt — per-molecule SNP information
- editing_fraction_per_cluster.tsv — recomputed editing fractions per cluster group

## How to apply

Run the Sicelore SNPMatrix pipeline (Step 5 SNV calling) on the UMI consensus molecule BAM (molecules.GE.tags.bam from step 4.b.5) with the SNP descriptor file, the valid barcode list (ValidBarcodes.csv), and MINRN=0 / MINQV=0 so every molecule covering the editing site is retained rather than filtered. Parse the PREFIX_matrix.txt [Cell] x [SNP position - base] molecule-count matrix together with PREFIX_molinfos.txt per-molecule information. Map each cell barcode to its t-SNE cluster group (progenitors 1,2,5; immature GABAergic 6 and glutamatergic 3; mature GABAergic 7 and glutamatergic 4), then sum edited-base versus total molecule counts at the R/G position per group. Compute editing fraction as edited molecules divided by total molecules covering the position, and compare the recomputed fractions against the reported trend of 9.3% (progenitor), 14.3% (immature GABAergic), 16.4% (immature glutamatergic), 26.7% (mature GABAergic) and 26% (mature glutamatergic).

## Related tools

- **Sicelore** (SNPMatrix pipeline producing the per-cell molecule-count matrix at SNV/editing positions from the UMI consensus BAM) — https://github.com/ucagenomix/sicelore-2.1
- **samtools** (BAM manipulation and indexing of molecules.GE.tags.bam prior to SNPMatrix) — https://github.com/samtools/samtools

## Examples

```
java -jar Sicelore.jar SnpMatrix -i molecules.GE.tags.bam -b ValidBarcodes.csv -s snps.txt MINRN=0 MINQV=0
```

## Evaluation signals

- Recomputed editing fractions increase monotonically with maturation: progenitor clusters (1,2,5) < immature clusters (3,6) < mature clusters (4,7)
- Recomputed per-group fractions approximate the reported values: 9.3%, 14.3%, 16.4%, 26.7%, 26%
- Edited + unedited molecule counts at the R/G position per group match total molecules covering the site in PREFIX_matrix.txt (MINRN=0/MINQV=0 retains all molecules)
- editing_fraction_per_cluster.tsv contains one fraction per cluster group and only barcodes present in ValidBarcodes.csv

## Limitations

- Nanopore sequencing errors can misassign UMIs (about one-third mis-assigned without correction); molecule consensus correction mitigates this but residual errors perturb base counts at the editing site
- Low molecule counts at the R/G site within small clusters make editing-fraction estimates noisy
- The maturation trend depends on prior t-SNE cluster assignments (1,2,5 progenitor; 3,6 immature; 4,7 mature); misclustered cells shift the per-group fractions

## Evidence

- [methods] SNP calling retains all molecules when filters are disabled: "MINRN (required): Minimum read number to keep the molecule for SNP calling"
- [readme] Sicelore workflow includes optional per-cell SNV calling: "Step 5 (optional) - Single Nucleotide Variant calling cell by cell"
- [abstract] ScNaUmi-seq enables single-cell RNA editing detection: "ScNaUmi-seq allows defining splicing and SNVs (RNA editing) at a single cell level"
