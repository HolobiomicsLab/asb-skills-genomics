---
name: variant-calling-metrics-computation
description: Use when benchmarking germline SNV/small-indel callers (e.g., Clairvoyante
  vs. Nanopolish) on ONT reads against a GIAB truth set, and you need F1, FPR, FNR
  and precision at a restricted site set such as 1KGp3 common variants.
when_to_use_negative:
- The goal is somatic or trio-based variant calling — the paper explicitly plans but
  does not support these.
- Evaluating indel calling on ONT data as the primary metric — Clairvoyante's ONT
  indel F1-score remains around 50%, so these metrics will not reflect acceptable
  indel performance.
- Novel sequencing technologies/chemistries lacking a high-quality GIAB-style training
  dataset — Clairvoyante cannot be applied there, and metrics would be uninterpretable.
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_0625
- http://edamontology.org/topic_3168
tools:
- name: Clairvoyante
  role: Multi-task five-layer CNN called on ONT alignments via callVarBam.py to produce
    the evaluated variant call set
  repo: https://github.com/aquaskyline/Clairvoyante
- name: Nanopolish
  role: Whole-genome ONT variant caller used as the baseline call set for head-to-head
    benchmarking at 1KGp3 sites
- name: NGMLR
  role: Re-aligns ONT reads to the GRCh37 decoy version 5 reference (v0.2.3) before
    variant calling
- name: Tensorflow
  role: Framework on which the Clairvoyante model checkpoint used for calling is built
    and loaded
- name: Oxford Nanopore
  role: Source of the rel3 HG001 whole-genome reads (MinION, 1D ligation, R9.4) used
    as the evaluation dataset
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-019-09025-z
    title: A multi-task convolutional deep neural network for variant calling in single
      molecule sequencing
  source_content_hash: sha256:92d1f8e4dab16f7668f91120dc3ec92e9856bae859c7b75c6858e91db5f92947
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/aquaskyline/Clairvoyante
  tools:
  - Clairvoyante
  - NGMLR
  - Nanopolish
  - Oxford Nanopore
  - Tensorflow
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_0625
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# variant-calling-metrics-computation

## Summary

Benchmark a germline small-variant caller (Clairvoyante on ONT data) and a baseline caller (Nanopolish) against the GIAB HG001 v3.3.2 truth set by restricting calls to 1KGp3 common variant sites and computing F1-score, FPR, FNR, and precision per caller.

## When to use

Use when benchmarking germline SNV/small-indel callers (e.g., Clairvoyante vs. Nanopolish) on ONT reads against a GIAB truth set, and you need F1, FPR, FNR and precision at a restricted site set such as 1KGp3 common variants.

## When NOT to use

- The goal is somatic or trio-based variant calling — the paper explicitly plans but does not support these.
- Evaluating indel calling on ONT data as the primary metric — Clairvoyante's ONT indel F1-score remains around 50%, so these metrics will not reflect acceptable indel performance.
- Novel sequencing technologies/chemistries lacking a high-quality GIAB-style training dataset — Clairvoyante cannot be applied there, and metrics would be uninterpretable.

## Inputs

- HG001 Oxford Nanopore rel3 whole-genome alignments (BAM, MinION R9.4, 1D ligation)
- Clairvoyante TensorFlow model checkpoint trained on HG001 (chr1 excluded, 1999 epochs)
- GRCh37 decoy version 5 reference genome
- 1KGp3 common variant site list (BED/VCF) for restricting call sets
- GIAB HG001 v3.3.2 truth VCF for chromosome 1

## Outputs

- Clairvoyante chr1 call set restricted to 1KGp3 common sites (VCF)
- Nanopolish chr1 call set restricted to 1KGp3 common sites (VCF)
- Per-caller benchmark metrics (F1, FPR, FNR, precision) written to chr1_benchmark_metrics.csv

## How to apply

