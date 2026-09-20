---
name: vcf-truth-set-comparison
description: Use when a variant-calling VCF (e.g., Clairvoyante or Nanopolish output
  for HG001 Illumina/PacBio/ONT data) must be scored against the GIAB v3.3.2 HG001
  truth set to produce F1, FPR, FNR, and precision metrics.
when_to_use_negative:
- Evaluating somatic or trio-based variant calling — Clairvoyante and this GIAB-HG001
  benchmark target germline SNV/small-indel calling only
- Evaluating a technology/chemistry with no high-quality GIAB-style truth dataset
  available for training and benchmarking
- Assessing long-indel accuracy — the caller only provides exact alternative alleles
  for indels ≤4 bp, so indel-dominated comparisons (especially ONT indels, F1 ~50%)
  require extra caution
edam_operation: http://edamontology.org/operation_3197
edam_topics:
- http://edamontology.org/topic_0625
tools:
- name: RTG vcfeval
  role: Haplotype-aware genotype matching of caller VCF against GIAB v3.3.2 HG001
    truth set to classify TP/FP/FN
- name: Clairvoyante
  role: TensorFlow five-layer CNN caller producing the query VCF for Illumina, PacBio,
    and ONT HG001 benchmarks
  repo: https://github.com/aquaskyline/Clairvoyante
- name: Nanopolish
  role: Baseline ONT whole-genome variant caller whose chr1 calls are compared under
    the same truth-set evaluation
- name: NGMLR
  role: Realignment of PacBio and ONT reads to GRCh37 decoy version 5 (v0.2.3) prior
    to calling and evaluation
provenance:
  source_task_ids:
  - task_004
  - task_003
  - task_001
  source_papers:
  - doi: 10.1038/s41467-019-09025-z
    title: A multi-task convolutional deep neural network for variant calling in single
      molecule sequencing
  source_content_hash: sha256:693a0969b07cb9a0a334ecc2ea34ba81caf35904f8a39c98083484912619dd72
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/aquaskyline/Clairvoyante
  tools:
  - Clairvoyante
  - Illumina
  - NGMLR
  - Nanopolish
  - Oxford Nanopore
  - PacBio
  - RTG vcfeval
  edam_operation: http://edamontology.org/operation_3197
  edam_topics:
  - http://edamontology.org/topic_0625
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 11
---

# vcf-truth-set-comparison

## Summary

Benchmark a caller's VCF against the Genome-in-a-Bottle v3.3.2 HG001 truth dataset using RTG vcfeval haplotype-aware genotype matching, then derive FPR, FNR, precision, and F1 per sequencing technology. This is the evaluation scheme used to validate Clairvoyante's reported F1-scores of 99.67/95.78/90.53% at 1KP common variants and 98.65/92.57/87.26% genome-wide.

## When to use

Use when a variant-calling VCF (e.g., Clairvoyante or Nanopolish output for HG001 Illumina/PacBio/ONT data) must be scored against the GIAB v3.3.2 HG001 truth set to produce F1, FPR, FNR, and precision metrics.

## When NOT to use

- Evaluating somatic or trio-based variant calling — Clairvoyante and this GIAB-HG001 benchmark target germline SNV/small-indel calling only
- Evaluating a technology/chemistry with no high-quality GIAB-style truth dataset available for training and benchmarking
- Assessing long-indel accuracy — the caller only provides exact alternative alleles for indels ≤4 bp, so indel-dominated comparisons (especially ONT indels, F1 ~50%) require extra caution

## Inputs

- Caller output VCF (e.g., Clairvoyante genome-wide or chr1 callset, genotyped)
- Genome-in-a-Bottle (GIAB) version 3.3.2 truth VCF for HG001
- Reference genome FASTA for vcfeval (GRCh37 decoy version 5)
- Optional site filter BED/VCF (1KG project phase 3 common variant site list)

## Outputs

- RTG vcfeval summary with TP/FP/FN counts per technology or configuration
- Metrics table (e.g., vcfeval_f1_scores.csv or chr1_benchmark_metrics.csv) with FPR, FNR, precision, and F1-score

## How to apply

