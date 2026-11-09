---
name: pearson-correlation-benchmark-evaluation
description: Use when a regression model's continuous predictions (e.g. ΔΨ per variant)
  must be scored against held-out experimental values on an independent test set,
  as in CAGI-style blind challenges.
when_to_use_negative:
- The prediction task is classification rather than continuous regression on ΔΨ values
  — PCC is not the right metric
- Predictions and reference values come from different variant sets or cannot be joined
  one-to-one, since unmatched variants would distort the correlation
- The method cannot supply wild-type Ψ for the affected exons, because without experimental
  Ψ the PCCs on the training and test sets drop by about 0.1
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3524
- http://edamontology.org/topic_3320
tools:
- name: PredPSI-SVR
  role: Generates the per-variant ΔΨ predictions (OUTPUT.dpsi) that are scored against
    blind-test reference ΔΨ by PCC
  repo: https://github.com/chenkenbio/PredPSI-SVR
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1002/humu.23785
    title: Predicting the change of exon splicing caused by genetic variant using
      support vector regression
  source_content_hash: sha256:ba44202c9358f6dd326cbbfd089aa0bdcea4062ebab68e904277ba3287e47041
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/chenkenbio/PredPSI-SVR
  tools:
  - Perl
  - PredPSI-SVR
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3524
  - http://edamontology.org/topic_3320
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# pearson-correlation-benchmark-evaluation

## Summary

Evaluates a regression predictor (e.g. PredPSI-SVR predicted ΔΨ for splicing variants) against blind-test reference values by joining predictions with experimental measurements and computing the Pearson correlation coefficient (PCC), the metric used to rank entries in the CAGI 5 vex-seq challenge.

## When to use

Use when a regression model's continuous predictions (e.g. ΔΨ per variant) must be scored against held-out experimental values on an independent test set, as in CAGI-style blind challenges.

## When NOT to use

- The prediction task is classification rather than continuous regression on ΔΨ values — PCC is not the right metric
- Predictions and reference values come from different variant sets or cannot be joined one-to-one, since unmatched variants would distort the correlation
- The method cannot supply wild-type Ψ for the affected exons, because without experimental Ψ the PCCs on the training and test sets drop by about 0.1

## Inputs

- Predicted ΔΨ values per variant (e.g. OUTPUT.dpsi from PredPSI-SVR)
- Experimental/blind-test reference ΔΨ values for the same variants (e.g. TS1098 vex-seq blind test values)
- Optional per-variant Ψ predictions (OUTPUT.psi) for sanity-checking filtering

## Outputs

- Pearson correlation coefficient (PCC) between predicted and experimental ΔΨ across all blind-test variants
- Joined prediction-vs-reference table keyed by variant/exon identifier

## How to apply

Run the prediction pipeline (e.g. ./main.sh with the '-p' option supplying wild-type Ψ) on the blind-test variant VCF to produce per-variant ΔΨ predictions in OUTPUT.dpsi. Join the predicted ΔΨ values with the experimental ΔΨ reference values for the same variants by variant/exon identifier. Compute the Pearson correlation coefficient across all blind-test variants; for PredPSI-SVR on TS1098 (1098 variants on chromosomes 9 to X) the reported PCC is 0.566, versus 0.570 PCC in 10-fold cross-validation on the training set. Note that PredPSI-SVR also reports PCC improvement from 0.566 to 0.665 when comparing feature sets, so report the PCC alongside the variant count actually scored. Validate that filtering (intergenic regions, variants >200 bp from splice sites) reduced the variant count relative to the input VCF before computing the correlation.

## Related tools

- **PredPSI-SVR** (Generates the per-variant ΔΨ predictions (OUTPUT.dpsi) that are scored against blind-test reference ΔΨ by PCC) — https://github.com/chenkenbio/PredPSI-SVR

## Examples

```
./main.sh example/sample.vcf -p example/sample.psi -o example/outdir
```

## Evaluation signals

- Reproducing the PredPSI-SVR blind-test PCC of 0.566 on TS1098 (the rank-2 CAGI vex-seq result) is the acceptance criterion
- OUTPUT.psi contains fewer variants than the input VCF, consistent with intergenic and >200 bp splice-site-distance filtering
- Every scored variant has both a prediction and an experimental ΔΨ value after the join (no unmatched rows)
- Cross-validation PCC on the training set should be near 0.570, in line with the reported 10-fold CV result
- The 8-feature greedy-selected model should outperform reduced feature sets (PCC 0.566 vs 0.516 with fewer features)

## Limitations

- Performance depends on the variant set actually scored; the pipeline filters variants in intergenic regions or more than 200 bp from splice sites, so PCC covers only retained variants
- Without experimental wild-type Ψ, PCCs on training and test sets drop by about 0.1
- Small sample size (the ΔΨ-filtered analysis retained only 53 mutations) limits statistical power and precluded deep learning approaches
- Only Human BodyMap Project 2.0 RNA-seq data was used; many exons cannot be analyzed due to sequencing depth

## Evidence
