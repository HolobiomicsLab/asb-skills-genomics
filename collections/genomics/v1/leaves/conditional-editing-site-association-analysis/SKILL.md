---
name: conditional-editing-site-association-analysis
description: Use when measuring A-to-I editing fractions at specific annotated sites
  from error-corrected Nanopore single-cell molecule BAMs and asking whether editing
  at one site changes the editing probability at another.
when_to_use_negative:
- Input is raw Nanopore reads without UMI-guided error-corrected consensus molecules
  (molecules.GE.tags.bam) — per-base editing calls will be dominated by sequencing
  errors
- The analysis goal is genome-wide de novo SNV/RNA-editing discovery rather than pre-specified
  site-level editing quantification and site-site association
- Data are short-read 3' scRNA-seq that lack full-length transcript coverage of both
  editing sites
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_3512
tools:
- name: Sicelore-2.1.jar (SNPMatrix)
  role: Counts molecules per allele at user-specified SNP/editing positions and two-position
    association entries from the tagged molecules BAM, producing snp_matrix.txt and
    snp_metrics.txt
  repo: https://github.com/ucagenomix/sicelore-2.1
- name: samtools
  role: Subsets the error-corrected molecules BAM to the genomic locus containing
    the editing sites (samtools view with a region string)
  repo: https://github.com/samtools/samtools
- name: Java
  role: Runtime required to execute Sicelore-2.1.jar (requires Java >= 13)
  repo: https://github.com/ucagenomix/sicelore-2.1
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-020-17800-6
    title: High throughput error corrected Nanopore single cell transcriptome sequencing
  source_content_hash: sha256:9c8dfd80fee93316df957ce4c35edfee18f89a476e824cf5a02f85d705d2aa35
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ucagenomix/sicelore-2.1
  tools:
  - Java
  - Sicelore-2.1.jar (SNPMatrix pipeline)
  - Sicelore-2.1.jar (SNPMatrix)
  - samtools
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_3512
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# conditional-editing-site-association-analysis

## Summary

Quantifies per-site RNA editing rates and tests whether editing at one site (e.g. Gria2 Q/R) is associated with editing at another (e.g. Gria2 R/G) by running Sicelore SNPMatrix on UMI error-corrected ScNaUmi-seq molecule BAMs and parsing joint two-site molecule counts.

## When to use

Use when measuring A-to-I editing fractions at specific annotated sites from error-corrected Nanopore single-cell molecule BAMs and asking whether editing at one site changes the editing probability at another.

## When NOT to use

- Input is raw Nanopore reads without UMI-guided error-corrected consensus molecules (molecules.GE.tags.bam) — per-base editing calls will be dominated by sequencing errors
- The analysis goal is genome-wide de novo SNV/RNA-editing discovery rather than pre-specified site-level editing quantification and site-site association
- Data are short-read 3' scRNA-seq that lack full-length transcript coverage of both editing sites

## Inputs

- Error-corrected molecules BAM with cellBC/UMI tags (molecules.GE.tags.bam from Sicelore step 4.b.5)
- ValidBarcodes.csv (valid cell barcode list)
- SNP descriptor CSV in 'chromosome,position,strand,name' format, including a pipe-joined two-position association entry
- samtools and the reference genome coordinates for the editing sites

## Outputs

- snp_matrix.txt (per-molecule base calls at each queried position, including joint two-site molecules)
- snp_metrics.txt (per-site and per-entry summary metrics)
- Per-site editing fractions (edited molecules / total molecules, e.g. 83.8% at Gria2 Q/R, 20.8% at R/G in E18 mouse brain)
- Conditional editing rates stratified by editing state at the partner site

## How to apply

Subset the Sicelore step 4.b.5 error-corrected molecules BAM (molecules.GE.tags.bam) to the locus of interest with samtools (e.g. chr3:80682936-80804791 for mouse Gria2). Build a SNP descriptor CSV in strict 'chromosome,strand,position,name' order, listing each site (Gria2_RG at chr3:80692286 on the minus strand, Gria2_QR at chr3:80706912) plus a pipe-joined two-position association entry 'chr3,80692286|80706912' so SNPMatrix emits joint molecule counts. Run sicelore-2.1.jar SNPMatrix with MINRN=0 and MINQV=0 so every molecule is retained for the editing-rate calculation (rationale: editing quantification should not discard low-support UMI consensus molecules). Parse snp_matrix.txt and snp_metrics.txt to compute editing fraction as edited-base molecules over total molecules at each position, then extract the joint two-site counts to derive conditional rates (e.g. Q/R 89.1% in R/G-edited vs 82% in R/G-unedited molecules; R/G 25.1% in Q/R-edited vs 15.8% in Q/R-unedited molecules), demonstrating reciprocal positive association. Correctly specify strand for minus-strand editing sites since the edited base call depends on strand orientation.

## Related tools

- **Sicelore-2.1.jar (SNPMatrix)** (Counts molecules per allele at user-specified SNP/editing positions and two-position association entries from the tagged molecules BAM, producing snp_matrix.txt and snp_metrics.txt) — https://github.com/ucagenomix/sicelore-2.1
- **samtools** (Subsets the error-corrected molecules BAM to the genomic locus containing the editing sites (samtools view with a region string)) — https://github.com/samtools/samtools
- **Java** (Runtime required to execute Sicelore-2.1.jar (requires Java >= 13)) — https://github.com/ucagenomix/sicelore-2.1

## Examples

```
samtools view molecules.GE.tags.bam chr3:80682936-80804791 > gria2.bam; java -jar -Xmx32g sicelore-2.1.jar SNPMatrix I=gria2.bam MINRN=0 MINQV=0 CSV=ValidBarcodes.csv SNP=snps.csv O=. PREFIX=snp
```

## Evaluation signals

- Recomputed editing fractions match reported E18 mouse brain values (83.8% at the Gria2 Q/R site, 20.8% at the R/G site) within a few percent
- Output tables contain entries for chr3:80692286 and chr3:80706912, confirming both descriptor positions were parsed
- The two-position association entry yields joint molecule counts from which conditional rates are derivable (Q/R: 89.1% vs 82%; R/G: 25.1% vs 15.8%)
- Setting MINRN=0 and MINQV=0 in the SNPMatrix invocation is verified, so no molecules were silently filtered before editing-rate computation
- Strand field in the SNP descriptor matches the reference orientation (Gria2_RG declared on the minus strand) and edited-base identity is consistent with that strand

## Limitations

- Requires UMI-guided error-corrected consensus molecules (Sicelore step 4.b.5 output); SNPMatrix on raw long reads would conflate Nanopore errors with true editing
- MINRN=0 and MINQV=0 retain all molecules, so very low-coverage sites may yield noisy editing fractions
- Sicelore requires Java >= 13 and depends on samtools/minimap2/spoa being on PATH; the snps.csv format must be exactly 'chromosome,position,strand,name' and the association entry uses pipe-joined positions

## Evidence

- [methods] SNPMatrix molecule filters and MINRN threshold: "MINRN (required): Minimum read number to keep the molecule for SNP calling"
- [methods] Sicelore workflow Step 5 is the SNP-calling entry point used for editing-rate analysis: "Step 5 (optional) - [Single Nucleotide Variant calling cell by cell](#snp-calling)"
- [readme] SiCeLoRe purpose and outputs supporting the molecule-level SNP matrix modality: [evidence span withheld — anchor status: external_doc]
- [readme] Sicelore-2.1 runtime prerequisites: "requires:

* Java >= 13
* [samtools](https://github.com/samtools/samtools)"
