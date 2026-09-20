---
name: sv-genotype-phenotype-mapping
description: Use when a genotype matrix of SVs (duplications, deletions, translocations)
  and a table of quantitative trait measurements across a strain panel are both available
  and the goal is to map SVs to trait variation.
when_to_use_negative:
- SV calls have not been filtered/validated yet (calls detected by fewer than two
  callers, breakpoints overlapping low-complexity regions or annotated LTRs) — the
  input catalogue should already be curated
- The question is variance-partitioning of trait heritability across variant classes
  rather than per-SV association testing (use mixed-model heritability estimation
  instead)
- Traits are binary or survival-type rather than quantitative; the permutation-threshold
  mixed-model design described here targets quantitative traits
edam_operation: http://edamontology.org/operation_3280
edam_topics:
- http://edamontology.org/topic_3517
tools:
- name: LDAK
  role: Mixed-model association testing of each SV-trait pair and estimation of additive
    effects
- name: SURVIVOR
  role: Produces the consensus, filtered SV catalogue whose genotype matrix feeds
    the GWAS
  repo: https://github.com/fritzsedlazeck/SURVIVOR
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/ncomms14061
    title: Transient structural variations have strong effects on quantitative traits
      and reproductive isolation in fission yeast
  source_content_hash: sha256:cffdaef5986ae8526addec6868e4a51286b1a947d8065c38d72e5d403db6c7a2
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/fritzsedlazeck/SURVIVOR
  tools:
  - LDAK
  - SURVIVOR
  edam_operation: http://edamontology.org/operation_3280
  edam_topics:
  - http://edamontology.org/topic_3517
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# sv-genotype-phenotype-mapping

## Summary

Test structural variations (SVs) for association with quantitative traits using mixed-model GWAS with permutation-derived significance thresholds, as done in the S. pombe SV study where 19 of 68 SVs were trait-associated.

## When to use

Use when a genotype matrix of SVs (duplications, deletions, translocations) and a table of quantitative trait measurements across a strain panel are both available and the goal is to map SVs to trait variation.

## When NOT to use

- SV calls have not been filtered/validated yet (calls detected by fewer than two callers, breakpoints overlapping low-complexity regions or annotated LTRs) — the input catalogue should already be curated
- The question is variance-partitioning of trait heritability across variant classes rather than per-SV association testing (use mixed-model heritability estimation instead)
- Traits are binary or survival-type rather than quantitative; the permutation-threshold mixed-model design described here targets quantitative traits

## Inputs

- SV genotype matrix for 68 SVs (duplications, deletions, translocations, inversions) with minor allele counts ≥5 across the strain panel
- Phenotype table of quantitative traits measured across the S. pombe strain panel (up to 228 traits)
- SNP genotypes (139,396 sites with MAC ≥5) and indel genotypes (22,058 sites with MAC ≥5)
- Population structure / kinship information for the mixed model

## Outputs

- sv_trait_associations.tsv containing per-SV-trait effect sizes and P-values
- List of significant SV-trait associations passing trait-specific 5% family-wise error permutation thresholds
- Trait-specific significance threshold values from permutation of trait values

## How to apply

Restrict the SV genotype matrix to the 68 SVs with minor allele counts ≥5 and match it to the phenotype table of quantitative traits measured across the S. pombe strain panel. For each SV-trait pair, run a mixed-model association test in LDAK to estimate the additive effect of the SV on the trait. Compute trait-specific significance thresholds for 5% family-wise error rates by permuting trait values, and mark SV-trait associations exceeding the permutation threshold as significant. Include the corresponding SNPs (139,396) and indels (22,058) with MAC ≥5 in the association framework so SV effects are tested alongside other variant classes. Write the full results table, including effect sizes and P-values, to a TSV (sv_trait_associations.tsv); in the source study the expected outcome was 19 significant SVs (15 duplications, five deletions, one translocation) with median effect size 14% (range 6–33%).

## Related tools

- **LDAK** (Mixed-model association testing of each SV-trait pair and estimation of additive effects)
- **SURVIVOR** (Produces the consensus, filtered SV catalogue whose genotype matrix feeds the GWAS) — https://github.com/fritzsedlazeck/SURVIVOR

## Evaluation signals

- Only SVs with minor allele count ≥5 enter the test set — the input matrix should contain exactly 68 SVs
- Each trait has its own permutation-derived threshold at 5% family-wise error rate, not a single genome-wide cutoff
- Expected result scale: 19 significant SVs (15 duplications, five deletions, one translocation), i.e. 28% of tested SVs
- Reported effect sizes should fall in the range 6–33% with a median near 14%
- Output file sv_trait_associations.tsv must contain effect sizes and P-values for every SV-trait pair tested

## Limitations

- SVs are often weakly linked to nearby SNPs (45% have r2 ≤ 0.6) and may be badly tagged by SNPs, so omitting SVs from GWAS can overestimate SNP effects and contribute to missing heritability
- Prioritizing specificity over sensitivity means some true SVs are missing from the catalogue and thus never tested
- Translocations and inversions are harder to genotype accurately; the study did not re-examine their genotypes

## Evidence

- [supplementary] Mixed model GWAS using 68 SVs, 139,396 SNPs and 22,058 indels with permutation-based thresholds: [evidence span withheld — anchor status: not_found]
- [abstract] 19 SVs significantly associated with traits in GWAS: "Nineteen SVs (28%) were significantly associated with traits (15 duplications, five deletions and one translocation)"
- [abstract] Permutation thresholds for trait-specific significance: "Trait-specific significance thresholds for 5% family-wise error rates were computed via"
- [discussion] SVs weakly linked to SNPs are badly tagged in GWAS: [evidence span withheld — anchor status: not_found]
- [abstract] GWAS restricted to SVs with minor allele count ≥5: "using all 68 SVs with minor allele counts ⩾5"
