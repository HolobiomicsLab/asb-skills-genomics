---
name: cnv-genotype-to-gene-region-mapping
description: Use when per-strain CNV/duplication genotypes (e.g. from cn.MOPS or SURVIVOR
  consensus calls) must be intersected with genome annotation to define per-strain
  affected gene sets, as a precursor to dosage-expression analyses.
when_to_use_negative:
- The structural variants of interest are translocations or inversions — the source
  study explicitly did not re-examine these because they were more challenging to
  genotype accurately
- The input is SNP or indel genotype data rather than copy-number/duplication calls;
  this skill maps dosage intervals to genes, not single-nucleotide variants
- No expression data are available for the duplicated/non-duplicated strain pair —
  the mapping alone cannot test dosage-driven induction
edam_operation: http://edamontology.org/operation_3196
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: cn.MOPS
  role: Copy-number inference whose initial population variant calls were manually
    curated to provide per-strain CNV genotypes for mapping to duplicated gene regions
- name: SURVIVOR
  role: Consensus merging of SV caller outputs and SV filtering (size/region-based)
    to produce the genotype catalogue from which duplication intervals are extracted
  repo: https://github.com/fritzsedlazeck/SURVIVOR
- name: manual expert review
  role: Manual vetting of SV calls by visual inspection of read alignments in multiple
    strains before genotypes are trusted for gene-region mapping
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/ncomms14061
    title: Transient structural variations have strong effects on quantitative traits
      and reproductive isolation in fission yeast
  source_content_hash: sha256:9e92ed35e600d73ed80ab284be2840ae18c82f52dd596ca571ddfa27b7095d82
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/fritzsedlazeck/SURVIVOR
  tools:
  - SURVIVOR
  - cn.MOPS
  - manual expert review
  edam_operation: http://edamontology.org/operation_3196
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# CNV genotype to gene-region mapping

## Summary

Maps duplication/CNV genotypes in S. pombe strains to the sets of genes contained within the duplicated regions, enabling downstream tests of whether gene dosage drives expression induction in duplicated relative to non-duplicated clonal strains.

## When to use

Use when per-strain CNV/duplication genotypes (e.g. from cn.MOPS or SURVIVOR consensus calls) must be intersected with genome annotation to define per-strain affected gene sets, as a precursor to dosage-expression analyses.

## When NOT to use

- The structural variants of interest are translocations or inversions — the source study explicitly did not re-examine these because they were more challenging to genotype accurately
- The input is SNP or indel genotype data rather than copy-number/duplication calls; this skill maps dosage intervals to genes, not single-nucleotide variants
- No expression data are available for the duplicated/non-duplicated strain pair — the mapping alone cannot test dosage-driven induction

## Inputs

- Per-strain CNV/duplication genotypes (consensus VCF from SURVIVOR merge of Delly/Lumpy/Pindel/cn.MOPS calls, with copy numbers for 49 CNVs segregating in clonal populations)
- S. pombe gene annotation (to assign genes to duplicated-region intervals)
- ArrayExpress E-MTAB-4019 DNA microarray expression data from log phase cultures
- Strain metadata linking each duplicated strain to its non-duplicated clonal counterpart and clonal-cluster assignment

## Outputs

- Per-strain-pair set of genes located within the duplicated region
- Per-pair expression induction statistic and significance for within-duplication genes
- per_pair_expression_induction.tsv (strain pair, duplicated-gene set, induction statistic, significance)
- Spearman rank correlation between duplication copy number and expression induction across the eight strain pairs

## How to apply

Start from curated CNV genotypes — in the source study these came from a SURVIVOR consensus of Delly, Lumpy, Pindel and cn.MOPS calls, filtered to SVs detected by at least two callers (315 calls), removing breakpoints overlapping low-complexity regions or annotated LTRs. For each duplicated strain, take the duplication call's genomic interval and intersect it with the S. pombe gene annotation to define the set of genes located within the duplicated region. Restrict expression re-examination to CNVs (translocations and inversions were excluded because they were more challenging to genotype accurately) and, following the study, re-examine only the 49 CNVs present within at least one clonal population. Pair each duplicated strain with a non-duplicated strain from the same clonal cluster, then compute expression induction of the within-duplication gene set in the duplicated strain relative to its clonal counterpart using the E-MTAB-4019 microarray data (log phase cultures). Assess induction significance per pair and correlate duplication copy number with induction across pairs (Spearman rank correlation) to detect dose-dependence.

## Related tools

- **cn.MOPS** (Copy-number inference whose initial population variant calls were manually curated to provide per-strain CNV genotypes for mapping to duplicated gene regions)
- **SURVIVOR** (Consensus merging of SV caller outputs and SV filtering (size/region-based) to produce the genotype catalogue from which duplication intervals are extracted) — https://github.com/fritzsedlazeck/SURVIVOR
- **manual expert review** (Manual vetting of SV calls by visual inspection of read alignments in multiple strains before genotypes are trusted for gene-region mapping)

## Examples

```
git clone https://github.com/fritzsedlazeck/SURVIVOR.git; cd SURVIVOR/Debug; make
```

## Evaluation signals

- In the source study, 7 of 8 strain pairs showed significant induction of within-duplication gene expression; replication should recover a similar proportion of significant pairs
- The Spearman rank correlation between duplication copy number and expression induction across strain pairs should be computed and reported alongside per-pair results
- Genes outside the duplicated regions should be tracked as a contrast set — the study found only 'some genes outside the duplicated regions also showed altered expression levels'
- per_pair_expression_induction.tsv must contain the required columns: strain pair, duplicated-gene set, induction statistic, significance
- Each mapped gene set should be verified against the duplication call boundaries: only genes located within the duplicated interval are included

## Limitations

- The study prioritized high specificity over high sensitivity, so not all SVs were detected — some genuine duplications may be missing from the genotype catalogue used for mapping
- Translocations and inversions were not re-examined/genotyped, so this mapping cannot cover those SV classes
- CNV copy number was only weakly correlated with SNP variation in nearby regions (Spearman ρ=0.22, P=0.041), so CNV states cannot be imputed from nearby SNP patterns
- CNVs are transient within clonal populations (40 of 49 CNVs clearly segregating), so genotype states can differ even among closely related strains and must be verified per strain

## Evidence

- [abstract] Seven of eight pairs induced, correlated with copy number: "In seven out of the eight strain pairs, the expression levels of genes within duplications were significantly induced"
- [abstract] Paired expression assay design: "We assayed transcript expression from log phase cultures using DNA microarrays, each time comparing a duplicated to a non-duplicated strain from within the same clonal population."
- [abstract] Restriction to CNVs for genotyping re-examination: "Since translocations and inversions were more challenging to accurately genotype, we did not re-examine these variants."
- [supplementary] CNV genotype basis from cn.MOPS curation: "Manual curation of initial population variant calls from cn.MOPS"
- [abstract] Transient CNV states require per-strain genotyping: "This analysis verified that 40 out of these 49 CNVs (37 duplications, three deletions) were clearly segregating within at least one clonal cluster"
- [readme] SURVIVOR filter command used to produce filtered genotype VCF: [evidence span withheld — anchor status: external_doc]