Obtain HG001 Oxford Nanopore rel3 whole-genome alignments (MinION, 1D ligation kits, R9.4 chemistry) and realign them to the GRCh37 decoy version 5 reference with NGMLR v0.2.3, since the paper re-aligned PacBio and ONT reads rather than trusting deposited alignments. Load the Clairvoyante TensorFlow checkpoint trained on HG001 with chromosome 1 excluded (1999 epochs) and call variants over chr1 with callVarBam.py, then restrict the resulting call set to 1KGp3 common variant sites; run Nanopolish in whole-genome calling mode on the same alignments and restrict its calls identically. Compare both restricted call sets to the GIAB HG001 v3.3.2 truth set on chromosome 1, counting true positives, false positives, and false negatives to derive F1-score, FPR, FNR, and precision. The decision point is the site restriction: metrics are only comparable to Table 4 when both callers and the truth set are intersected with the same 1KGp3 common sites. Correct application reproduces Clairvoyante F1 90.53% (FPR 2.96%, FNR 14.78%, precision 96.55%) and Nanopolish F1 86.93% (FPR 0.04%, FNR 21.57%, precision 97.51%).

## Related tools

- **Clairvoyante** (Multi-task five-layer CNN called on ONT alignments via callVarBam.py to produce the evaluated variant call set) — https://github.com/aquaskyline/Clairvoyante
- **Nanopolish** (Whole-genome ONT variant caller used as the baseline call set for head-to-head benchmarking at 1KGp3 sites)
- **NGMLR** (Re-aligns ONT reads to the GRCh37 decoy version 5 reference (v0.2.3) before variant calling)
- **Tensorflow** (Framework on which the Clairvoyante model checkpoint used for calling is built and loaded)
- **Oxford Nanopore** (Source of the rel3 HG001 whole-genome reads (MinION, 1D ligation, R9.4) used as the evaluation dataset)

## Examples

```
F1 = 2*TP / (2*TP + FP + FN); precision = TP / (TP + FP); FPR = FP / (FP + TN); write to chr1_benchmark_metrics.csv
```

## Evaluation signals

- Clairvoyante metrics on HG001 chr1 at 1KGp3 common sites match F1 90.53%, FPR 2.96%, FNR 14.78%, precision 96.55% (Table 4).
- Nanopolish metrics on the same sites match F1 86.93%, FPR 0.04%, FNR 21.57%, precision 97.51%.
- Both call sets and the GIAB v3.3.2 truth set are intersected with the identical 1KGp3 common variant site list before metric computation.
- ONTS alignments were re-aligned with NGMLR v0.2.3 to GRCh37 decoy version 5, not evaluated from the deposited alignments.
- chr1_benchmark_metrics.csv exists and contains one row per caller with all four metric columns populated.

## Limitations

- Multi-allelic SNPs are excluded from training and variants with two or more alternative alleles are ignored by the current Clairvoyante design, so these sites may distort metrics if not filtered.
- Clairvoyante does not consider base quality of sequencing reads and only provides exact alternative alleles for indels ≤4 bp.
- Model training can get stuck at a local optimum early and may need a higher learning rate, affecting the checkpoint quality being benchmarked.
- ONT data are available only for HG001, limiting cross-sample evaluation and combined-sample training on ONT data.

## Evidence

- [abstract] Clairvoyante achieves 99.67, 95.78, 90.53% F1-score on 1KP common variants, and 98.65, 92.57, 87.26% F1-score for whole-genome analysis: "Clairvoyante achieves 99.67, 95.78, 90.53% F1-score on 1KP common variants, and 98.65, 92.57, 87.26% F1-score for whole-genome analysis"
- [discussion] Reads were extracted from the downloaded alignments and aligned again to GRCh37 decoy version 5 using NGMLR27 version 0.2.3: "Reads were extracted from the downloaded alignments and aligned again to GRCh37 decoy version 5 using NGMLR27 version 0.2.3"
- [discussion] we used as our truth dataset the latest dataset version 3.3.2 for HG001: "we used as our truth dataset the latest dataset version 3.3.2 for HG001"
- [readme] Clairvoyante uses all available CPU cores by default in `callVar.py`, 4 threads by default in `callVarBam.py`.: "Clairvoyante uses all available CPU cores by default in `callVar.py`, 4 threads by default in `callVarBam.py`."
- [discussion] the Indel F1-score remains around 50%: "it still has a large room to be improved in detecting Indels, especially for ONT data, in which the Indel F1-score remains around 50%"
