---
name: chh-methylation-training-data-balancing
description: Use when building training data for plant 5mC methylation detection where
  CHH-positive samples are scarce and per-9-mer context balance between positive and
  negative classes is required.
when_to_use_negative:
- When processing CpG or CHG contexts, where the scarcity of methylation-positive
  samples and per-9-mer balancing criterion described for CHH does not apply
- When the positive and negative npz files are already balanced or the input is the
  final balanced_chh_training_samples.npz, not raw extract_hc_sites output
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_3474
tools:
- name: DeepPlant
  role: Produces the positive and negative CHH npz feature files via extract_hc_sites
    (kmer_size 13, CHH motif) that are the inputs to this balancing step
  repo: https://github.com/xiaochuanle/DeepPlant
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-025-58576-x
    title: Accurate cross-species 5mC detection for Oxford Nanopore sequencing in
      plants with DeepPlant
  source_content_hash: sha256:bd04e7ab658bd69f83a875afa68652ddebfec868004c7cd36248887a1321447e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/xiaochuanle/DeepPlant
  tools:
  - DeepPlant
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.45454545454545453
n_claims: 11
---

# chh-methylation-training-data-balancing

## Summary

Balances CHH methylation-positive and negative training samples across 9-mer sequence contexts when constructing the DeepPlant dataset from S. miltiorrhiza, S. tuberosum, and R. communis, so every retained 9-mer motif contributes paired positive/negative examples.

## When to use

Use when building training data for plant 5mC methylation detection where CHH-positive samples are scarce and per-9-mer context balance between positive and negative classes is required.

## When NOT to use

- When processing CpG or CHG contexts, where the scarcity of methylation-positive samples and per-9-mer balancing criterion described for CHH does not apply
- When the positive and negative npz files are already balanced or the input is the final balanced_chh_training_samples.npz, not raw extract_hc_sites output

## Inputs

- Positive CHH npz feature file from DeepPlant extract_hc_sites (uint8 site-info array plus data array)
- Negative CHH npz feature file from DeepPlant extract_hc_sites (uint8 site-info array plus data array)
- Tab-delimited site info strings encoding each sample's 9-mer context centered on the CHH motif
- k_max cap for samples per 9-mer context

## Outputs

- balanced_chh_training_samples.npz in the same npz layout (uint8 site-info array plus data array)
- Balanced dataset with 1:1 overall positive:negative ratio and equal per-9-mer counts in both classes

## How to apply

Load the positive and negative CHH npz feature files produced by DeepPlant's extract_hc_sites (kmer_size 13, CHH motif) and parse each sample's tab-delimited site-info string to obtain the 9-mer context centered on the CHH motif. Compute the intersection of the positive and negative 9-mer sets and keep only samples whose 9-mer falls in this intersection, so a 9-mer context is covered only if present in at least 50 positive CHH training samples with at least an equal number of negatives. Cap the number of samples per 9-mer at k_max in both classes so no single context dominates, then randomly subsample each class so every retained 9-mer has equal per-9-mer counts in positives and negatives. Save the result as balanced_chh_training_samples.npz in the same npz layout (uint8 site-info array plus data array) and validate an overall 1:1 positive:negative ratio plus per-9-mer count equality between classes.

## Related tools

- **DeepPlant** (Produces the positive and negative CHH npz feature files via extract_hc_sites (kmer_size 13, CHH motif) that are the inputs to this balancing step) — https://github.com/xiaochuanle/DeepPlant

## Examples

```
from numpy import load; pos = load('chh_pos.npz', allow_pickle=True); neg = load('chh_neg.npz', allow_pickle=True)  # parse 9-mer contexts from tab-delimited site-info strings
```

## Evaluation signals

- Output npz opens with the same layout as input (uint8 site-info array plus data array)
- Overall positive:negative ratio is exactly 1:1
- Every retained 9-mer context contributes equal numbers of positive and negative samples
- Each retained 9-mer is present in at least 50 positive CHH training samples with at least an equal number of negative samples
- No single 9-mer context exceeds k_max samples in either class

## Limitations

- Scarcity of high-methylation CHH samples, although partially mitigated, continues to pose challenges for model training and generalization
- Reliance on species with naturally abundant CHH methylation may not capture the full diversity of methylation patterns across all plant species
- Potential for overfitting to specific sequence patterns; careful model optimization and validation needed

## Evidence

- [methods] The training sample construction required a per-9-mer balancing criterion: [evidence span withheld — anchor status: not_found]
- [discussion] Training dataset coverage of 9-mer CHH contexts: "Our comprehensive training dataset now covers 97.2% of all possible 9-mer CHH contexts, averaging over 9225 samples per context"
- [readme] README motivation for 9-mer motif datasets: "we reanalyzed bisulfite sequencing (BS-seq) data to screen species with abundant high-methylation CHH sites, generating new datasets that cover diverse 9-mer motifs"
