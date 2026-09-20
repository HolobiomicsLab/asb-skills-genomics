---
name: benchmark-roc-auc-comparison-m6a-prediction
description: Use when site-level m6A probabilities from competing Nanopore DRS methods
  are available along with binary ground-truth labels, and a quantitative AUC/AUPR
  comparison such as RedNano (0.8511) vs. m6Anet (0.8230) is required.
when_to_use_negative:
- Input is raw FAST5/TSV read-level data that has not yet been aggregated to site
  level and coverage-filtered ($4>=20) — run feature extraction and aggregation first
- No binary ground-truth labels (0/1) exist for the m6A sites, so AUC/AUPR cannot
  be computed against truth
- The task is read-level modification calling rather than transcriptome-level site
  benchmarking between methods
edam_operation: http://edamontology.org/operation_2945
tools:
- name: RedNano
  role: Supervised method under evaluation; its predict.py emits site-level modification
    probabilities (comb_basecall_raw model, signal_lens 65, hidden_size 128) benchmarked
    by AUC/AUPR
  repo: https://github.com/Derryxu/RedNano
- name: scikit-learn
  role: Dependency of the RedNano environment used for computing the AUC/AUPR benchmark
    metrics on site-level predictions
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bioinformatics/btae375
    title: RNA m6A detection using raw current signals and basecalling errors from
      Nanopore direct RNA sequencing reads
  source_content_hash: sha256:019fd3edf40230bc07100b7ae3dd57d1da0aae8ca79b0b89e80207b74bfd6c4c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Derryxu/RedNano
  tools:
  - CUDA
  - Python
  - RedNano
  - h5py
  - jvarkit
  - minimap2
  - ont-tombo
  - samtools
  - scikit-learn
  edam_operation: http://edamontology.org/operation_2945
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5882352941176471
n_claims: 17
---

# benchmark-roc-auc-comparison-m6a-prediction

## Summary

Benchmark site-level m6A prediction methods (e.g. RedNano vs. m6Anet) on Nanopore direct RNA sequencing data by comparing area under the ROC curve (AUC) and area under the precision-recall curve (AUPR) against ground-truth labels.

## When to use

Use when site-level m6A probabilities from competing Nanopore DRS methods are available along with binary ground-truth labels, and a quantitative AUC/AUPR comparison such as RedNano (0.8511) vs. m6Anet (0.8230) is required.

## When NOT to use

- Input is raw FAST5/TSV read-level data that has not yet been aggregated to site level and coverage-filtered ($4>=20) — run feature extraction and aggregation first
- No binary ground-truth labels (0/1) exist for the m6A sites, so AUC/AUPR cannot be computed against truth
- The task is read-level modification calling rather than transcriptome-level site benchmarking between methods

## Inputs

- RedNano site-level prediction file (test.features.sort.aggre.c20.results.tsv with transcript, pos, strand, prob, label columns)
- Competing method (m6Anet) site-level m6A probabilities on the same dataset
- Binary ground-truth m6A labels (0/1) per site, e.g. from HEK293T DRS data (PRJEB40872)
- Coverage-filtered site-level feature table (test.features.sort.aggre.c20.tsv, column-4 coverage >= 20)

## Outputs

- Site-level m6A probability table per method (transcript, pos, strand, prob, label)
- AUC and AUPR values per method and motif context (RRACH and DRACH), e.g. RedNano 0.8511 vs. m6Anet 0.8230 for RRACH
- Comparative benchmark verdict (whether RedNano attains higher AUC and AUPR than the competitor)

## How to apply

Prepare inputs by running the RedNano pipeline on the DRS dataset: preprocess FAST5 reads (multi_to_single_fast5, guppy v3.1.5 with rna_r9.4.1_70bps_hac.cfg, tombo v1.5.1 resquiggle into RawGenomeCorrected_001), map reads with minimap2 (v2.17, -ax map-ont), convert BAM to TSV with jvarkit sam2tsv, then extract combined raw-signal and basecalling-error features (extract_features.py with -k 5, -s 65, -n 30). Aggregate read-level features to site level with aggre_features_to_site_level.py and retain only sites with column-4 coverage >= 20 via the awk '$4>=20' filter to produce test.features.sort.aggre.c20.tsv. Run RedNano's predict.py (--model_type comb_basecall_raw, --signal_lens 65, --hidden_size 128, --num_iterations 1000) to obtain per-site modification probabilities, and run the competing method (m6Anet) on the same data. Score both methods' site-level probabilities against ground-truth 0/1 labels, computing AUC and AUPR per motif context (RRACH and DRACH); accept RedNano as superior only when both AUC and AUPR exceed the competitor's on the same dataset.

## Related tools

- **RedNano** (Supervised method under evaluation; its predict.py emits site-level modification probabilities (comb_basecall_raw model, signal_lens 65, hidden_size 128) benchmarked by AUC/AUPR) — https://github.com/Derryxu/RedNano
- **scikit-learn** (Dependency of the RedNano environment used for computing the AUC/AUPR benchmark metrics on site-level predictions)

## Examples

```
CUDA_VISIBLE_DEVICES=0 python scripts/predict.py --input_file test.features.sort.aggre.c20.tsv --model models/mil_allspecies_model_states.pt --num_iterations 1000 --batch_size 1 --signal_lens 65 --model_type comb_basecall_raw --output_file test.features.sort.aggre.c20.results.tsv
```

## Evaluation signals

- RedNano should yield AUC ≈ 0.8511 (RRACH) and ≈ 0.8513 (DRACH), exceeding m6Anet's 0.8230 and 0.8187 on the human HEK293T dataset
- Prediction output file must contain transcript, pos, strand, prob, label columns with probabilities in [0,1]
- All evaluated sites must satisfy the coverage filter (column-4 >= 20) present in test.features.sort.aggre.c20.tsv
- Both methods must be evaluated on the identical site set and labels so the AUC/AUPR comparison is paired and fair
- RedNano should also show higher AUPR than m6Anet, not just higher AUC, to satisfy the acceptance criterion

## Limitations

- AUC/AUPR conclusions are specific to datasets where coverage >= 20 per site was enforceable; low-coverage sites are excluded by the awk '$4>=20' filter
- Reported metrics depend on the exact RedNano configuration (comb_basecall_raw, signal_lens 65, hidden_size 128) and comparable preprocessing for the competitor (m6Anet)
- The comparison is transcriptome-level on the tested species (synthesized, Arabidopsis, human); performance may differ on other organisms or motif contexts

## Evidence

- [other] RedNano achieves higher AUCs and AUPRs than m6Anet, with AUC of 0.8511 versus 0.8230 for RRACH and 0.8513 versus 0.8187 for DRACH.: [evidence span withheld — anchor status: not_found]
- [abstract] surpasses existing methods with higher AUC/AUPR across all datasets: "The results demonstrate that RedNano surpasses existing methods by achieving higher area under the ROC curve (AUC) and area under the precision-recall curve (AUPRs) in all three datasets"
- [readme] coverage filter producing c20 test table: "awk 'BEGIN{ FS=OFS="\t" } $4>=20 {print $0}' test.features.sort.aggre.tsv > test.features.sort.aggre.c20.tsv"
- [readme] site-level prediction output format: "Output: Modified probability for each site are in the following format: | transcript | pos | strand | prob | label |"
