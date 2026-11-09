---
name: hpo-phenotype-based-prioritization
description: Use when prioritizing candidate causal variants from an annotated trio/family
  multisample VCF and known disease phenotypes (HPO IDs) are available to boost precision
  without switching inheritance filtering logic.
when_to_use_negative:
- No HPO terms or phenotype description for the disease exist — the empty-list configuration
  is the correct baseline and HPO soft filtering cannot contribute signal.
- 'CNV-driven disease: CNV analysis is not yet available in eDiVA, so point-mutation
  prioritization with HPO terms would miss such causes.'
- A strict precision-only screen where recall loss from incomplete public gene-phenotype
  maps is unacceptable for the cohort at hand.
edam_operation: http://edamontology.org/operation_0362
edam_topics:
- http://edamontology.org/topic_0625
tools:
- name: eDiVA
  role: Provides the eDiVA-Prioritize module that applies HPO-ID soft filtering and
    phenotype-based gene prioritization on ranked multisample variant lists
  repo: https://github.com/mbosio85/ediva
- name: PhenoDB
  role: Phenotype-aware comparator tool used to benchmark HPO-based prioritization
    performance
- name: Phen–Gen
  role: Phenotype-aware comparator tool (DCOD-score) used to benchmark HPO-based prioritization
    performance
provenance:
  source_task_ids:
  - task_006
  - task_007
  source_papers:
  - doi: 10.1002/humu.23772
    title: eDiVA—Classification and prioritization of pathogenic variants for clinical
      diagnostics
  source_content_hash: sha256:6becabf8048461c8db490fe1c436ee97b7fc9ed78ec37ac71b139d9ee165f462
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mbosio85/ediva
  tools:
  - PhenoDB
  - Phen–Gen
  - eDiVA
  edam_operation: http://edamontology.org/operation_0362
  edam_topics:
  - http://edamontology.org/topic_0625
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# hpo-phenotype-based-prioritization

## Summary

Soft-filters and ranks candidate causal variants from a ranked multisample VCF using user-supplied Human Phenotype Ontology (HPO) term IDs, as implemented in the eDiVA-Prioritize module, to improve precision of familial disease variant prioritization.

## When to use

Use when prioritizing candidate causal variants from an annotated trio/family multisample VCF and known disease phenotypes (HPO IDs) are available to boost precision without switching inheritance filtering logic.

## When NOT to use

- No HPO terms or phenotype description for the disease exist — the empty-list configuration is the correct baseline and HPO soft filtering cannot contribute signal.
- CNV-driven disease: CNV analysis is not yet available in eDiVA, so point-mutation prioritization with HPO terms would miss such causes.
- A strict precision-only screen where recall loss from incomplete public gene-phenotype maps is unacceptable for the cohort at hand.

## Inputs

- combined.variants.supplement.ranked.csv (annotated, eDiVA-Score-ranked multisample variant table)
- pedigree.tree pedigree file from eDiVA_prepare_for_annotation
- hpo_list.txt: text file with HPO terms describing the disease
- inheritance mode selection (dominant_denovo, recessive, dominant_inherited, Xlinked, compound)

## Outputs

- combined.variants.supplement.filtered.<inheritance>.csv: variants passing filters, sorted by eDiVA-Score
- combined.variants.supplement.filtered.<inheritance>.csv.xlsx: Excel shortlist with extra OMIM and ClinVar information
- variant_prioritization_report.xlsx with one spreadsheet per inheritance type

## How to apply

Run eDiVA-Annotate + eDiVA-Score on the joint multisample VCF (combined.variants.supplement.vcf, produced with the pedigree.tree file) to obtain combined.variants.supplement.ranked.csv, then run eDiVA-Prioritize with --MODE standard, --PEDIGREE pedigree.tree, --FAMILY_TYPE trio, and --HPO hpo_list.txt containing the HPO terms describing the simulated disease. The HPO IDs act as soft phenotype-based gene prioritization applied on top of the lenient default filter setting (Table S1), while the inheritance mode (dominant_denovo, recessive, compound) is chosen separately. Expect a roughly two-fold reduction in false positives versus an empty HPO list (Resource/empty_list.txt) at the cost of some recall from incomplete gene-phenotype maps. Compare the ranked output (combined.variants.supplement.filtered.<inheritance>.csv) with and without HPO terms to confirm the precision/recall trade-off before adopting the HPO-filtered shortlist.

## Related tools

- **eDiVA** (Provides the eDiVA-Prioritize module that applies HPO-ID soft filtering and phenotype-based gene prioritization on ranked multisample variant lists) — https://github.com/mbosio85/ediva
- **PhenoDB** (Phenotype-aware comparator tool used to benchmark HPO-based prioritization performance)
- **Phen–Gen** (Phenotype-aware comparator tool (DCOD-score) used to benchmark HPO-based prioritization performance)

## Examples

```
nextflow run eDiVA-predict.nf --OUTF ./ --ANNOTATED combined.variants.supplement.ranked.csv --MODE standard --PEDIGREE pedigree.tree --FAMILY_TYPE trio --HPO hpo_list.txt --INHERITANCE recessive
```

## Evaluation signals

- HPO-enabled run shows approximately a two-fold reduction in average false positives versus the empty HPO list across all tested inheritance modes.
- Recall reduction vs. no-HPO run is at most mild, attributable to incomplete gene-phenotype maps in public databases.
- Causal recessive homozygous and compound heterozygous variants rank within the top five candidates (median = 1); dominant de novo within the top 25 (median = 4).
- Output files combined.variants.supplement.filtered.<inheritance>.csv exist, are sorted by eDiVA-Score, and only contain variants compatible with the selected inheritance pattern.

## Limitations

- Incomplete maps of known genotype-phenotype (or gene-phenotype) relations in public databases led to a mild reduction in recall when filtering by in silico gene panels.
- The HPO term list is a soft filter — overly specific or sparse HPO annotations will shrink the candidate gene panel and can drop the true causal variant.
- Benchmark relies on semisynthetic trios (6,811 ClinVar spike-in trios from the CEPH trio NA12878/NA12891/NA12892) rather than fully real disease cohorts.

## Evidence

- [readme] HPO terms soft filtering: "HPO_list= Text file with HPO terms to be used for soft filtering. It can point to an empty file such as in Resource/empty_list.txt"
- [methods] HPO ID prioritization step: "prioritization based on user defined clinical phenotypes (as HPO IDs)"
- [abstract] Phenotype improves precision: "Clinical information, such as disease phenotype or mode of inheritance, is incorporated to improve the precision of the prioritization algorithm."
- [discussion] Recall limitation of gene-phenotype maps: "incomplete maps of known genotype–phenotype (or gene–phenotype) relations in public databases led to a mild reduction in recall."
- [intro] Prioritize module purpose: "eDiVA returns a shortlist of candidate variants compatible with the selected disease inheritance model and the pedigree information"
