---
name: auc-benchmark-comparison-across-predictors
description: Use when two or more continuous variant scores (e.g. |ΔΨ|, SPIDEX ΔΨ,
  MaxEntScan |ΔMES|) must be compared for binary prioritization power on the same
  labeled mutations, with significance testing and scope-stratified subsets.
when_to_use_negative:
- The task is predicting a continuous ΔΨ value rather than ranking/binary prioritization
  — use PCC-based evaluation instead, as in the vex-seq challenge analysis
- The candidate score is a binary classifier output or lacks a continuous ranking
  statistic usable for AUC computation
- Variants lie in intergenic regions or more than 200 bp from splice sites, where
  PredPSI-SVR is not applicable after its VCF filtering step
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3320
- http://edamontology.org/topic_3511
tools:
- name: PredPSI-SVR
  role: Score under test; ranks synonymous mutations by absolute predicted ΔΨ, run
    with (-p) and without (-p) experimental wild-type Ψ
  repo: https://github.com/chenkenbio/PredPSI-SVR
- name: SPANR
  role: Competing baseline; provides SPIDEX ΔΨ scores for the same variants
- name: MaxEntScan
  role: Competing baseline; scored via the summed absolute change |ΔMES5|+|ΔMES3|,
    with limited scanning scope relative to PredPSI-SVR
- name: ANNOVAR
  role: Annotates the input VCF with hg19 ensGene annotations to enable the splice-proximity
    and region filtering before scoring
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1002/humu.23785
    title: Predicting the change of exon splicing caused by genetic variant using
      support vector regression
  source_content_hash: sha256:013babdceb84483284597fbaab06d7366544828efb13d59d672efdb5bc0559f8
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/chenkenbio/PredPSI-SVR
  tools:
  - ANNOVAR
  - MaxEntScan
  - Perl
  - PredPSI-SVR
  - SPANR
  - samtools
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3320
  - http://edamontology.org/topic_3511
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# auc-benchmark-comparison-across-predictors

## Summary

