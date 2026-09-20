---
name: umi-error-corrected-consensus-analysis
description: Use when ScNaUmi-seq 10x-linked Nanopore reads are available and you
  need UMI-guided error-corrected consensus molecules to call site-level sequence
  variation (SNPs/RNA editing) per molecule or per cell at defined genomic positions.
when_to_use_negative:
- Input reads have not been cell-barcode/UMI assigned and collapsed into consensus
  molecules (Sicelore steps 1-4b must run first; raw Nanopore reads are too noisy
  for per-position editing calls)
- The analysis goal is cell-by-cell SNV calling on high-coverage data where stricter
  molecule filters (MINRN > 0) are appropriate and permissive MINRN=0/MINQV=0 retention
  is unnecessary
- The target variants are short-read-only splice/junction events rather than sequence
  variation at defined genomic positions
edam_operation: http://edamontology.org/operation_3222
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: Sicelore-2.1.jar
  role: UMI-guided consensus molecule generation (step 4.b) and SNPMatrix per-molecule
    base counting at target SNP positions
  repo: https://github.com/ucagenomix/sicelore-2.1
- name: samtools
  role: Subsetting the molecules BAM to the locus of interest (e.g. chr3:80682936-80804791
    for Gria2)
  repo: https://github.com/samtools/samtools
- name: spoa
  role: Partial-order alignment consensus required for Sicelore step 4.b per-UMI consensus
    sequences
  repo: https://github.com/rvaser/spoa
- name: minimap2
  role: Mapping long reads against the reference genome in step 2 of the Sicelore
    workflow
  repo: https://github.com/lh3/minimap2
- name: Java
  role: Runtime required to execute Sicelore-2.1.jar (Java >= 13, e.g. -Xmx32g for
    SNPMatrix)
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-020-17800-6
    title: High throughput error corrected Nanopore single cell transcriptome sequencing
  source_content_hash: sha256:237b42034ccdfa1b09df395550069fd0378e05677f9ef5a18710ac411a7e4c62
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ucagenomix/sicelore-2.1
  tools:
  - Java
  - Sicelore-2.1.jar
  - Sicelore-2.1.jar (SNPMatrix pipeline)
  - minimap2
  - samtools
  - spoa
  edam_operation: http://edamontology.org/operation_3222
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5384615384615384
n_claims: 13
---

# umi-error-corrected-consensus-analysis

## Summary

Generates per-UMI consensus molecules from Nanopore long reads with Sicelore (step 4.b, using spoa POA consensus) and queries the error-corrected molecules BAM for sequence variation such as A-to-I RNA editing at specific loci, e.g. Gria2 Q/R and R/G sites in E18 mouse brain ScNaUmi-seq data.

## When to use

Use when ScNaUmi-seq 10x-linked Nanopore reads are available and you need UMI-guided error-corrected consensus molecules to call site-level sequence variation (SNPs/RNA editing) per molecule or per cell at defined genomic positions.

## When NOT to use

- Input reads have not been cell-barcode/UMI assigned and collapsed into consensus molecules (Sicelore steps 1-4b must run first; raw Nanopore reads are too noisy for per-position editing calls)
- The analysis goal is cell-by-cell SNV calling on high-coverage data where stricter molecule filters (MINRN > 0) are appropriate and permissive MINRN=0/MINQV=0 retention is unnecessary
- The target variants are short-read-only splice/junction events rather than sequence variation at defined genomic positions

## Inputs

- UMI/cellBC-tagged Nanopore long-read BAM after Sicelore steps 1-3
- Error-corrected consensus molecules BAM (molecules.GE.tags.bam from Sicelore step 4.b.5)
- Valid cell barcode CSV (ValidBarcodes.csv)
- SNP descriptor CSV listing target positions ('chromosome,position,strand,name') and optional pipe-separated multi-position association entries
- 10x Genomics barcode whitelist (e.g. 3M-february-2018.txt.gz)

## Outputs

- snp_matrix.txt with per-molecule base calls at the requested SNP positions
- snp_metrics.txt with per-position molecule counts
- Per-site editing fractions (edited-base molecules / total molecules) and joint two-site editing rates

## How to apply

