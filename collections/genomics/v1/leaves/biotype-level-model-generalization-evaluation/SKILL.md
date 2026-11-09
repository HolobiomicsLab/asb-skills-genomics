---
name: biotype-level-model-generalization-evaluation
description: Use when a trained raw-signal classifier (e.g., RISER mRNA/mtRNA models)
  must be tested on an independent held-out DRS dataset to measure biotype-level generalization
  via accuracy, precision, TPR and FPR.
when_to_use_negative:
- The reads were generated with a training or tuning dataset (human heart, GM24385,
  HEK293) — the skill specifically measures generalization on unseen data
- You need live enrichment/depletion decision-making during sequencing rather than
  offline model evaluation on pre-recorded signals
- The target RNA class lacks reliable 3'-end biotype labels; accurate lncRNA identification
  from the 3' end signal remains challenging per the paper
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0102
tools:
- name: Guppy (v4.0.14)
  role: Basecalls held-out fast5 files using FLO-MIN106/SQK-RNA002 options prior to
    biotype labeling
- name: minimap2 (v2.17)
  role: Maps basecalled reads to GENCODE release 34 (GRCh38.p13) with -ax map-ont
    --secondary=no to assign biotype labels
- name: samtools (v1.10)
  role: Filters alignments with -F 2324 to retain only high-confidence primary mappings
- name: BoostNano
  role: Trims the sequencing adapter and variable-length poly(A) tail from raw nanopore
    signals before testing
  repo: https://github.com/haotianteng/BoostNano
- name: RISER
  role: Provides the trained mRNA/mtRNA models and test script (riser/test.py, best_model.pth)
    used to classify held-out signals
  repo: https://github.com/comprna/riser
- name: PyTorch (v1.9.0)
  role: Runtime for building and testing the RISER models on a single NVIDIA Tesla
    V100 GPU
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-024-48673-8
    title: Biochemical-free enrichment or depletion of RNA classes in real-time during
      direct RNA sequencing with RISER
  source_content_hash: sha256:ca11b2dac55d8d6484b7bc3f3f4878a5aec8f10f9a6d1fd575c0b173ada0d2c6
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/haotianteng/BoostNano
  tools:
  - BoostNano
  - Guppy (v4.0.14)
  - PyTorch (v1.9.0)
  - RISER
  - minimap2 (v2.17)
  - samtools (v1.10)
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# biotype-level-model-generalization-evaluation

## Summary

Evaluates a trained RISER RNA-class model on an independent held-out nanopore direct RNA sequencing (DRS) dataset to test whether the model generalizes to data unseen during training, tuning, and architecture selection, and quantifies performance per RNA biotype.

## When to use

Use when a trained raw-signal classifier (e.g., RISER mRNA/mtRNA models) must be tested on an independent held-out DRS dataset to measure biotype-level generalization via accuracy, precision, TPR and FPR.

## When NOT to use

- The reads were generated with a training or tuning dataset (human heart, GM24385, HEK293) — the skill specifically measures generalization on unseen data
- You need live enrichment/depletion decision-making during sequencing rather than offline model evaluation on pre-recorded signals
- The target RNA class lacks reliable 3'-end biotype labels; accurate lncRNA identification from the 3' end signal remains challenging per the paper

## Inputs

- Held-out Nanopore fast5 raw signal files (HeLa poly(A)+ DRS, FLO-MIN106/SQK-RNA002)
- GENCODE reference transcriptome release 34 (GRCh38.p13) for high-confidence biotype labels
- Trained RISER model checkpoint (best_model.pth) and config yaml
- Pretrained BoostNano model (boostnano/models) for adapter/poly(A) trimming

## Outputs

- Per-read .tsv classification results from riser/test.py
- Model-level metrics: accuracy (~94% mRNA, ~99% mtRNA), precision, TPR, FPR
- Per-biotype accuracy breakdown (mRNA, mtRNA, lncRNA, 'other')

## How to apply