Benchmarks competing variant-prioritization scores by computing AUCs on a shared labeled mutation set, including stratified subsets (e.g., outside a competitor's scanning scope), with significance testing. Used in PredPSI-SVR to show it outperforms SPANR and MaxEntScan on pathogenic synonymous mutations.

## When to use

Use when two or more continuous variant scores (e.g. |ΔΨ|, SPIDEX ΔΨ, MaxEntScan |ΔMES|) must be compared for binary prioritization power on the same labeled mutations, with significance testing and scope-stratified subsets.

## When NOT to use

- The task is predicting a continuous ΔΨ value rather than ranking/binary prioritization — use PCC-based evaluation instead, as in the vex-seq challenge analysis
- The candidate score is a binary classifier output or lacks a continuous ranking statistic usable for AUC computation
- Variants lie in intergenic regions or more than 200 bp from splice sites, where PredPSI-SVR is not applicable after its VCF filtering step

## Inputs

- Filtered synonymous-mutation VCF (variants within 200 bp of splice sites, non-intergenic)
- Experimental wild-type Ψ file (for the -p option, SynonMut-psi setting)
- Per-variant scores from each predictor (PredPSI-SVR OUTPUT.dpsi, SPIDEX ΔΨ, MaxEntScan |ΔMES5|+|ΔMES3|)
- Binary labels marking which mutations affect splicing / are pathogenic

## Outputs

- AUC per predictor per dataset subset (e.g. 0.579 PredPSI-SVR on SynonMut-psi; 0.508 noPSI on SynonMut-psi; 0.575 noPSI on SynonMut-complete)
- Scope-stratified AUC comparison on the 11024 out-of-MaxEntScan-scope mutations (PredPSI-SVR-noPSI 0.536 vs SPANR 0.501)
- Pairwise significance P-values comparing AUCs

## How to apply

Run each scorer on the identical filtered variant VCF: PredPSI-SVR with the -p option supplying experimental wild-type Ψ (SynonMut-psi setting) and without -p (PredPSI-SVR-noPSI on SynonMut-complete); SPANR via its SPIDEX ΔΨ; MaxEntScan as the summed absolute change |ΔMES5|+|ΔMES3|. Compute an AUC for each score against the binary splicing-affecting/pathogenic label, treating scores such as |ΔΨ| as ranking statistics. When one method covers only a subset (MaxEntScan cannot scan the full splice region), report a scope-split comparison over the 11024 mutations outside MaxEntScan's scanning scope so scores are compared on their common and disjoint strengths. Assess differences between paired AUCs with significance tests (e.g. P=0.036 for PredPSI-SVR vs SPANR) and report both relative differences (6.6% higher than MaxEntScan) and exact AUC values.

## Related tools

- **PredPSI-SVR** (Score under test; ranks synonymous mutations by absolute predicted ΔΨ, run with (-p) and without (-p) experimental wild-type Ψ) — https://github.com/chenkenbio/PredPSI-SVR
- **SPANR** (Competing baseline; provides SPIDEX ΔΨ scores for the same variants)
- **MaxEntScan** (Competing baseline; scored via the summed absolute change |ΔMES5|+|ΔMES3|, with limited scanning scope relative to PredPSI-SVR)
- **ANNOVAR** (Annotates the input VCF with hg19 ensGene annotations to enable the splice-proximity and region filtering before scoring)

## Examples

```
./main.sh synonmut.vcf -p synonmut.psi -o outdir  # then compute AUCs vs SPIDEX ΔΨ and |ΔMES5|+|ΔMES3| on the same filtered variants
```

## Evaluation signals

- Reproduced AUCs match reported values: PredPSI-SVR 0.579 on SynonMut-psi, noPSI 0.508 on SynonMut-psi and 0.575 on SynonMut-complete
- Out-of-scope subset reproduces PredPSI-SVR-noPSI AUC 0.536 vs SPANR 0.501 over the 11024 mutations outside MaxEntScan's scanning scope
- Significance outcomes replicate: PredPSI-SVR vs SPANR P=0.036 (SynonMut-psi); noPSI vs SPANR P=0.004 and vs MaxEntScan P=0.049 (SynonMut-complete); out-of-scope vs SPANR P=0.018
- Same variant set scored by all predictors after the 200 bp splice-site/intergenic filter — OUTPUT.psi row counts reflect filtered VCF
- Relative improvement sanity check: PredPSI-SVR is 6.6% higher than MaxEntScan on SynonMut-psi

## Limitations

- PredPSI-SVR with experimental Ψ (AUC 0.579) outperforms the no-Ψ mode (0.508 on SynonMut-psi); without experimental Ψ, performance drops substantially (PCCs dropped by about 0.1)
- AUCs near 0.5 (e.g. 0.501 for SPANR out-of-scope) indicate weak prioritization power on this task for all methods, so absolute performance, not just relative ranking, should be reported
- MaxEntScan covers a restricted scanning scope; comparisons restricted to its scope would exclude 11024 mutations where PredPSI-SVR still ranks variants

## Evidence

- [results] PredPSI-SVR achieved an AUC of 0.579 on SynonMut-psi, significantly better than SPANR (P-value=0.036) and 6.6% higher than MaxEntScan: [evidence span withheld — anchor status: not_found]
- [results] Score the same variants with SPANR (SPIDEX ΔΨ) and with MaxEntScan using the summed absolute change |ΔMES5|+|ΔMES3|.: [evidence span withheld — anchor status: not_found]
- [readme] The PredPSI-SVR will filter the VCF file at first to remove variants in intergenic regions or distant to splice sites (more than 200 bp ): "The `PredPSI-SVR` will filter the VCF file at first to remove variants in intergenic regions or distant to splice sites (more than 200 bp )"
- [intro] A further test indicated that the PredPSI-SVR is helpful for prioritizing pathogenic synonymous mutations: "Additional application indicated that PredPSI-SVR is helpful for prioritizing pathogenic synonymous mutations"
- [discussion] Another limitation of our method is its need for wild-type Ψ of exons. Without experimental Ψ, PCCs on the training set and test set dropped by about 0.1: "Another limitation of our method is its need for wild-type Ψ of exons. Without experimental Ψ, PCCs on the training set and test set dropped by about 0.1"