Run the Sicelore step 4.b workflow (quickrun-2.1.step4b.sh pattern) to collapse reads per cellBC-UMI into consensus molecules (molecules.GE.tags.bam), using spoa for partial-order-alignment consensus; keep molecule filters permissive (MINRN=0, MINQV=0) to retain all molecules when quantifying editing rates rather than only high-confidence SNVs. Subset the molecules BAM to the locus of interest with samtools (e.g. 'samtools view molecules.GE.tags.bam chr3:80682936-80804791' for the Gria2 locus). Build a SNP descriptor CSV in 'chromosome,strand,name' format (e.g. Gria2_RG at chr3:80692286 minus strand, Gria2_QR at chr3:80706912), adding a pipe-separated two-position entry 'chr3,80692286|80706912' for conditional/two-site association rates. Run the Sicelore SNPMatrix task and parse snp_matrix.txt and snp_metrics.txt, computing per-site editing fraction as edited-base molecules over total molecules, including joint two-site counts to test whether editing at one site influences editing at the other.

## Related tools

- **Sicelore-2.1.jar** (UMI-guided consensus molecule generation (step 4.b) and SNPMatrix per-molecule base counting at target SNP positions) — https://github.com/ucagenomix/sicelore-2.1
- **samtools** (Subsetting the molecules BAM to the locus of interest (e.g. chr3:80682936-80804791 for Gria2)) — https://github.com/samtools/samtools
- **spoa** (Partial-order alignment consensus required for Sicelore step 4.b per-UMI consensus sequences) — https://github.com/rvaser/spoa
- **minimap2** (Mapping long reads against the reference genome in step 2 of the Sicelore workflow) — https://github.com/lh3/minimap2
- **Java** (Runtime required to execute Sicelore-2.1.jar (Java >= 13, e.g. -Xmx32g for SNPMatrix))

## Examples

```
samtools view molecules.GE.tags.bam chr3:80682936-80804791 > gria2.bam; java -jar -Xmx32g sicelore-2.1.jar SNPMatrix I=gria2.bam MINRN=0 MINQV=0 CSV=ValidBarcodes.csv SNP=snps.csv O=. PREFIX=snp
```

## Evaluation signals

- Recomputed editing fractions from snp_matrix.txt match reported E18 mouse brain values within a few percent (83.8% at the Gria2 Q/R site, 20.8% at the R/G site)
- Output tables snp_matrix.txt and snp_metrics.txt contain entries for chr3:80692286 and chr3:80706912
- Molecule filters were left permissive (MINRN=0 and MINQV=0) so total molecule denominators are not artificially reduced
- Joint two-site counts reproduce the conditional association: Q/R editing 89.1% vs 82% for R/G-edited vs unedited RNAs; R/G editing 25.1% vs 15.8% for Q/R-edited vs unedited RNAs
- SNP descriptor CSV parses correctly in the required 'chromosome,position,strand,name' format with the pipe-separated association entry

## Limitations

- Sicelore requires Java >= 13 and spoa for step 4.b consensus generation
- Barcode assignment accuracy depends on edit distance: edit distance 1 yields > 99.9% accuracy, while edit distance 2 yields 5-10% more assigned reads at the cost of lower accuracy
- Nanopore raw-read error rates make per-site calls unreliable without UMI-guided error correction; results should be validated against known reference values at the target positions

## Evidence

- [readme] SiCeLoRe is a suite of tools dedicated to cell barcode / UMI (unique molecular identifier) assignment and bioinformatics analysis of highly multiplexed single cell Nanopore or PacBIo long read sequencing data.: [evidence span withheld — anchor status: external_doc]
- [readme] Step 4b generates matrices from consensus sequences per UMI.: "option b: using consensus sequences per UMI"
- [methods] MINRN and MINQV control which molecules are kept for SNP calling.: "MINRN (required): Minimum read number to keep the molecule for SNP calling"
- [abstract] The paper reports UMI-guided error correction enabling high accuracy full-length sequences.: "UMI guided error correction allows to generate high accuracy full length sequence information"
- [methods] MINEVIDENCE filters low-support isoforms during consensus modeling.: "Transcripts isoforms showing an exon structure supported by less than MINEVIDENCE (default=5) UMIs are filtered out"
