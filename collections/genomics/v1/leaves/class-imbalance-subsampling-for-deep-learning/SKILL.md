---
name: class-imbalance-subsampling-for-deep-learning
description: Use when a deep-learning training set (e.g., DeepPlant CHH npz features
  from extract_hc_sites) is skewed toward methylation-negative samples and you need
  paired positive/negative representation per 9-mer motif context.
when_to_use_negative:
- You are building training data for CpG or CHG contexts, where methylation-positive
  samples are abundant and no scarcity-driven balancing is needed
- The npz site info strings lack the tab-delimited 9-mer context centered on the motif,
  so per-context parity cannot be computed
- You are running methylation calling/inference on new data rather than constructing
  a balanced training dataset
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_3474
- http://edamontology.org/topic_0091
tools:
- name: DeepPlant
  role: Its extract_hc_sites step produces the positive/negative CHH npz feature files
    that this balancing procedure consumes
  repo: github.com/xiaochuanle/DeepPlant
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-025-58576-x
    title: Accurate cross-species 5mC detection for Oxford Nanopore sequencing in
      plants with DeepPlant
  source_content_hash: sha256:cfc0f31d348767a50c039c40b0e0231cfe3dd7b7874c01e2ec7ded140d1b05dd
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - DeepPlant
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_3474
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# class-imbalance-subsampling-for-deep-learning

## Summary

Balances methylation-positive and methylation-negative CHH training samples for DeepPlant by per-9-mer-context subsampling of npz feature files, addressing the scarcity of CHH-positive nanopore samples for cross-species 5mC model training.

## When to use

Use when a deep-learning training set (e.g., DeepPlant CHH npz features from extract_hc_sites) is skewed toward methylation-negative samples and you need paired positive/negative representation per 9-mer motif context.

## When NOT to use

- You are building training data for CpG or CHG contexts, where methylation-positive samples are abundant and no scarcity-driven balancing is needed
- The npz site info strings lack the tab-delimited 9-mer context centered on the motif, so per-context parity cannot be computed
- You are running methylation calling/inference on new data rather than constructing a balanced training dataset

## Inputs

- Positive CHH npz feature file from DeepPlant extract_hc_sites (uint8 site-info array plus data array)
- Negative CHH npz feature file from DeepPlant extract_hc_sites (uint8 site-info array plus data array)
- kmer_size parameter (13 in this workflow) and per-9-mer cap k_max

## Outputs

- balanced_chh_training_samples.npz with an overall 1:1 positive:negative ratio and equal per-9-mer counts in both classes, in the same npz layout (uint8 site-info array plus data array)

## How to apply

Load the positive and negative CHH npz feature files produced by DeepPlant's extract_hc_sites (run with kmer_size 13 on the CHH motif) and parse each sample's tab-delimited site info string to recover the 9-mer context centered on the CHH motif. Compute the intersection of the positive and negative 9-mer sets and keep only samples whose 9-mer falls in this intersection, requiring each retained context to have at least 50 positive training samples and at least an equal number of negatives. Cap samples per 9-mer at k_max in both classes so no single motif dominates, then randomly subsample each class so every retained 9-mer has equal per-9-mer counts in positives and negatives. Save the result as balanced_chh_training_samples.npz in the same npz layout as the inputs (uint8 site-info array plus data array). This is necessary because CHH methylation-positive samples are scarce, so species with abundant high-methylation CHH sites (e.g., S. miltiorrhiza, S. tuberosum, R. communis) were screened via BS-seq to build coverage across 9-mer motifs.

## Related tools

- **DeepPlant** (Its extract_hc_sites step produces the positive/negative CHH npz feature files that this balancing procedure consumes) — github.com/xiaochuanle/DeepPlant

## Examples

```
extract_hc_sites pod5_dir bam_path reference.fa plant write_dir pos.npz neg.npz 13 16
```

## Evaluation signals

- Output balanced_chh_training_samples.npz shows an overall 1:1 positive:negative sample ratio
- Every retained 9-mer context contributes equal numbers of positive and negative samples (per-9-mer parity check)
- Each retained 9-mer was present in at least 50 positive CHH training samples before subsampling
- Output preserves the input npz layout: uint8 site-info array plus data array
- Sanity range: the full training dataset covers 97.2% of all possible 9-mer CHH contexts, averaging over 9225 samples per context

## Limitations

- Scarcity of high-methylation CHH samples remains only partially mitigated and continues to challenge model training and generalization
- Reliance on species with naturally abundant CHH methylation may not capture the full diversity of methylation patterns across all plant species
- In vitro enzyme-catalyzed methylation is ineffective for generating CHH positive samples because CHH methylation depends on RdDM (non-coding RNA guidance plus DRM1/DRM2)
- Potential for overfitting to specific sequence patterns, underscoring the need for careful model optimization and validation

## Evidence

- [abstract] We address the scarcity of methylation-positive CHH training samples through screening species with abundant high-methylation CHH sites using bisulfite-sequencing: "We address the scarcity of methylation-positive CHH training samples through screening species with abundant high-methylation CHH sites using bisulfite-sequencing"
- [discussion] Our comprehensive training dataset now covers 97.2% of all possible 9-mer CHH contexts, averaging over 9225 samples per context: "Our comprehensive training dataset now covers 97.2% of all possible 9-mer CHH contexts, averaging over 9225 samples per context"
- [discussion] The scarcity of high-methylation CHH samples, although partially mitigated in this study, continues to pose challenges for model training and generalization: "The scarcity of high-methylation CHH samples, although partially mitigated in this study, continues to pose challenges for model training and generalization"
- [methods] Usage: extract_hc_sites [--help] [--version] pod5_dir bam_path reference_path ref_type write_dir pos neg kmer_size num_workers: "Usage: extract_hc_sites [--help] [--version] pod5_dir bam_path reference_path ref_type write_dir pos neg kmer_size num_workers"
