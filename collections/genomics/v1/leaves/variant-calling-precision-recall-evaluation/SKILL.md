---
name: variant-calling-precision-recall-evaluation
description: Use when benchmarking a long-read SNV caller's accuracy on GIAB PacBio
  genomes (NA12878, NA24385, NA24149, NA24143) or assessing how precision/recall change
  with sub-sampled coverage (e.g. 28x to 64x).
when_to_use_negative:
- The question is structural variant or indel calling accuracy — Longshot only calls
  SNVs and accurate short indel calling with SMS reads is noted as challenging due
  to high insertion/deletion error rates.
- The truth set is not a short-read-derived high-confidence call set (e.g. de novo
  assemblies without benchmark regions) — GIAB sets are biased in favor of short-read
  technologies, so some caller false positives may be truth-set false negatives.
- Input is raw reads without an indexed, sorted BAM/CRAM aligned to the reference
  used by the truth set — variant-level comparison requires coordinate-matched calls.
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3168
tools:
- name: Longshot
  role: SNV caller under evaluation; run with --auto_max_cov (-A) and region-based
    per-chromosome processing against the indexed reference
  repo: https://github.com/pjedge/longshot
- name: HapCUT2
  role: Haplotype-informed phasing engine invoked iteratively inside Longshot during
    genotyping; cited as the phasing method underpinning accuracy
- name: Pacific Biosciences
  role: SMS read source for the GIAB whole-genome datasets being benchmarked
provenance:
  source_task_ids:
  - task_002
  - task_001
  source_papers:
  - doi: 10.1038/s41467-019-12493-y
    title: Longshot enables accurate variant calling in diploid genomes from single-molecule
      long read sequencing
  source_content_hash: sha256:ce0df0342c822db74189bf79a47af991bb65eb39b4bd6723053fd85054dbe9d7
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/pjedge/longshot
  tools:
  - HapCUT2
  - Longshot
  - Pacific Biosciences
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# variant-calling-precision-recall-evaluation

## Summary

Evaluates SNV caller accuracy on whole-genome PacBio SMS data by comparing caller VCFs against GIAB high-confidence truth sets and computing precision (TP/(TP+FP)) and recall (TP/(TP+FN)), optionally as a function of coverage. This is the benchmarking approach used to show Longshot's accuracy (e.g. P=0.995, R=0.968 on NA12878 44x) versus WhatsHap and Clairvoyante.

## When to use

Use when benchmarking a long-read SNV caller's accuracy on GIAB PacBio genomes (NA12878, NA24385, NA24149, NA24143) or assessing how precision/recall change with sub-sampled coverage (e.g. 28x to 64x).

## When NOT to use

- The question is structural variant or indel calling accuracy — Longshot only calls SNVs and accurate short indel calling with SMS reads is noted as challenging due to high insertion/deletion error rates.
- The truth set is not a short-read-derived high-confidence call set (e.g. de novo assemblies without benchmark regions) — GIAB sets are biased in favor of short-read technologies, so some caller false positives may be truth-set false negatives.
- Input is raw reads without an indexed, sorted BAM/CRAM aligned to the reference used by the truth set — variant-level comparison requires coordinate-matched calls.

## Inputs

- Sorted, indexed PacBio SMS BAM alignments of GIAB genomes (NA12878, NA24385, NA24149, NA24143) with varying coverage
- Indexed reference FASTA (hs37d5 or hg38)
- GIAB high-confidence variant call sets (truth VCFs) and high-confidence regions (BED)
- Caller output VCFs (e.g. Longshot, WhatsHap, Clairvoyante) restricted to SNVs

## Outputs

- Per-sample SNV precision, recall, and FDR values (e.g. table1_benchmark_results.csv, longshot_giab_accuracy_metrics.csv)
- Precision/recall as a function of coverage for sub-sampled genomes
- Wall-clock runtime per caller per sample (optional, for caller comparisons)

## How to apply

Obtain GIAB PacBio SMS BAM alignments (e.g. NA12878 44x NGMLR-aligned, or 45x BLASR-aligned to hs37d5) and the corresponding GIAB high-confidence variant call sets. Run the caller under test with its recommended filters — for Longshot, default min_mapq 20, min_cov 6, plus the max-coverage cutoff d+5*sqrt(d) (via --auto_max_cov, where d is the median read depth), the two-tailed Fisher's exact strand-bias filter (p < 0.01), and the density filter removing >10 SNVs in a 500 bp window. Restrict comparisons to SNVs within GIAB high-confidence regions and apply a GQ cutoff matched to the median read coverage (GQ 50 is used for short-read variants). Count true positives, false positives, and false negatives against the truth set, then compute precision = TP/(TP+FP) and recall = TP/(TP+FN). For coverage-dependence, sub-sample BAMs to target depths (e.g. NA24385 at 28x/37x/46x/64x) and repeat; expected validation points include NA12878 30x P=0.9942/R=0.9592 and NA24385 recall rising from 0.9608 at 28x to 0.9798 at 64x. Tabulate per-sample precision/recall (and runtime for caller-vs-caller comparisons) into a CSV.

## Related tools

- **Longshot** (SNV caller under evaluation; run with --auto_max_cov (-A) and region-based per-chromosome processing against the indexed reference) — https://github.com/pjedge/longshot
- **HapCUT2** (Haplotype-informed phasing engine invoked iteratively inside Longshot during genotyping; cited as the phasing method underpinning accuracy)
- **Pacific Biosciences** (SMS read source for the GIAB whole-genome datasets being benchmarked)

## Examples

```
longshot --bam pacbio.bam --ref ref.fa --out output.vcf
```

## Evaluation signals

- Reproduced NA12878 30x metrics: precision ≈ 0.9942, recall ≈ 0.9592; 45x recall ≈ 0.9734.
- NA24385 sub-sampled recall increases monotonically from 0.9608 at 28x to 0.9798 at 64x while precision changes only moderately.
- Counted TP/FP/FN are restricted to SNVs inside GIAB high-confidence regions with the coverage-matched GQ cutoff applied — counts without this restriction are invalid.
- Longshot FDR on NA12878 44x ≈ 0.5% (vs 1.6% for Clairvoyante); precision > 0.99 expected on high-confidence regions.
- FDR can drop to ~0.3% if known common indels are additionally filtered out — a sanity check on FP composition.

## Limitations

- GIAB and Platinum Genomes truth sets were generated using short-read datasets and are therefore biased in favor of short-read technologies; some caller false positives may be truth-set false negatives.
- Correctly mapping PacBio reads in highly similar segmental duplications remains a challenge, affecting accuracy in those regions.
- Allele observations with phred-scaled quality below 7.0 are discarded, reducing the effective read depth for SMS reads.
- The max-coverage filter (d+5*sqrt(d)) removes copy-number-amplification false SNVs but the -A auto coverage mode is slower than a fixed max_cov.

## Evidence

- [discussion] High-confidence variant call sets generated by the GIAB project were used for assessing accuracy of SNV calling: "High-confidence variant call sets generated by the GIAB project were used for assessing accuracy of SNV calling"
- [discussion] depth filter: [evidence span withheld — anchor status: not_found]
- [discussion] GQ cutoff: "For Longshot, we used a variable GQ cutoff (matched to the median read coverage) for filtering variants."
- [discussion] strand bias filter: [evidence span withheld — anchor status: not_found]
- [readme] example invocation: "longshot --bam example_data/pacbio_reads_30x.bam --ref example_data/genome.fa --out example_data/longshot_output.vcf"
