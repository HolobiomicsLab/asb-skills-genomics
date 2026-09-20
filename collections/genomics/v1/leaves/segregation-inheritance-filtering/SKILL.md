---
name: segregation-inheritance-filtering
description: Use when filtering a ranked multisample variant list (combined.variants.supplement.ranked.csv)
  against pedigree.tree so only variants compatible with the chosen inheritance mode
  (e.g. dominant_denovo, recessive, compound, Xlinked) survive for prioritization.
when_to_use_negative:
- The sample group is a large cohort rather than a small family or parent–child trio
  — eDiVA-Prioritize is designed for variants from small groups of related samples
  analyzed jointly.
- The question is CNV detection — CNV analysis was not yet available in eDiVA (integration
  pending at publication time).
- Variants have not yet been jointly multisample-called and scored — filtering requires
  combined.variants.supplement.ranked.csv from eDiVA-Annotate + eDiVA-Score first.
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_3673
- http://edamontology.org/topic_0622
tools:
- name: eDiVA
  role: Runs the eDiVA-Prioritize nextflow step that removes variants not segregating
    with the selected inheritance mode, applies rarity filters, and soft-prioritizes
    by HPO terms
  repo: https://github.com/mbosio85/ediva
- name: GATK
  role: Upstream variant calling via eDiVA-Predict follows GATK best practices to
    produce the multisample VCFs that feed the inheritance filter
provenance:
  source_task_ids:
  - task_007
  - task_006
  source_papers:
  - doi: 10.1002/humu.23772
    title: eDiVA—Classification and prioritization of pathogenic variants for clinical
      diagnostics
  source_content_hash: sha256:d39a96b34be75480a368e0ebb6d1f53a517c1a5a12545d27773974ac4a98d4e5
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mbosio85/ediva
  tools:
  - GATK
  - PhenoDB
  - Phen–Gen
  - eDiVA
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_3673
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# segregation-inheritance-filtering

## Summary

Removes variants that do not segregate according to a selected disease inheritance mode (dominant de novo, dominant inherited, recessive homozygous, recessive compound heterozygous, or X-linked) using pedigree information in eDiVA-Prioritize, yielding a compatible candidate shortlist.

## When to use

Use when filtering a ranked multisample variant list (combined.variants.supplement.ranked.csv) against pedigree.tree so only variants compatible with the chosen inheritance mode (e.g. dominant_denovo, recessive, compound, Xlinked) survive for prioritization.

## When NOT to use

- The sample group is a large cohort rather than a small family or parent–child trio — eDiVA-Prioritize is designed for variants from small groups of related samples analyzed jointly.
- The question is CNV detection — CNV analysis was not yet available in eDiVA (integration pending at publication time).
- Variants have not yet been jointly multisample-called and scored — filtering requires combined.variants.supplement.ranked.csv from eDiVA-Annotate + eDiVA-Score first.

## Inputs

- combined.variants.supplement.ranked.csv (annotated, eDiVA-Score-ranked multisample VCF output)
- pedigree.tree file from eDiVA_prepare_for_annotation
- Selected inheritance mode string (dominant_inherited, recessive, dominant_denovo, Xlinked, compound)
- hpo_list.txt (HPO terms for soft phenotype prioritization, or empty list)
- gene exclusion list (Resource/gene_exclusion_list.txt or empty file)

## Outputs

- combined.variants.supplement.<mode>.csv (all variants annotated with inheritance-pattern compatibility flag)
- combined.variants.supplement.filtered.<mode>.csv (variants passing filters, sorted by eDiVA-Score)
- combined.variants.supplement.filtered.<mode>.csv.xlsx (Excel with extra OMIM and ClinVar information)
- variant_prioritization_report.xlsx (one spreadsheet per inheritance type)

## How to apply

