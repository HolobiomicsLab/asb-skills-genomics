---
name: variant-prioritization-benchmarking
description: Use when you need to quantify recall and false-positive counts of a variant
  prioritizer on trios with known spiked-in causal variants, e.g. testing whether
  HPO terms improve precision.
when_to_use_negative:
- Input is a fully real disease cohort without known causal variants — recall cannot
  be computed without spiked-in or known truth variants
- The question concerns CNV detection — CNV analysis is not yet available in eDiVA
  and would confound the benchmark
- You need phenotypically unbiased recall estimates — HPO-based soft filtering reduces
  recall due to incomplete gene-phenotype maps in public databases
edam_operation: http://edamontology.org/operation_3225
edam_topics:
- http://edamontology.org/topic_3673
- http://edamontology.org/topic_0622
tools:
- name: eDiVA
  role: Full pipeline used to generate semisynthetic trios, annotate/score variants,
    and run prioritization under HPO and no-HPO configurations
  repo: https://github.com/mbosio85/ediva
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1002/humu.23772
    title: eDiVA—Classification and prioritization of pathogenic variants for clinical
      diagnostics
  source_content_hash: sha256:8834475d17bf241eed4cffb4694fcd1cd38bbbbed7574e534b0e0d9eaee79b72
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mbosio85/ediva
  tools:
  - eDiVA
  edam_operation: http://edamontology.org/operation_3225
  edam_topics:
  - http://edamontology.org/topic_3673
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# variant-prioritization-benchmarking

## Summary

Benchmark a clinical variant prioritization pipeline (eDiVA-Prioritize) on semisynthetic disease trios to quantify how phenotype (HPO)-based soft filtering trades false positives against recall across inheritance modes.

## When to use

Use when you need to quantify recall and false-positive counts of a variant prioritizer on trios with known spiked-in causal variants, e.g. testing whether HPO terms improve precision.

## When NOT to use

- Input is a fully real disease cohort without known causal variants — recall cannot be computed without spiked-in or known truth variants
- The question concerns CNV detection — CNV analysis is not yet available in eDiVA and would confound the benchmark
- You need phenotypically unbiased recall estimates — HPO-based soft filtering reduces recall due to incomplete gene-phenotype maps in public databases

## Inputs

- 6,811 semisynthetic parent-child trios (ClinVar pathogenic variants spiked into 1000 Genomes CEPH trio NA12878/NA12891/NA12892 WES data)
- joint multisample VCFs (combined.variants.supplement.vcf) and pedigree.tree files per trio
- combined.variants.supplement.ranked.csv files from eDiVA-Annotate + eDiVA-Score
- HPO ID list per simulated disease (hpo_list.txt) and an empty list (Resource/empty_list.txt)

## Outputs

- ediva_hpo_comparison.csv with per-inheritance-mode recall and average false positives for both configurations
- eDiVA-Prioritize output shortlists (combined.variants.supplement.filtered.<mode>.csv) per configuration and inheritance mode

## How to apply

Regenerate the 6,811 semisynthetic parent-child trios by spiking ClinVar pathogenic variants into real WES data from the 1000 Genomes CEPH trio (NA12878/NA12891/NA12892) using eDiVA-Predict, then produce joint multisample VCFs (combined.variants.supplement.vcf) and pedigree.tree files via eDiVA_prepare_for_annotation. Annotate and rank variants with eDiVA-Annotate + eDiVA-Score (random forest, 1000 trees) to obtain combined.variants.supplement.ranked.csv files. Run eDiVA-Prioritize twice with identical parameters (MODE standard, FAMILY_TYPE trio, lenient default filter setting): configuration (a) with an empty HPO list (Resource/empty_list.txt) and configuration (b) with HPO IDs describing each simulated disease (hpo_list.txt) for soft phenotype-based gene prioritization, choosing inheritance modes among dominant_denovo, recessive (homozygous and compound), and similar. For each configuration and each inheritance mode (recessive homozygous: 3,353 variants; dominant de novo: 2,592; recessive compound heterozygous: 866), compute recall — whether the causal variant appears in the output shortlist — and the average number of false positives across all benchmarked cases as a precision proxy, then assemble per-mode metrics into ediva_hpo_comparison.csv.

## Related tools

- **eDiVA** (Full pipeline used to generate semisynthetic trios, annotate/score variants, and run prioritization under HPO and no-HPO configurations) — https://github.com/mbosio85/ediva

## Examples

```
nextflow run eDiVA-predict.nf --OUTF ./ --ANNOTATED combined.variants.supplement.ranked.csv --MODE standard --PEDIGREE pedigree.tree --FAMILY_TYPE trio --HPO hpo_list.txt --INHERITANCE choose
```

## Evaluation signals

- The HPO-enabled configuration shows a two-fold reduction in false positives across all tested inheritance modes relative to the empty-HPO configuration
- Recall reduction in the HPO configuration is at most mild and attributable to incomplete gene-phenotype maps in public databases
- Per-mode variant counts match the benchmark: recessive homozygous 3,353; dominant de novo 2,592; recessive compound heterozygous 866
- Recall for each configuration is verifiable by checking whether the known causal ClinVar variant appears in the output candidate shortlist
- Output CSV (ediva_hpo_comparison.csv) is ordered by inheritance mode and configuration with one row per mode/configuration pair

## Limitations

- Filtering by in silico gene panels (HPO soft prioritization) caused a reduction in recall
- Incomplete maps of genotype-phenotype (gene-phenotype) relations in public databases limit HPO-based prioritization recall
- The benchmark relies on semisynthetic trios rather than fully real disease cohorts
- eDiVA-Prioritize is designed for small groups of related samples (families or parent-child trios), not large unrelated cohorts

## Evidence

- [discussion] HPO soft filtering trade-off: "incomplete maps of known genotype–phenotype (or gene–phenotype) relations in public databases led to a mild reduction in recall."
- [discussion] semisynthetic benchmark design: "a highly realistic benchmark test combining real WES data of a parent–child trio with thousands of pathogenic ClinVar variants to generate 6,811 semisynthetic disease trios"
- [methods] prioritization filter chain: "(b) removal of all variants that do not segregate according to the selected inheritance mode; (c) filtering based on functional and population genomic features"
- [methods] HPO-based prioritization step: "(d) prioritization based on user defined clinical phenotypes (as HPO IDs)"
- [readme] HPO parameter in Prioritize README: "HPO_list= Text file with HPO terms to be used for soft filtering. It can point to an empty file such as in Resource/empty_list.txt"
- [readme] Prioritize invocation from README: "nextflow run eDiVA-predict.nf --OUTF ./ --ANNOTATED combined.variants.supplement.ranked.csv --MODE standard"
