---
name: per-site-methylation-frequency-binarization
description: Use when preparing per-read (output.per_read) or per-site (output.per_site)
  methylation predictions for binarized benchmarking against discrete bisulfite ground-truth
  labels.
when_to_use_negative:
- Sites with 20–80% methylation frequency, which EpiQC reported to have poor concordance
  between ONT and short-read methods and should be excluded from evaluation
- Low-coverage sites (<10X for HG002-4 ground truth, <5X for NIH3T3), where binarized
  labels are unreliable
- Intermediate probability per-read predictions where the user wishes to retain continuous
  confidence rather than a binary call
edam_operation: http://edamontology.org/operation_3802
edam_topics:
- http://edamontology.org/topic_3674
- http://edamontology.org/topic_3295
tools:
- name: DeepMod2
  role: Produces per-read methylation probability scores and per-site aggregated methylation
    frequencies that are binarized by this skill
  repo: https://github.com/WGLab/DeepMod2
- name: modkit
  role: Post-processing tool for modified-base BAM (MM/ML tags) used to summarize
    per-site methylation counts in bedMethyl format
  repo: https://github.com/nanoporetech/modkit
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-024-45778-y
    title: A signal processing and deep learning framework for methylation detection
      using Oxford Nanopore sequencing
  source_content_hash: sha256:b00c6241d755cf5269ef44a5b82baa23c24a034573c9097b61de34e3b0977cdd
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/WGLab/DeepMod2
  tools:
  - DeepMod2
  - modkit
  edam_operation: http://edamontology.org/operation_3802
  edam_topics:
  - http://edamontology.org/topic_3674
  - http://edamontology.org/topic_3295
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# per-site-methylation-frequency-binarization

## Summary

Binarizes 5mC methylation calls from Oxford Nanopore methylation callers (e.g., DeepMod2) by applying a 0.5 probability-score threshold to per-read predictions and a ≥50% methylation-frequency cutoff to per-site predictions, enabling F1-score evaluation against short-read bisulfite ground truth.

## When to use

Use when preparing per-read (output.per_read) or per-site (output.per_site) methylation predictions for binarized benchmarking against discrete bisulfite ground-truth labels.

## When NOT to use

- Sites with 20–80% methylation frequency, which EpiQC reported to have poor concordance between ONT and short-read methods and should be excluded from evaluation
- Low-coverage sites (<10X for HG002-4 ground truth, <5X for NIH3T3), where binarized labels are unreliable
- Intermediate probability per-read predictions where the user wishes to retain continuous confidence rather than a binary call

## Inputs

- DeepMod2 per-read output TSV (output.per_read) with probability scores per CpG per read
- DeepMod2 per-site output (output.per_site / output.per_site.aggregated) with methylation frequencies
- Modified-base BAM file with MM/ML tags from DeepMod2
- Short-read bisulfite ground truth (EpiQC, GEO GSE186383) restricted to high-confidence CpGs

## Outputs

- Binarized per-read methylated/unmethylated labels per CpG per read
- Binarized per-site methylated/unmethylated labels per CpG
- Coordinate-strand matched prediction/ground-truth pairs for precision, recall, and F1 computation

## How to apply

Binarize DeepMod2 per-read predictions using the 0.5 probability-score threshold so each CpG on each read is labelled methylated or unmethylated. For per-site calls, aggregate per-read predictions into a per-site methylation frequency (percentage of total reads with methylated cytosine), then label a CpG methylated when frequency ≥50% and unmethylated otherwise. Match binarized CpG calls to ground-truth CpGs by genomic coordinate and strand, excluding sites with 20–80% methylation from evaluation per EpiQC concordance findings. This binary labelling mirrors the scheme used to train DeepMod2, where sites with ≥90% methylation were positive and <10% were negative labels at coverage ≥10X.

## Related tools

- **DeepMod2** (Produces per-read methylation probability scores and per-site aggregated methylation frequencies that are binarized by this skill) — https://github.com/WGLab/DeepMod2
- **modkit** (Post-processing tool for modified-base BAM (MM/ML tags) used to summarize per-site methylation counts in bedMethyl format) — https://github.com/nanoporetech/modkit

## Examples

```
python deepmod2 detect --bam reads.bam --input INPUT_DIR --model MODEL --file_type FILE_TYPE --threads NUM_THREADS --ref ref.fa --output MOD_CALLS
```

## Evaluation signals

- Per-read F1 against bisulfite ground truth falls in the 95.7–97.1% range on R9.4.1 Ashkenazim trio (HG002, HG003, HG004) datasets
- Overall per-read F1 ~95% and per-site F1 ~99% on human cell lines against short-read sequencing ground truth
- Correlation between binarized/aggregate methylation calls and short-read sequencing exceeds r > 0.95
- Excluded evaluation sites (20–80% methylation) are absent from the matched prediction/ground-truth comparison set
- All matched CpGs agree on coordinate and strand before precision/recall/F1 computation

## Limitations

- Filtering intermediate per-read probability predictions with stricter thresholds reduces coverage of per-site predictions and statistical power, especially for low-coverage samples
- DeepMod2 models show a slight bias to overestimate methylation at some CpG sites, possibly from residual heterogeneity in training labels
- Binarized per-site labels discard per-read methylation probabilities, which current differential methylation tools ignore anyway but which carry long-read co-occurrence information
- Ground-truth truth sets only cover extreme methylation classes (≥90%/<10% per-read; ≥80%/<20% per-site), so binarized accuracy is not representative of intermediate-methylation sites

## Evidence

- [results] CpG sites with methylation frequency ≥50% were labelled as methylated and labelled unmethylated otherwise: "CpG sites with methylation frequency ≥50% were labelled as methylated and labelled unmethylated otherwise"
- [results] A threshold of 0.5 for probability score is used to predict a CpG site as methylated: "A threshold of 0.5 for probability score is used to predict a CpG site as methylated, however, user specified thresholds can be used to exclude intermediate probability predictions"
- [results] all per-read predictions for a site are combined to calculate the percentage of total reads with methylated cytosine: "In the post-processing step, all per-read predictions for a site are combined to calculate the percentage of total reads with methylated cytosine"
- [results] the ground truth only included sites that were mostly methylated (≥80% methylation) and mostly unmethylated (<20% methylation): "the ground truth only included sites that were mostly methylated (≥80% methylation) and mostly unmethylated (<20% methylation) with ample coverage (≥10X for HG002-4 and ≥5X for NIH3T3)"
- [results] Coverage ≥10 Positive Label: ≥90% methylation Negative Label: <10% methylation: [evidence span withheld — anchor status: not_found]
