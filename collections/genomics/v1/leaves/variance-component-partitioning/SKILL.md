---
name: variance-component-partitioning
description: Use when you have quantitative trait values plus genotype matrices from
  multiple variant classes (e.g. SNPs, indels, CNVs, rearrangements) and need the
  fraction of trait variance explained by each class rather than per-variant associations.
when_to_use_negative:
- The goal is SV discovery, genotyping or filtering of structural-variant calls (use
  the SURVIVOR consensus-calling pipeline instead — this skill starts from already-called,
  filtered genotypes).
- The goal is testing individual variant-trait associations with permutation-based
  significance thresholds; run the mixed-model GWAS step instead of (or in addition
  to) variance partitioning.
- Trait phenotypes are binary/case-control or survival-type outcomes where linear
  mixed-model heritability on normalized quantitative values is not appropriate.
edam_topics:
- http://edamontology.org/topic_0625
tools:
- name: LDAK
  role: Constructs the SNP-only and composite (SNP/CNV/rearrangement) kinship matrices
    and runs the mixed-model heritability estimation per trait
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/ncomms14061
    title: Transient structural variations have strong effects on quantitative traits
      and reproductive isolation in fission yeast
  source_content_hash: sha256:dc36f7051bf7614f57b7f922f8267abd043960af670d3fc03801100850d31a6e
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - LDAK
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0625
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# variance-component-partitioning

## Summary

Estimate narrow-sense heritability of quantitative traits and partition trait variance across variant classes (SNPs, CNVs, rearrangements) using LDAK mixed models with separate kinship matrices per variant class, so that contributions of structural variants beyond SNPs can be quantified.

## When to use

Use when you have quantitative trait values plus genotype matrices from multiple variant classes (e.g. SNPs, indels, CNVs, rearrangements) and need the fraction of trait variance explained by each class rather than per-variant associations.

## When NOT to use

- The goal is SV discovery, genotyping or filtering of structural-variant calls (use the SURVIVOR consensus-calling pipeline instead — this skill starts from already-called, filtered genotypes).
- The goal is testing individual variant-trait associations with permutation-based significance thresholds; run the mixed-model GWAS step instead of (or in addition to) variance partitioning.
- Trait phenotypes are binary/case-control or survival-type outcomes where linear mixed-model heritability on normalized quantitative values is not appropriate.

## Inputs

- SNP/indel genotype matrix (139,396 SNPs and 22,058 indels, minor allele count >=5)
- SV genotype matrix for 68 structural variants (deletions, duplications, inversions, translocations) with minor allele count >=5
- Normalized quantitative trait values for 228 traits across the strain panel
- Per-variant-class kinship matrices (SNP-only GRM; separate GRMs for SNPs, CNVs, rearrangements) built in LDAK

## Outputs

- Per-trait SNP-only heritability estimate (proportion of trait variance explained by additive SNP effects)
- Per-trait variance fractions for SNPs, CNVs and rearrangements from the composite mixed model
- variance_components.tsv — table of all per-trait variance-component estimates

## How to apply

Load the filtered genotypes — here 139,396 SNPs and 22,058 indels with minor allele counts >=5, plus the 68 SV genotypes with minor allele counts >=5 — together with normalized values for all 228 quantitative traits. Use LDAK to construct a SNP-only kinship matrix for the baseline model, and for the composite model construct three separate kinship matrices: one for SNPs, one for CNVs and one for rearrangements. For each trait, run LDAK mixed-model estimation first with the SNP-only kinship (giving SNP-only heritability, reported range 0-74%, median 30%), then with the composite model to partition variance among SNPs, CNVs and rearrangements (reported averages: SNPs 24%, CNVs 7%, rearrangements 4%). Rationale: SVs are often in weak linkage with SNPs, so fitting SNP-only models can overestimate SNP contributions; including SV kinship components corrects for this missing-heritability bias. Compile all per-trait estimates into variance_components.tsv with the SNP-only heritability and the three variance-component fractions.

## Related tools

- **LDAK** (Constructs the SNP-only and composite (SNP/CNV/rearrangement) kinship matrices and runs the mixed-model heritability estimation per trait)

## Evaluation signals

- Each trait's composite variance fractions are non-negative and together with residual variance account for the trait's total heritable variance
- Per-class averages across the 228 traits reproduce the reported values: SNPs ~24%, CNVs ~7%, rearrangements ~4% of trait variance
- Distribution of SNP-only heritability across traits spans the reported range 0-74% with a median near 30%
- variance_components.tsv contains one row per trait with the SNP-only heritability and all three composite variance-component fractions populated
- Composite-model SNP fraction is consistently lower than or comparable to the SNP-only estimate for traits where CNVs/rearrangements contribute variance, reflecting reassignment of variance to SV components

## Limitations

- The study prioritized high specificity over high sensitivity in SV calling, so some SVs were not detected and their variance contribution is missed
- SVs in weak linkage with SNPs may be badly tagged by SNPs, so SNP-only heritability estimates can overestimate SNP impact and contribute to missing heritability
- Translocations and inversions were not re-examined for genotyping accuracy because they are challenging to genotype; errors in SV genotypes propagate into the composite kinship

## Evidence

- [abstract] composite model partitioning: "On average, SNPs explained 24% of trait variance, CNVs 7% and rearrangements 4%"
- [abstract] mixed-model estimation: "For each phenotype, we used mixed model analysis to estimate the total proportion of variance explained by the additive contribution of genomic variants"
- [abstract] variant inputs: [evidence span withheld — anchor status: not_found]
- [discussion] weak linkage caveat: [evidence span withheld — anchor status: not_found]
