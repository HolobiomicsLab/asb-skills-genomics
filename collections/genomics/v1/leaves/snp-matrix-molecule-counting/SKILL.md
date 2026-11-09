---
name: snp-matrix-molecule-counting
description: Use when you need per-molecule SNP or RNA-editing fractions from an error-corrected
  molecules BAM (Sicelore step 4.b consensus output), e.
when_to_use_negative:
- Input is raw, UMI-uncorrected Nanopore reads — per-read SNP calls are dominated
  by sequencing errors and require consensus building (step 4.b) first
- The goal is de novo variant discovery across an unknown locus — this skill counts
  molecules only at positions pre-listed in the SNP CSV
- Data is short-read 3' scRNA-seq without UMI-guided consensus molecules; SNPMatrix
  expects cellBC/UMI-annotated long-read molecule BAMs
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_0102
tools:
- name: Sicelore-2.1.jar
  role: Provides the SNPMatrix subcommand that counts per-molecule bases at the CSV-defined
    SNP positions from the molecules BAM
  repo: https://github.com/ucagenomix/sicelore-2.1
- name: samtools
  role: Subsets the error-corrected molecules BAM to the locus of interest before
    SNPMatrix (e.g. samtools view ... chr3:80682936-80804791)
  repo: https://github.com/samtools/samtools
- name: Java
  role: Runtime required to execute the Sicelore jar (>= 13) with a large heap (e.g.
    -Xmx32g)
  repo: https://github.com/ucagenomix/sicelore-2.1
- name: spoa
  role: Generates the UMI consensus molecules BAM (step 4.b) that serves as the upstream
    input for SNP counting
  repo: https://github.com/rvaser/spoa
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-020-17800-6
    title: High throughput error corrected Nanopore single cell transcriptome sequencing
  source_content_hash: sha256:b4c14912a560d258b78eb8d08cef90c37c0c97fd64054abb823500fc75383838
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ucagenomix/sicelore-2.1
  tools:
  - Java
  - Sicelore-2.1.jar
  - Sicelore-2.1.jar (SNPMatrix pipeline)
  - samtools
  - spoa
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# snp-matrix-molecule-counting

## Summary

Builds per-molecule SNP/RNA-editing count matrices from error-corrected ScNaUmi-seq consensus molecules using the Sicelore-2.1 SNPMatrix step (Step 5), enabling quantification of editing rates at predefined positions and joint (two-site) editing association, e.g. Gria2 Q/R and R/G A-to-I editing in E18 mouse brain.

## When to use

Use when you need per-molecule SNP or RNA-editing fractions from an error-corrected molecules BAM (Sicelore step 4.b consensus output), e.g. quantifying A-to-I editing at known sites like Gria2 chr3:80706912 (Q/R) and chr3:80692286 (R/G) and testing whether editing at one site is associated with editing at another.

## When NOT to use

- Input is raw, UMI-uncorrected Nanopore reads — per-read SNP calls are dominated by sequencing errors and require consensus building (step 4.b) first
- The goal is de novo variant discovery across an unknown locus — this skill counts molecules only at positions pre-listed in the SNP CSV
- Data is short-read 3' scRNA-seq without UMI-guided consensus molecules; SNPMatrix expects cellBC/UMI-annotated long-read molecule BAMs

## Inputs

- Error-corrected consensus molecules BAM (e.g. molecules.GE.tags.bam from Sicelore step 4.b.5)
- Valid cell barcode list CSV (e.g. ValidBarcodes.csv)
- SNP descriptor CSV with lines formatted 'chromosome,position,strand,name' plus optional 'chromosome,pos1|pos2' two-position association entries
- Locus coordinates (chromosome:start-end) for samtools subsetting

## Outputs

- snp_matrix.txt — per-SNP (and two-site association) molecule counts, cell by cell
- snp_metrics.txt — per-site metrics for computing editing fractions
- Per-site editing rate table (edited-base molecules / total molecules at each position) and conditional two-site editing rates

## How to apply

