---
name: cross-validation-pcc-evaluation
description: Use when you need to score a ΔΨ regression model on the CAGI-5 vex-seq
  data by 10-fold cross-validation on TR957 plus a blind test on TS1098, comparing
  predicted vs. measured delta-PSI via PCC.
when_to_use_negative:
- There are no measured/observed ΔΨ values for the variants, so a Pearson correlation
  against ground truth cannot be computed.
- The task is variant classification or harmfulness ranking (e.g. the SilVA-style
  synonymous-variant task) rather than regression on continuous ΔΨ values.
- The variant set is the heavily filtered 53-mutation subset used only for the 2-standard-deviation
  sensitivity analysis, where PCC estimates are unstable.
edam_operation: http://edamontology.org/operation_3658
edam_topics:
- http://edamontology.org/topic_3474
- http://edamontology.org/topic_3512
- http://edamontology.org/topic_2269
tools:
- name: PredPSI-SVR
  role: Pipeline whose ΔΨ predictions are evaluated by 10-fold CV PCC and blind-test
    PCC
  repo: https://github.com/chenkenbio/PredPSI-SVR
- name: libsvm
  role: Support vector regression library used to train the model with 10-fold cross-validation
    on TR957
provenance:
  source_task_ids:
  - task_006
  - task_005
  source_papers:
  - doi: 10.1002/humu.23785
    title: Predicting the change of exon splicing caused by genetic variant using
      support vector regression
  source_content_hash: sha256:bb23bb4e2277de8b9e38796720e618d6511ca31208374db5f0677534daf78109
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/chenkenbio/PredPSI-SVR
  tools:
  - Perl
  - PredPSI-SVR
  - Python 3
  - libsvm
  - samtools
  edam_operation: http://edamontology.org/operation_3658
  edam_topics:
  - http://edamontology.org/topic_3474
  - http://edamontology.org/topic_3512
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# cross-validation-pcc-evaluation

## Summary

Evaluates ΔΨ-prediction regression models (e.g. PredPSI-SVR trained with libsvm) by 10-fold cross-validation on the vex-seq training set (TR957) and on an independent blind test set (TS1098), reporting the Pearson correlation coefficient (PCC) between predicted and measured ΔΨ.

## When to use

Use when you need to score a ΔΨ regression model on the CAGI-5 vex-seq data by 10-fold cross-validation on TR957 plus a blind test on TS1098, comparing predicted vs. measured delta-PSI via PCC.

## When NOT to use

- There are no measured/observed ΔΨ values for the variants, so a Pearson correlation against ground truth cannot be computed.
- The task is variant classification or harmfulness ranking (e.g. the SilVA-style synonymous-variant task) rather than regression on continuous ΔΨ values.
- The variant set is the heavily filtered 53-mutation subset used only for the 2-standard-deviation sensitivity analysis, where PCC estimates are unstable.

## Inputs

- TR957 vex-seq challenge training dataset with observed ΔΨ values
- TS1098 vex-seq independent blind test set with observed ΔΨ values
- Predicted ΔΨ values from the trained SVR model (e.g. OUTPUT.dpsi)
- libsvm training/feature files for the SVR model

## Outputs

- 10-fold cross-validation PCC on the training set
- Independent-test PCC on the blind test set
- Metrics file (e.g. pcc_results.csv) recording both PCC values

## How to apply

Train the libsvm support vector regression model on the TR957 vex-seq training variants and run 10-fold cross-validation, then apply the trained model to the TS1098 independent test set to write predicted ΔΨ values (e.g. to OUTPUT.dpsi). For each evaluation, compute the Pearson correlation coefficient (PCC) between predicted and observed ΔΨ and record both values (e.g. in pcc_results.csv or a metrics file). Compare the reproduced numbers against the reported baselines: full PredPSI-SVR 0.570 (10-fold CV) and 0.566 (blind test), the noPSI variant 0.525/0.479, and the 6-feature server model 0.590/0.577. Apply the same variant-filtering convention before evaluation (VCF variants in intergenic regions or more than 200 bp from splice sites are removed), and use the ΔΨ 2-standard-deviation filter only when reproducing the 53-mutation sensitivity analysis. Treat matched PCCs within numerical tolerance as validation that the pipeline was applied correctly.

## Related tools

- **PredPSI-SVR** (Pipeline whose ΔΨ predictions are evaluated by 10-fold CV PCC and blind-test PCC) — https://github.com/chenkenbio/PredPSI-SVR
- **libsvm** (Support vector regression library used to train the model with 10-fold cross-validation on TR957)

## Examples

```
svm-train -v 10 TR957_features.txt; ./main.sh example/sample.vcf -o example/outdir  # OUTPUT.dpsi -> PCC vs. observed ΔΨ
```

## Evaluation signals

- Reproduced 10-fold CV PCC on TR957 should match the reported 0.570 for the full PredPSI-SVR model within numerical tolerance.
- Reproduced blind-test PCC on TS1098 should match the reported 0.566 (2nd place in the CAGI-5 vex-seq challenge).
- noPSI variant (run without the '-p' option) should yield CV PCC 0.525 and independent-test PCC 0.479.
- Final 6-feature server model (priPhyloP and minDistTSS removed) should yield CV PCC 0.590 and blind PCC 0.577, showing the 8-feature greedy selection is not globally optimal.
- The number of evaluated variants in the output should be fewer than the input VCF when the 200 bp splice-site distance filter applies.

## Limitations

- Without experimental wild-type Ψ as input (noPSI mode), PCCs on the training and test sets drop by about 0.1.
- Small sample size prevented the use of more powerful algorithms like deep learning and hindered effective use of branch point sequence and pre-mRNA secondary structure features.
- Only Human BodyMap Project 2.0 RNA-seq data were used; many exons cannot be analyzed due to sequencing depth, and tissue-specific Ψ is not incorporated.

## Evidence

- [abstract] The trained model achieved a Pearson correlation coefficient (PCC) of 0.570 in the 10-fold cross-validation: "The trained model achieved a Pearson correlation coefficient (PCC) of 0.570 in the 10-fold cross-validation"
- [discussion] PCCs of 0.570 and 0.566 for the 10-fold CV on the training dataset and strictly independent test set: "PCCs of 0.570 and 0.566 for the 10-fold CV on the training dataset and strictly independent test set"
- [discussion] By removing ΔΨ with absolute values less than two times of the standard deviation, we observed improvement of correlations for all methods on the remained 53 mutations: "By removing ΔΨ with absolute values less than two times of the standard deviation, we observed improvement of correlations for all methods on the remained 53 mutations"
- [readme] PredPSI-SVR will filter the VCF file at first to remove variants in intergenic regions or distant to splice sites: "The `PredPSI-SVR` will filter the VCF file at first to remove variants in intergenic regions or distant to splice sites (more than 200 bp )"
- [discussion] Without experimental Ψ, PCCs on the training set and test set dropped by about 0.1: "Without experimental Ψ, PCCs on the training set and test set dropped by about 0.1"
