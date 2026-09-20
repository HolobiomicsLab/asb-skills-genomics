---
name: structural-variant-genotype-distance-stratification
description: Use when crossing pairs of strains where variant call sets (SNPs, SVs,
  rearrangements, CNVs) and an offspring fitness measure (e.g. spore viability) are
  available and you must rank which variant classes explain reproductive isolation.
when_to_use_negative:
- When the goal is heritability partitioning or GWAS of quantitative traits rather
  than cross-level viability-distance association
- When only one variant class is available — partial correlations controlling for
  the other class (SNPs vs. rearrangements) cannot be performed
- When SV calls are uncurated single-caller outputs — the study prioritized high specificity,
  accepting SVs detected by at least two callers and manually vetting alignments
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3053
tools:
- name: SURVIVOR
  role: Merges and compares multi-sample SV VCFs to generate the consensus catalogue
    whose per-pair unshared counts define the genetic distances
  repo: https://github.com/fritzsedlazeck/SURVIVOR
- name: Delly
  role: One of four SV inference packages whose initial predictions feed the consensus
    call set
- name: Lumpy
  role: One of four SV inference packages whose initial predictions feed the consensus
    call set
- name: Pindel
  role: One of four SV inference packages whose initial predictions feed the consensus
    call set
- name: cn.MOPs
  role: CNV inference package feeding the CNV distance class
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/ncomms14061
    title: Transient structural variations have strong effects on quantitative traits
      and reproductive isolation in fission yeast
  source_content_hash: sha256:45d3d5b35bc1c5d3b1e2df4db955569a3c251facf8c4ca34da394cdb79260b78
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/fritzsedlazeck/SURVIVOR
  tools:
  - Delly
  - Lumpy
  - Pindel
  - SURVIVOR
  - cn.MOPs
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3053
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# structural-variant-genotype-distance-stratification

## Summary

Quantifies parental genetic distance (number of unshared SNPs, all SVs, rearrangements only, or CNVs) between fission yeast strains and correlates each variant class with hybrid spore viability using Kendall tau and partial correlations, to identify which variant classes independently contribute to intrinsic reproductive isolation.

## When to use

Use when crossing pairs of strains where variant call sets (SNPs, SVs, rearrangements, CNVs) and an offspring fitness measure (e.g. spore viability) are available and you must rank which variant classes explain reproductive isolation.

## When NOT to use

- When the goal is heritability partitioning or GWAS of quantitative traits rather than cross-level viability-distance association
- When only one variant class is available — partial correlations controlling for the other class (SNPs vs. rearrangements) cannot be performed
- When SV calls are uncurated single-caller outputs — the study prioritized high specificity, accepting SVs detected by at least two callers and manually vetting alignments

## Inputs

- Curated per-strain variant call sets: SNPs, all SVs (deletions, duplications, inversions, translocations), rearrangements only, CNVs only
- Pairwise cross spore-viability measurements for hybrid offspring
- Optional: multi-sample VCFs from SV callers (Delly, Lumpy, Pindel, cn.MOPs) merged into a consensus with SURVIVOR

## Outputs

- Kendall tau and P values for each variant-class distance vs. spore viability (e.g. SNPs tau=-0.35 P=1.6e-4; SVs tau=-0.26; rearrangements tau=-0.36 P=2.0e-4; CNVs tau=-0.10 P=0.28)
- Partial correlation P values for each factor controlling for the other variant class
- Results table, e.g. genetic_distance_viability_kendall_correlations.tsv

## How to apply

Compute pairwise parental distances as the number of unshared variants per class (SNPs, all SVs, rearrangements only, CNVs only) from curated call sets — in this study a consensus catalogue produced with SURVIVOR followed by rigorous filtering and manual inspection. Compute Kendall tau rank correlations between each distance class and cross spore viability. Then estimate the independent influence of each factor using partial correlations, controlling for the other variant class. Interpret significance per class: in the source study, rearrangements (tau=-0.36, P=2.0e-4) and SNPs (tau=-0.35, P=1.6e-4) remained significant under partial correlation (P=0.04 and P=0.02), whereas CNVs did not (tau=-0.10, P=0.28). Write the results as a TSV table of tau and P values per variant class.

## Related tools

- **SURVIVOR** (Merges and compares multi-sample SV VCFs to generate the consensus catalogue whose per-pair unshared counts define the genetic distances) — https://github.com/fritzsedlazeck/SURVIVOR
- **Delly** (One of four SV inference packages whose initial predictions feed the consensus call set)
- **Lumpy** (One of four SV inference packages whose initial predictions feed the consensus call set)
- **Pindel** (One of four SV inference packages whose initial predictions feed the consensus call set)
- **cn.MOPs** (CNV inference package feeding the CNV distance class)

## Examples

```
from scipy.stats import kendalltau; tau, p = kendalltau(parental_unshared_snp_distance, spore_viability)
```

## Evaluation signals

- Sign and approximate magnitude of tau: rearrangements and SNP distances should give tau near -0.35 to -0.36 with P < 0.001; CNV distance should be near tau=-0.10 and non-significant (P=0.28)
- Partial correlation must leave both SNPs and rearrangements individually significant (P=0.04 and P=0.02) for concordance with the published interpretation
- Distances computed as counts of unshared variants per class should be symmetric across parent pairs and integer-valued
- All four variant classes (SNPs, all SVs, rearrangements, CNVs) appear as separate rows in the output TSV with tau and P columns
- SV distances must derive from the filtered consensus (at least two callers, low-complexity/LTR-filtered, manually vetted), not raw single-caller output

## Limitations

- CNV copy-number state is transient and only weakly correlated with nearby SNP variation (Spearman rho=0.22, P=0.041), so CNV distances may not reflect stable divergence
- The pipeline prioritized high specificity over sensitivity, so not all SVs were detected; distances may be underestimates
- Translocations and inversions were not re-examined because they are difficult to genotype accurately; only 45% of SVs are strongly linked to SNPs (r2>0.6), so SNP distances may not tag SV divergence well

## Evidence

- [abstract] Both SV-distance and SNP-distance inversely correlate with hybrid viability: "Both SV-distance (number of unshared SVs between parents) and SNP-distance inversely correlated with hybrid viability"
- [abstract] Partial correlation step: "we also estimated the influence of each factor alone using partial correlations."
- [discussion] Consensus SV curation prerequisite: "we applied a consensus approach to identify SVs (SURVIVOR), followed by rigorous filtering and manual inspection of all calls"
- [readme] SURVIVOR purpose from README: "SURVIVOR is a tool set for simulating/evaluating SVs, merging and comparing SVs within and among samples, and includes various methods to reformat or summarize SVs."