Provide the pedigree.tree file from eDiVA_prepare_for_annotation together with the annotated ranked CSV, and run eDiVA-Prioritize with MODE standard (or strict), FAMILY_TYPE trio, an HPO list (or Resource/empty_list.txt to skip phenotype soft filtering), a gene exclusion list, and --INHERITANCE set to one of 'dominant_inherited', 'recessive', 'dominant_denovo', 'Xlinked', or 'compound'. The module applies the inheritance-compatibility filter first, then functional/population-genomic rarity filters (lenient defaults per Table S1, AF ≤ 0.01), and outputs per-inheritance CSVs plus an Excel variant_prioritization_report.xlsx with OMIM/ClinVar annotations. Benchmark the shortlist by checking whether the causal variant is retained (recall) and counting average false positives across cases; note that in semisynthetic trios with 3,353 recessive homozygous, 2,592 dominant de novo, and 866 compound heterozygous ClinVar spike-ins, eDiVA ranked causal recessive variants at median rank 1 and dominant de novo at median rank 4.

## Related tools

- **eDiVA** (Runs the eDiVA-Prioritize nextflow step that removes variants not segregating with the selected inheritance mode, applies rarity filters, and soft-prioritizes by HPO terms) — https://github.com/mbosio85/ediva
- **GATK** (Upstream variant calling via eDiVA-Predict follows GATK best practices to produce the multisample VCFs that feed the inheritance filter)

## Examples

```
nextflow run eDiVA-Prioritize.nf --OUTF ./ --ANNOTATED combined.variants.supplement.ranked.csv --MODE standard --PEDIGREE pedigree.tree --FAMILY_TYPE trio --HPO hpo_list.txt --EXCLUSIONLIST Resource/gene_exclusion_list.txt --INHERITANCE recessive
```

## Evaluation signals

- Output folder named after the inheritance mode contains both combined.variants.supplement.<mode>.csv (with compatibility column) and the filtered version sorted by eDiVA-Score
- The AFFECTED column in the concatenated family_info file is verified correct (0 for unaffected, 1 for affected) before running, since segregation relies on it
- In benchmark trios, causal recessive homozygous and compound heterozygous variants appear within the top five candidates (median = 1) and dominant de novo within the top 25 (median = 4)
- Adding HPO soft filtering should give roughly a two-fold reduction in false positives across inheritance modes, at the cost of at most a mild recall reduction from incomplete gene–phenotype maps
- variant_prioritization_report.xlsx contains one spreadsheet per inheritance type with updated results

## Limitations

- HPO-based soft filtering after segregation filtering reduced recall because gene–phenotype maps in public databases are incomplete.
- CNV analysis is not covered; only SNV/indel segregation by inheritance mode is supported.
- Only five inheritance modes are supported; unlisted disease models cannot be filtered directly.

## Evidence

- [intro] eDiVA returns a shortlist of candidate variants compatible with the selected disease inheritance model and the pedigree information: "eDiVA returns a shortlist of candidate variants compatible with the selected disease inheritance model and the pedigree information"
- [methods] removal of variants not segregating with inheritance mode; filtering on functional/population features; HPO prioritization: "(b) removal of all variants that do not segregate according to the selected inheritance mode; (c) filtering based on functional and population genomic features"
- [readme] eDiVA-Prioritize parameters: MODE standard, PEDIGREE pedigree.tree, FAMILY_TYPE trio, HPO list, INHERITANCE among five modes: "nextflow run eDiVA-predict.nf --OUTF ./\ --ANNOTATED combined.variants.supplement.ranked.csv \ --MODE standard \ --PEDIGREE pedigree.tree \ --FAMILY_TYPE trio"
- [results] Using HPO IDs for prioritization produced a two-fold reduction in false positives across all tested inheritance modes, but filtering by in silico gene panels also caused a reduction in recall.: [evidence span withheld — anchor status: not_found]
- [discussion] incomplete maps of known genotype–phenotype relations in public databases led to a mild reduction in recall: "incomplete maps of known genotype–phenotype (or gene–phenotype) relations in public databases led to a mild reduction in recall."