Basecall the held-out fast5 files (e.g., HeLa poly(A)+ DRS) with Guppy (v4.0.14, FLO-MIN106/SQK-RNA002 options), map to the GENCODE release 34 transcriptome (GRCh38.p13) with minimap2 (v2.17, -ax map-ont --secondary=no), and remove unmapped reads with samtools (v1.10, -F 2324). Discard reads whose primary mapping biotype does not match the most frequent secondary mapping biotype, so only high-confidence biotype labels remain (positive class: protein_coding for the mRNA model; negative classes per the paper's biotype lists). Trim the sequencing adapter and variable-length poly(A) tail from raw signals with BoostNano so evaluation isolates model performance from signal preprocessing effects. Run the trained models (riser/test.py with best_model.pth and config yaml under PyTorch v1.9.0) over all test signals (607,740 HeLa reads), then compute accuracy, precision, TPR and FPR from the per-read .tsv outputs, reporting accuracy per biotype with mRNA, mtRNA and lncRNA shown separately and remaining biotypes aggregated as 'other'. Accept the evaluation only if mRNA accuracy is approximately 94% and mtRNA accuracy approximately 99%, matching Fig. 3a-d.

## Related tools

- **Guppy (v4.0.14)** (Basecalls held-out fast5 files using FLO-MIN106/SQK-RNA002 options prior to biotype labeling)
- **minimap2 (v2.17)** (Maps basecalled reads to GENCODE release 34 (GRCh38.p13) with -ax map-ont --secondary=no to assign biotype labels)
- **samtools (v1.10)** (Filters alignments with -F 2324 to retain only high-confidence primary mappings)
- **BoostNano** (Trims the sequencing adapter and variable-length poly(A) tail from raw nanopore signals before testing) — https://github.com/haotianteng/BoostNano
- **RISER** (Provides the trained mRNA/mtRNA models and test script (riser/test.py, best_model.pth) used to classify held-out signals) — https://github.com/comprna/riser
- **PyTorch (v1.9.0)** (Runtime for building and testing the RISER models on a single NVIDIA Tesla V100 GPU)

## Examples

```
python boostnano_eval.py -i INPUT_FAST5_FOLDER -o OUTPUT_FOLDER -m MODEL_FOLDER --replace
```

## Evaluation signals

- mRNA model accuracy on the held-out HeLa test set is approximately 94%
- mtRNA model accuracy on the held-out HeLa test set is approximately 99%
- Computed precision, TPR and FPR match Fig. 3a-d (e.g., mRNA precision ~0.99, TPR ~0.94, FPR ~0.05)
- Every test read carries a high-confidence biotype label: primary mapping biotype equals the most frequent secondary mapping biotype
- Per-biotype accuracy table reports mRNA, mtRNA, and lncRNA separately with remaining biotypes aggregated as 'other'

## Limitations

- Accurate lncRNA identification from the 3' end signal remains challenging, limiting what a 3'-end-signal classifier can learn for this class
- Cell type-specific lncRNA expression makes it harder for predictive models to generalize to new samples
- Using larger training datasets, especially for lowly expressed RNA classes, would be needed to improve sensitivity to rare RNAs

## Evidence

- [methods] Guppy basecalling of held-out fast5 files: "Fast5 files were basecalled with Guppy (v4.0.14) using options –flowcell FLO-MIN106 –kit SQK-RNA002"
- [methods] Mapping to GENCODE release 34 with minimap2: "mapped to the GENCODE reference transcriptome (release 34, assembly GRCh38.p13) with minimap233 (v2.17) using options -ax map-ont --secondary=no"
- [readme] Trimming adapter and poly(A) tail before testing: [evidence span withheld — anchor status: external_doc]
- [discussion] lncRNA generalization caveat: "At present, accurate lncRNA identification from the 3′ end signal remains challenging"
- [readme] What the RISER classifier consumes: "RISER accurately classifies RNA molecules live during sequencing, directly from the first ~70-280nt-worth of raw nanopore signals with a convolutional neural network"