Run RTG vcfeval with the GIAB HG001 v3.3.2 truth VCF as baseline and the caller's VCF as the query, using haplotype-aware genotype matching so true positives require matching genotype, not just overlapping position. Extract TP, FP, and FN counts from the vcfeval summary output and compute Precision = TP/(TP+FP), Recall = TP/(TP+FN), FPR = FP/(FP+TP), FNR = FN/(FN+TP), and F1 = 2·precision·recall/(precision+recall). If the evaluation is restricted to a site subset (e.g., 1KG phase 3 common variant sites) or a single chromosome (e.g., HG001 chr1), intersect the call set with those sites before running vcfeval rather than filtering the truth set. When benchmarking per technology, keep each technology's alignment preprocessing intact (Illumina downsampled to 50×; PacBio and ONT realigned to GRCh37 decoy version 5 with NGMLR v0.2.3) since realignment strongly affects indel-sensitive recall. For whole-genome claims with a 0.2 alternative allele frequency cutoff, verify computed F1 values reproduce the reported 98.65% (Illumina), 92.57% (PacBio), and 87.26% (Oxford Nanopore) figures.

## Related tools

- **RTG vcfeval** (Haplotype-aware genotype matching of caller VCF against GIAB v3.3.2 HG001 truth set to classify TP/FP/FN)
- **Clairvoyante** (TensorFlow five-layer CNN caller producing the query VCF for Illumina, PacBio, and ONT HG001 benchmarks) — https://github.com/aquaskyline/Clairvoyante
- **Nanopolish** (Baseline ONT whole-genome variant caller whose chr1 calls are compared under the same truth-set evaluation)
- **NGMLR** (Realignment of PacBio and ONT reads to GRCh37 decoy version 5 (v0.2.3) prior to calling and evaluation)

## Examples

```
rtg vcfeval -b giab_hg001_v3.3.2.vcf.gz -c clairvoyante_illumina_calls.vcf.gz -t grch37_decoy_v5 -o vcfeval_out && grep -E 'True-pos|False-pos|False-neg' vcfeval_out/summary.txt
```

## Evaluation signals

- Computed whole-genome F1-scores at the 0.2 allele-frequency cutoff match 98.65% (Illumina), 92.57% (PacBio), and 87.26% (Oxford Nanopore rel5)
- Best Illumina configuration (HG001-trained model, 999 epochs) reproduces FPR 0.25%, FNR 0.41%, precision 99.75%, F1 99.67% at 1KGp3 common sites
- Best ONT chr1 configuration (1999-epoch model) reproduces F1 90.53%, FPR 2.96%, FNR 14.78%, precision 96.55%; Nanopolish baseline gives F1 86.93%, FPR 0.04%, FNR 21.57%, precision 97.51%
- vcfeval output partitions every call into exactly TP, FP, or FN; TP+FN equals the truth-set variant count within the evaluated regions
- Metrics file (vcfeval_f1_scores.csv) contains one precision/recall/F1 row per technology with values in the 0–100% range

## Limitations

- The truth set (GIAB v3.3.2 HG001) anchors evaluation to standard samples; Clairvoyante cannot be applied to novel technologies or chemistries lacking such a dataset
- Truth-set comparison is genotyper-level: ONT indel F1-scores remain around 50% and need substantial improvement
- Multi-allelic variants (two or more alternative alleles) are ignored by the caller's design, so they will systematically appear as discordances in the comparison
- No mapping-quality filtering is recommended before calling, so alignment quality is not a confound you can tune during evaluation

## Evidence

- [discussion] GIAB truth-set choice: "we used as our truth dataset the latest dataset version 3.3.2 for HG001"
- [readme] Reported whole-genome F1 targets: "achieved 98.65%, 92.57%, 87.26% F1 score on the whole genome, in Illumina, PacBio, and Oxford Nanopore data, respectively."
- [results] Genotype-aware classification workflow: [evidence span withheld — anchor status: not_found]
- [abstract] Per-technology F1 finding: "Clairvoyante achieves 99.67, 95.78, 90.53% F1-score on 1KP common variants, and 98.65, 92.57, 87.26% F1-score for whole-genome analysis"
