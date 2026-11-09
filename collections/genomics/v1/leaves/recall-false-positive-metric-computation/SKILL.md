---
name: recall-false-positive-metric-computation
description: Use when evaluating a candidate-variant shortlist from a prioritization
  run where ground-truth causal variants are known (e.g. semisynthetic spiked-in ClinVar
  variants), to quantify recall and false positives per inheritance mode.
when_to_use_negative:
- Input is a real disease cohort without known ground-truth causal variants — recall
  cannot be computed without spiked-in or resolved causal variants
- The task is variant calling accuracy (e.g. SNV/indel sensitivity of eDiVA-Predict)
  rather than candidate shortlist prioritization quality
- You need formal precision (1 - FP/(TP+FP)) rather than the average-false-positives
  proxy used when shortlists vary in length
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3325
tools:
- name: eDiVA
  role: Produces the ranked and filtered candidate shortlists (eDiVA-Prioritize) whose
    recall and false-positive counts are being computed
  repo: https://github.com/mbosio85/ediva
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1002/humu.23772
    title: eDiVA—Classification and prioritization of pathogenic variants for clinical
      diagnostics
  source_content_hash: sha256:d02f5b1ab375527a7fdc5a0e20d407c1bf379bdffffd5803f3ca170cc549d64a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mbosio85/ediva
  tools:
  - eDiVA
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3325
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 6
---

# recall-false-positive-metric-computation

## Summary

Compute recall and average false positives as a precision proxy when benchmarking a variant prioritization tool across inheritance modes, as done for eDiVA-Prioritize on 6,811 semisynthetic parent-child trios built from ClinVar pathogenic variants spiked into the 1000 Genomes CEPH trio.

## When to use

Use when evaluating a candidate-variant shortlist from a prioritization run where ground-truth causal variants are known (e.g. semisynthetic spiked-in ClinVar variants), to quantify recall and false positives per inheritance mode.

## When NOT to use

- Input is a real disease cohort without known ground-truth causal variants — recall cannot be computed without spiked-in or resolved causal variants
- The task is variant calling accuracy (e.g. SNV/indel sensitivity of eDiVA-Predict) rather than candidate shortlist prioritization quality
- You need formal precision (1 - FP/(TP+FP)) rather than the average-false-positives proxy used when shortlists vary in length

## Inputs

- Prioritized candidate shortlists (eDiVA combined.variants.supplement.filtered.<mode>.csv per configuration and inheritance mode)
- Ground-truth causal variant identities from the 6,811 semisynthetic ClinVar-spiked trios
- Inheritance mode labels (dominant_denovo, recessive homozygous, recessive compound heterozygous)
- Configuration metadata (HPO list vs. empty list, filter settings)

## Outputs

- Per-mode, per-configuration recall values (fraction of trios where the causal variant is in the shortlist)
- Average number of false positives per inheritance mode and configuration (precision proxy)
- Comparison table (ediva_hpo_comparison.csv) ordered by inheritance mode and configuration

## How to apply

For each benchmarked case, check whether the causal (spiked-in) variant appears in the tool's output shortlist to score recall (1 if present, 0 otherwise), and count all other reported variants as false positives; average false positives across all cases as a precision proxy. Stratify the metrics by inheritance mode — recessive homozygous (3,353 variants), dominant de novo (2,592), recessive compound heterozygous (866) — and by configuration (e.g. eDiVA-Prioritize run with an empty HPO list via Resource/empty_list.txt vs. HPO IDs in hpo_list.txt, keeping MODE standard, FAMILY_TYPE trio, and the lenient default filter setting of Table S1 identical). Assemble per-mode, per-configuration metrics into a comparison table (e.g. ediva_hpo_comparison.csv) ordered by inheritance mode and configuration. Validate that a phenotype-aware configuration shows the expected trade-off: fewer false positives (the paper reports a two-fold reduction with HPO IDs across all tested modes) accompanied by at most a mild recall reduction due to incomplete gene-phenotype maps in public databases.

## Related tools

- **eDiVA** (Produces the ranked and filtered candidate shortlists (eDiVA-Prioritize) whose recall and false-positive counts are being computed) — https://github.com/mbosio85/ediva

## Examples

```
nextflow run eDiVA-predict.nf --OUTF ./ --ANNOTATED combined.variants.supplement.ranked.csv --MODE standard --PEDIGREE pedigree.tree --FAMILY_TYPE trio --HPO hpo_list.txt --INHERITANCE choose
```

## Evaluation signals

- For each trio, exactly one ground-truth causal variant exists per mode; recall is binary per case and must lie in [0,1] when aggregated
- Phenotype-aware (HPO) configuration should show roughly two-fold fewer average false positives than the empty-HPO configuration across all tested inheritance modes
- Recall under HPO-based prioritization should drop only mildly relative to the no-phenotype run; a large recall loss signals over-aggressive filtering or incorrect HPO term mapping
- Per-mode sample sizes must match the benchmark counts (3,353 recessive homozygous, 2,592 dominant de novo, 866 recessive compound heterozygous)
- The comparison table must be complete for both configurations and all inheritance modes, ordered by mode and configuration

## Limitations

- The benchmark relies on semisynthetic trios (ClinVar pathogenic variants spiked into real 1000 Genomes CEPH trio WES data), not fully real disease cohorts
- HPO-based filtering can reduce recall because gene-phenotype relations in public databases are incomplete
- Average false positives is only a proxy for precision; it is not a formal precision metric and depends on shortlist length

## Evidence

- [methods] Running eDiVA-Prioritize with HPO IDs versus without produced a two-fold reduction in false positives, but filtering caused reduced recall (Source card task_007 finding): [evidence span withheld — anchor status: not_found]
- [discussion] incomplete genotype-phenotype maps limit HPO recall (article discussion): "incomplete maps of known genotype–phenotype (or gene–phenotype) relations in public databases led to a mild reduction in recall."
- [discussion] semisynthetic benchmark of 6,811 trios with ClinVar variants into real WES trio data: "a highly realistic benchmark test combining real WES data of a parent–child trio with thousands of pathogenic ClinVar variants to generate 6,811 semisynthetic disease trios"
- [methods] lenient default filter setting used in prioritization runs: "By default, eDiVA applies a lenient filter setting defined in Table S1"
- [readme] HPO list parameter can point to an empty file, enabling the no-phenotype configuration: "HPO_list= Text file with HPO terms to be used for soft filtering. It can point to an empty file such as in Resource/empty_list.txt"
