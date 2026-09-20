---
name: variant-call-evaluation-against-giab-truth-set
description: Use when you have called variants (VCF) from long-read sequencing of
  a benchmark sample such as NA12878 and need quantitative accuracy (precision/recall/F1)
  against GIAB high-confidence truth sets.
when_to_use_negative:
- The sample has no GIAB high-confidence truth set for the reference genome used (e.g.
  non-benchmark organisms or non-standard references)
- You want to evaluate structural variant or indel calling accuracy — Longshot calls
  only SNVs and GIAB comparison here targets SNV accuracy
- The caller's output VCF has not been genotype-quality filtered, since unfiltered
  calls confound precision/F1 comparisons
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_3673
- http://edamontology.org/topic_0622
tools:
- name: Longshot
  role: SNV caller whose filtered output VCF is benchmarked against GIAB high-confidence
    calls
  repo: https://github.com/pjedge/longshot
- name: Genome in a Bottle (GIAB)
  role: Provides the high-confidence truth call set used to assess SNV-calling accuracy
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-019-12493-y
    title: Longshot enables accurate variant calling in diploid genomes from single-molecule
      long read sequencing
  source_content_hash: sha256:e0846b7daf0c8f4109c89c733055ba2579306f8691ee2a01b170a6ed32150aba
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/pjedge/longshot
  tools:
  - Genome in a Bottle (GIAB)
  - Longshot
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_3673
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 12
---

# variant-call-evaluation-against-giab-truth-set

## Summary

Evaluates SNV calls from a long-read variant caller (e.g. Longshot on ONT or PacBio data) against the Genome in a Bottle (GIAB) high-confidence call set, computing precision, recall, and F1 from true/false positive and negative counts.

## When to use

Use when you have called variants (VCF) from long-read sequencing of a benchmark sample such as NA12878 and need quantitative accuracy (precision/recall/F1) against GIAB high-confidence truth sets.

## When NOT to use

- The sample has no GIAB high-confidence truth set for the reference genome used (e.g. non-benchmark organisms or non-standard references)
- You want to evaluate structural variant or indel calling accuracy — Longshot calls only SNVs and GIAB comparison here targets SNV accuracy
- The caller's output VCF has not been genotype-quality filtered, since unfiltered calls confound precision/F1 comparisons

## Inputs

- Called variant VCF from a long-read SNV caller (e.g. Longshot output)
- GIAB high-confidence call set for the matching reference genome (GRCh38/hs37d5)
- Median read coverage of the dataset (to select the GQ cutoff)
- Optional: known common indels VCF for false-positive reduction

## Outputs

- Contingency counts of true positives, false positives, false negatives
- Precision, recall, and F1 score (e.g. written to na12878_ont_accuracy.tsv)

## How to apply

Align the long reads (e.g. NA12878 ONT reads with minimap2 to GRCh38, or PacBio 45x reads to hs37d5 with BLASR), run the caller (e.g. `longshot --bam ont.bam --ref grch38.fa --out calls.vcf`), and filter the resulting VCF at a genotype-quality cutoff. For Longshot, the paper used a variable GQ cutoff matched to the median read coverage (e.g. GQ 65 for the NA12878 37x ONT run) and GQ 50 fixed for short-read comparisons. Compare the filtered calls to the GIAB high-confidence call set for the same reference, counting true positives, false positives, and false negatives, then compute precision, recall, and F1. Interpret false positives cautiously because the GIAB truth sets were generated from short-read data and may miss long-read-only true variants.

## Related tools

- **Longshot** (SNV caller whose filtered output VCF is benchmarked against GIAB high-confidence calls) — https://github.com/pjedge/longshot
- **Genome in a Bottle (GIAB)** (Provides the high-confidence truth call set used to assess SNV-calling accuracy)

## Examples

```
longshot --bam ont.bam --ref grch38.fa --out na12878_ont_longshot.vcf  # then compare filtered VCF (GQ 65) to GIAB high-confidence calls for GRCh38
```

## Evaluation signals

- On NA12878 37x ONT data at GQ 65, expect precision ~0.991 and recall ~0.933, giving a best F1 of 0.967 against GIAB high-confidence calls on GRCh38
- Contingency counts (TP, FP, FN) must be computed from the same reference genome used for both the caller and the truth set
- Filtering known common indels should reduce the false discovery rate to ~0.3%
- The GQ cutoff should be matched to the median read coverage for Longshot (GQ 50 fixed for short-read comparators)
- Example-data sanity check: Longshot output on example_data/pacbio_reads_30x.bam should match ground_truth_variants.vcf

## Limitations

- GIAB and Platinum Genomes truth sets were generated from short-read data and are biased in favor of short-read technologies, so some Longshot false positives may actually be false negatives in the GIAB call sets
- Allele quality filtering (default phred threshold 7.0) reduces the effective read depth for SMS reads
- Accurate short indel calling with SMS reads remains challenging due to the high insertion/deletion error rate
- Correctly mapping PacBio reads in highly similar segmental duplications remains a challenge

## Evidence

- [discussion] High-confidence variant call sets generated by the GIAB project were used for assessing accuracy of SNV calling: "High-confidence variant call sets generated by the GIAB project were used for assessing accuracy of SNV calling"
- [discussion] Variable GQ cutoff matched to median coverage: "For Longshot, we used a variable GQ cutoff (matched to the median read coverage) for filtering variants."
- [discussion] Truth sets biased toward short reads: "The GIAB and Platinum Genomes variant sets used to assess variant-calling accuracy in this study were generated using short-read datasets, are therefore biased in favor of short-read technologies16"
- [discussion] FDR reduced by filtering known indels: [evidence span withheld — anchor status: not_found]
- [readme] Example data validation against ground truth: "The output can be compared to ground_truth_variants.vcf for accuracy."
