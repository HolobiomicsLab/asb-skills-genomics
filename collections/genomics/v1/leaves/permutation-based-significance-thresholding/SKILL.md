---
name: permutation-based-significance-thresholding
description: Use when running mixed-model GWAS across many variant-trait pairs (e.g.
  68 SVs, 139,396 SNPs, 22,058 indels vs. quantitative traits) and you need empirical,
  trait-specific significance thresholds instead of a generic genome-wide P-value
  cutoff.
when_to_use_negative:
- When testing a single pre-specified variant-trait hypothesis where simple correction
  suffices — the permutation null is designed for genome-wide, many-variant scans
  with correlated genotypes
- When phenotypes are not exchangeable across strains under the null (e.g. strong
  population structure not captured by the mixed model), since permutation breaks
  the intended null
- When the aim is SV discovery or genotyping (use SURVIVOR-based consensus calling/filtering),
  not association significance assessment
edam_operation: http://edamontology.org/operation_3196
edam_topics:
- http://edamontology.org/topic_0625
- http://edamontology.org/topic_3517
tools:
- name: LDAK
  role: Runs the mixed-model association tests on observed and permuted trait values
    to generate P-value distributions for threshold derivation
- name: SURVIVOR
  role: 'Upstream: merges and filters SV calls (consensus VCF) that supply the genotype
    matrix whose associations are thresholded'
  repo: https://github.com/fritzsedlazeck/SURVIVOR
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/ncomms14061
    title: Transient structural variations have strong effects on quantitative traits
      and reproductive isolation in fission yeast
  source_content_hash: sha256:dfe31d66846b9aaf4580dbeeadb39494298e5ab078de4d5551e14782efcc7708
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/fritzsedlazeck/SURVIVOR
  tools:
  - LDAK
  - SURVIVOR
  edam_operation: http://edamontology.org/operation_3196
  edam_topics:
  - http://edamontology.org/topic_0625
  - http://edamontology.org/topic_3517
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# permutation-based-significance-thresholding

## Summary

Establishes trait-specific GWAS significance cutoffs by permuting phenotype values and recomputing association statistics, controlling the family-wise error rate at 5%. Used in the S. pombe study to call significant SV-trait associations from mixed-model tests.

## When to use

Use when running mixed-model GWAS across many variant-trait pairs (e.g. 68 SVs, 139,396 SNPs, 22,058 indels vs. quantitative traits) and you need empirical, trait-specific significance thresholds instead of a generic genome-wide P-value cutoff.

## When NOT to use

- When testing a single pre-specified variant-trait hypothesis where simple correction suffices — the permutation null is designed for genome-wide, many-variant scans with correlated genotypes
- When phenotypes are not exchangeable across strains under the null (e.g. strong population structure not captured by the mixed model), since permutation breaks the intended null
- When the aim is SV discovery or genotyping (use SURVIVOR-based consensus calling/filtering), not association significance assessment

## Inputs

- SV genotype matrix restricted to 68 SVs with minor allele count ≥5 (VCF or numeric genotype matrix)
- SNP/indel genotype matrices (139,396 SNPs and 22,058 indels with MAC ≥5) if included in the joint model
- Phenotype table of quantitative traits measured across the S. pombe strain panel (e.g. 228 traits)
- Kinship/relationship matrix or covariates for the mixed model

## Outputs

- Trait-specific permutation significance thresholds (one per trait, at 5% FWER)
- Association results table (sv_trait_associations.tsv) with effect sizes and P-values per variant-trait pair
- Set of significant SV-trait associations exceeding the permutation threshold

## How to apply

For each quantitative trait, permute the trait values across strains while keeping the genotype matrix (e.g. the 68 SVs with minor allele count ≥5) fixed, and rerun the mixed-model association test (LDAK) on each permuted dataset. Collect the minimum P-values across variants within each permutation to build the empirical null distribution, then take the 5th percentile of these minima as the trait-specific threshold for a 5% family-wise error rate. Repeat per trait, since thresholds differ by trait heritability and distribution. Mark variant-trait pairs whose observed P-value falls below the permutation threshold as significant; in the source study this identified 19 SVs (15 duplications, five deletions, one translocation) with median effect size 14% (range 6–33%). Rationale: parametric extremes (e.g. Bonferroni over LD-structured variants) are miscalibrated for correlated SVs and SNPs, so empirical permutation preserves the correlation structure of the genotype data.

## Related tools

- **LDAK** (Runs the mixed-model association tests on observed and permuted trait values to generate P-value distributions for threshold derivation)
- **SURVIVOR** (Upstream: merges and filters SV calls (consensus VCF) that supply the genotype matrix whose associations are thresholded) — https://github.com/fritzsedlazeck/SURVIVOR

## Examples

```
ldak5.linux --pheno traits.txt --grm spombe_kinship --phenocol 3 --pvals sv_gwas --permutations 1000 --sv-genotypes sv_mac5.vcf
```

## Evaluation signals

- Each trait has its own computed threshold (thresholds differ across traits); a single global cutoff indicates the permutation step was skipped
- Number of significant SVs is in the expected range (19 SVs: 15 duplications, five deletions, one translocation) with median effect size 14% (range 6–33%)
- Permutation null distribution of minimum P-values is empirically derived; spot-check that ≤5% of permuted scans yield min-P below the reported threshold
- Output table contains effect sizes and P-values for all variant-trait pairs, not only significant hits
- Genotype matrix used in permutations matches the analysis set (MAC ≥5 filter applied: 68 SVs, 139,396 SNPs, 22,058 indels)

## Limitations

- Trait-specific thresholds must be recomputed per trait — they do not transfer between traits or datasets
- Permutation re-tests assume phenotype exchangeability; residual structure not modeled by the mixed model can inflate or deflate the null
- Failure to call SVs upstream (high-specificity pipeline) can bias the variant set and overestimate SNP contributions to trait variance, affecting what the thresholds are applied to
- SVs weakly linked to SNPs may be badly tagged, so SNP-only scans thresholded the same way will miss SV effects

## Evidence

- [abstract] Mixed model GWAS using 68 SVs, 139,396 SNPs and 22,058 indels with permutation-based thresholds: [evidence span withheld — anchor status: not_found]
- [abstract] 19 significant SVs with trait associations: "Nineteen SVs (28%) were significantly associated with traits (15 duplications, five deletions and one translocation)"
- [abstract] GWAS restricted to variants with MAC >= 5: [evidence span withheld — anchor status: not_found]
- [supplementary] All significant variants reported in supplementary GWAS results: "All variants significant in GWAS"