First subset the error-corrected molecules BAM produced by Sicelore step 4.b.5 (e.g. molecules.GE.tags.bam) to the locus of interest with samtools (e.g. chr3:80682936-80804791 for mouse Gria2). Build a SNP descriptor CSV in the strict 'chromosome,position,strand,name' format (strand matters, e.g. Gria2_RG on the minus strand at chr3:80692286 and Gria2_QR at chr3:80706912), adding an association entry with a '|' separator (chr3,80692286|80706912) to obtain joint two-site molecule counts. Run the Sicelore SNPMatrix subcommand with MINRN=0 and MINQV=0 to retain all molecules, or raise MINRN/MINQV to keep only molecules with sufficient read support and consensus quality. Parse the resulting snp_matrix.txt and snp_metrics.txt to compute per-site editing fraction as edited-base molecules over total molecules per position, including the joint two-site counts for conditional editing rates (e.g. Q/R editing 89.1% vs 82% for R/G-edited vs unedited RNAs).

## Related tools

- **Sicelore-2.1.jar** (Provides the SNPMatrix subcommand that counts per-molecule bases at the CSV-defined SNP positions from the molecules BAM) — https://github.com/ucagenomix/sicelore-2.1
- **samtools** (Subsets the error-corrected molecules BAM to the locus of interest before SNPMatrix (e.g. samtools view ... chr3:80682936-80804791)) — https://github.com/samtools/samtools
- **Java** (Runtime required to execute the Sicelore jar (>= 13) with a large heap (e.g. -Xmx32g)) — https://github.com/ucagenomix/sicelore-2.1
- **spoa** (Generates the UMI consensus molecules BAM (step 4.b) that serves as the upstream input for SNP counting) — https://github.com/rvaser/spoa

## Examples

```
java -jar -Xmx32g sicelore-2.1.jar SNPMatrix I=molecules.GE.tags.bam MINRN=0 MINQV=0 CSV=ValidBarcodes.csv SNP=snps.csv O=. PREFIX=snp
```

## Evaluation signals

- Recomputed editing fractions at chr3:80706912 (Q/R) and chr3:80692286 (R/G) match the reported E18 mouse brain values (83.8% and 20.8%) within a few percent
- Output tables snp_matrix.txt and snp_metrics.txt contain entries for chr3:80692286 and chr3:80706912 as defined in the SNP CSV
- Joint two-site counts satisfy the logical bound: molecules counted for the chr3,80692286|80706912 association do not exceed either single-site marginal count
- With MINRN=0 and MINQV=0, total molecules retained equal all molecules in the subset BAM (no molecule filtering applied)
- Reproduced conditional rates show positive association (Q/R editing 89.1% vs 82%, R/G editing 25.1% vs 15.8%)

## Limitations

- SNPMatrix counts only SNPs pre-listed in the descriptor CSV; no de novo variant discovery, and format errors (e.g. wrong strand flag) silently mislabel a site
- Accuracy depends on the upstream consensus step: molecules BAM must come from step 4.b (spoa-based UMI-guided error correction), otherwise Nanopore error rates inflate apparent editing
- Requires Java >= 13 and substantial memory (the reference invocation uses -Xmx32g); MINRN/MINQV choices directly affect which molecules enter the counts

## Evidence

- [methods] MINRN (required): Minimum read number to keep the molecule for SNP calling: "MINRN (required): Minimum read number to keep the molecule for SNP calling"
- [methods] MINRN / MINQV molecule filter for SNP calling: [evidence span withheld — anchor status: not_found]
- [readme] Step 5 (optional) - [Single Nucleotide Variant calling cell by cell](#snp-calling): "Step 5 (optional) - [Single Nucleotide Variant calling cell by cell](#snp-calling)"
- [other] java -jar -Xmx32g sicelore-2.1.jar SNPMatrix I=molecules.GE.tags.bam MINRN=0 MINQV=0 CSV=ValidBarcodes.csv SNP=snps.csv O=. PREFIX=snp: "java -jar -Xmx32g sicelore-2.1.jar SNPMatrix I=molecules.GE.tags.bam MINRN=0 MINQV=0 CSV=ValidBarcodes.csv SNP=snps.csv O=. PREFIX=snp"
- [other] Editing of one Gria2 site increases the probability that the other site is edited: [evidence span withheld — anchor status: not_found]
