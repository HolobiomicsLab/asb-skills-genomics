---
name: cross-species-chh-training-set-curation
description: Use when constructing a CHH training set for a nanopore 5mC deep learning
  model where methylation-positive CHH samples are scarce and per-9-mer context balance
  between positive and negative classes is required.
when_to_use_negative:
- Input features are not from a CHH context run (e.g. CpG or CHG motifs), where positive-sample
  scarcity and the 9-mer balancing criterion do not apply the same way
- The npz files have not yet been produced by extract_hc_sites — raw pod5/BAM inputs
  cannot be balanced directly
- You only need species screening or dataset generation, not class balancing of already-extracted
  sites
edam_operation: http://edamontology.org/operation_3802
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3295
tools:
- name: DeepPlant
  role: Provides the extract_hc_sites step that produces the positive and negative
    CHH npz feature files consumed by the balancing algorithm; the balanced dataset
    is used to train DeepPlant
  repo: https://github.com/xiaochuanle/DeepPlant
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-025-58576-x
    title: Accurate cross-species 5mC detection for Oxford Nanopore sequencing in
      plants with DeepPlant
  source_content_hash: sha256:467c6ed1b386b2e010b7d65934d11a6e767b6a3cbd5983e8ec9ae35cb09f3285
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/xiaochuanle/DeepPlant
  tools:
  - DeepPlant
  edam_operation: http://edamontology.org/operation_3802
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3295
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4166666666666667
n_claims: 12
---

# cross-species-chh-training-set-curation

## Summary

Curates a class-balanced CHH methylation training dataset for DeepPlant by keeping only 9-mer sequence contexts that have at least 50 positive samples and at least as many negatives, then subsampling so every retained 9-mer contributes equal positive and negative samples.

## When to use

Use when constructing a CHH training set for a nanopore 5mC deep learning model where methylation-positive CHH samples are scarce and per-9-mer context balance between positive and negative classes is required.

## When NOT to use

- Input features are not from a CHH context run (e.g. CpG or CHG motifs), where positive-sample scarcity and the 9-mer balancing criterion do not apply the same way
- The npz files have not yet been produced by extract_hc_sites — raw pod5/BAM inputs cannot be balanced directly
- You only need species screening or dataset generation, not class balancing of already-extracted sites

## Inputs

- Positive CHH npz feature file from DeepPlant extract_hc_sites (uint8 site-info array plus data array, kmer_size 13, CHH motif)
- Negative CHH npz feature file from DeepPlant extract_hc_sites (same layout)
- k_max cap parameter for maximum samples per 9-mer context

## Outputs

- balanced_chh_training_samples.npz in the same npz layout as the inputs, with overall 1:1 positive:negative ratio and per-9-mer class balance

## How to apply

Load the positive and negative CHH npz feature files produced by DeepPlant's extract_hc_sites (run with kmer_size 13 on the CHH motif) and parse the tab-delimited site-info strings to recover each sample's 9-mer context centered on the CHH motif. Compute the intersection of the positive and negative 9-mer sets and retain only samples whose 9-mer falls in that intersection, enforcing the criterion that a context is covered only if present in at least 50 positive training samples with at least an equal number of negatives. Cap samples per 9-mer at k_max in both classes so no single motif dominates, then randomly subsample each class so every retained 9-mer has equal per-9-mer counts in positives and negatives. Save the result as balanced_chh_training_samples.npz in the same npz layout (uint8 site-info array plus data array). Rationale: CHH methylation depends on RdDM, so high-methylation positives are rare; screening species with abundant high-methylation CHH sites (S. miltiorrhiza, S. tuberosum, R. communis) plus per-context pairing ensures the model sees paired positive/negative examples for each 9-mer motif.

## Related tools

- **DeepPlant** (Provides the extract_hc_sites step that produces the positive and negative CHH npz feature files consumed by the balancing algorithm; the balanced dataset is used to train DeepPlant) — https://github.com/xiaochuanle/DeepPlant

## Examples

```
python balance_chh.py --pos pos_chh.npz --neg neg_chh.npz --kmer 9 --min_pos 50 --k_max 5000 --out balanced_chh_training_samples.npz
```

## Evaluation signals

- Output npz has an overall 1:1 positive:negative sample ratio
- Every retained 9-mer context contributes equal numbers of positive and negative samples (per-9-mer balance check)
- Each retained 9-mer has at least 50 positive training samples and at least as many negatives before subsampling
- Output file preserves the input npz layout: uint8 site-info array plus data array
- Coverage check: the resulting training set should approach the paper's 97.2% coverage of all possible 9-mer CHH contexts (averaging over 9225 samples per context)

## Limitations

- Scarcity of high-methylation CHH samples, although partially mitigated, continues to pose challenges for model training and generalization
- Reliance on species with naturally abundant CHH methylation may not capture the full diversity of methylation patterns across all plant species
- In vitro enzyme-catalyzed DNA methylation is ineffective for generating CHH positives because CHH methylation depends on RdDM (non-coding RNA guidance and DRM1/DRM2 methyltransferases)

## Evidence

- [abstract] Scarcity of high-methylation CHH training samples addressed by screening species: "We address the scarcity of methylation-positive CHH training samples through screening species with abundant high-methylation CHH sites using bisulfite-sequencing"
- [discussion] Training dataset 9-mer coverage: "Our comprehensive training dataset now covers 97.2% of all possible 9-mer CHH contexts, averaging over 9225 samples per context"
- [methods] extract_hc_sites CLI usage: "Usage: extract_hc_sites [--help] [--version] pod5_dir bam_path reference_path ref_type write_dir pos neg kmer_size num_workers"
- [discussion] CHH methylation RdDM dependency limits artificial positive generation: "CHH methylation depends on RNA-directed DNA methylation (RdDM), which requires non-coding RNA guidance and the involvement of methyltransferases (DRM1 and DRM2)"
