---
name: nanopore-long-read-locus-subsetting
description: Use when you need locus-restricted RNA editing or SNV rates from ScNaUmi-seq
  error-corrected consensus molecules, e.g. measuring A-to-I editing at two linked
  sites like Gria2 Q/R and R/G and testing whether editing at one site influences
  the other.
when_to_use_negative:
- Input is raw Nanopore reads rather than UMI-corrected consensus molecules (molecules.GE.tags.bam);
  per-read base errors will swamp true editing signals
- The variant of interest spans transcripts across different chromosomes, since locus
  subsetting by a single samtools region assumes one contiguous range
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3512
tools:
- name: sicelore-2.1.jar
  role: SNPMatrix subcommand computes molecule-level base counts per SNP position
    and joint two-position association
  repo: https://github.com/ucagenomix/sicelore-2.1
- name: samtools
  role: Subsets the molecule BAM to the locus of interest by chromosomal region
  repo: https://github.com/samtools/samtools
- name: Java
  role: Runtime for sicelore-2.1.jar (requires Java >= 13)
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-020-17800-6
    title: High throughput error corrected Nanopore single cell transcriptome sequencing
  source_content_hash: sha256:4cb9ad1727f2f22f298ff96b9ebd92034d3e780773ec33697610d3521b245206
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ucagenomix/sicelore-2.1
  tools:
  - Java
  - Sicelore-2.1.jar (SNPMatrix pipeline)
  - samtools
  - sicelore-2.1.jar
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3512
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 10
---

# nanopore-long-read-locus-subsetting

## Summary

Extract error-corrected molecule BAM records (Sicelore step 4.b.5 consensus output) at a defined genomic locus with samtools, then run Sicelore SNPMatrix to compute per-site and joint two-site editing/SNV fractions from molecule counts.

## When to use

Use when you need locus-restricted RNA editing or SNV rates from ScNaUmi-seq error-corrected consensus molecules, e.g. measuring A-to-I editing at two linked sites like Gria2 Q/R and R/G and testing whether editing at one site influences the other.

## When NOT to use

- Input is raw Nanopore reads rather than UMI-corrected consensus molecules (molecules.GE.tags.bam); per-read base errors will swamp true editing signals
- The variant of interest spans transcripts across different chromosomes, since locus subsetting by a single samtools region assumes one contiguous range

## Inputs

- molecules.GE.tags.bam — error-corrected consensus molecule BAM from Sicelore step 4.b.5
- ValidBarcodes.csv — list of valid cell barcodes
- SNP descriptor CSV in 'chromosome,position,strand,name' format (e.g. chr3,80692286,-,Gria2_RG) including optional two-position association entries
- sicelore-2.1.jar, samtools, Java >= 13

## Outputs

- snp_matrix.txt — per-molecule base calls at each queried SNP position
- snp_metrics.txt — per-site editing/SNV fractions and joint two-site molecule counts

## How to apply

Subset the error-corrected molecules BAM produced by Sicelore step 4.b.5 (molecules.GE.tags.bam) to the locus of interest using samtools with a chromosomal range (e.g. chr3:80682936-80804791 for the Gria2 locus). Build a SNP descriptor CSV in 'chromosome,strand,position,name' format listing each site (Gria2_RG at chr3:80692286 minus strand, Gria2_QR at chr3:80706912) plus a pipe-delimited two-position entry 'chr3,80692286|80706912' to capture conditional joint editing. Run Sicelore-2.1 SNPMatrix with MINRN=0 and MINQV=0 so all molecules are retained, then parse snp_matrix.txt and snp_metrics.txt to compute each site's editing fraction as edited-base molecules divided by total molecules at that position, including joint two-site molecule counts. The two-position association entry is what makes conditional rates (edit-one-site-given-the-other) computable from a single run.

## Related tools

- **sicelore-2.1.jar** (SNPMatrix subcommand computes molecule-level base counts per SNP position and joint two-position association) — https://github.com/ucagenomix/sicelore-2.1
- **samtools** (Subsets the molecule BAM to the locus of interest by chromosomal region) — https://github.com/samtools/samtools
- **Java** (Runtime for sicelore-2.1.jar (requires Java >= 13))

## Examples

```
samtools view molecules.GE.tags.bam chr3:80682936-80804791 > gria2.bam; java -jar -Xmx32g sicelore-2.1.jar SNPMatrix I=gria2.bam MINRN=0 MINQV=0 CSV=ValidBarcodes.csv SNP=snps.csv O=. PREFIX=snp
```

## Evaluation signals

- snp_matrix.txt and snp_metrics.txt contain entries for both queried positions (e.g. chr3:80692286 and chr3:80706912)
- Recomputed per-site editing fractions match independent reported values within a few percent (validation: 83.8% at the Gria2 Q/R site and 20.8% at the R/G site for E18 mouse brain)
- Joint two-position molecule counts are non-zero and yield conditional rates consistent with published conditional findings (e.g. Q/R 89.1% vs 82%, R/G 25.1% vs 15.8%)
- MINRN=0 and MINQV=0 confirmed in the command so no molecules are silently dropped from the editing fraction denominators

## Limitations

- Editing rates depend on the quality of UMI-guided consensus correction; Nanopore raw read errors are only removed downstream of step 4.b.5
- Minus-strand positions must be declared with strand '-' in the SNP descriptor CSV or strand interpretation is wrong
- Default molecule filters (MINRN, MINQV) would drop low-coverage molecules if left at non-zero defaults; they must be explicitly set to 0 to retain all molecules

## Evidence

- [other] Subset the error-corrected molecules BAM to the Gria2 locus: [evidence span withheld — anchor status: not_found]
- [other] SNPMatrix command with default molecule filters: "Run 'java -jar -Xmx32g sicelore-2.1.jar SNPMatrix I=molecules.GE.tags.bam MINRN=0 MINQV=0 CSV=ValidBarcodes.csv SNP=snps.csv O=. PREFIX=snp'"
- [readme] SNP calling step in workflow: "Step 5 (optional) - [Single Nucleotide Variant calling cell by cell](#snp-calling)"
- [methods] MINRN/MINQV filter definition: "MINRN (required): Minimum read number to keep the molecule for SNP calling"
- [results] Editing rate finding at Gria2 sites: [evidence span withheld — anchor status: not_found]
