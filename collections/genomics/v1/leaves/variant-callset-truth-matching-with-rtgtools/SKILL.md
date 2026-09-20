---
name: variant-callset-truth-matching-with-rtgtools
description: Use when you need to quantify SNV/indel variant-calling accuracy — precision
  and recall — of one or more callsets against a benchmark truth set such as GIAB
  NA12878.
when_to_use_negative:
- Input is a raw sequencing signal file (BLOW5/POD5) — basecalling, alignment, and
  variant calling must precede truth matching
- No curated truth variant set exists for the sample/genome — precision and recall
  cannot be computed
- Goal is signal-similarity assessment rather than downstream variant-detection performance
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: RTGtools
  role: Scores each Clair3 variant callset against the NA12878 truth variants, producing
    TP/FP/recall/precision
- name: Clair3
  role: Calls SNV and indel variants from minimap2 alignments of basecalled simulated
    reads
- name: minimap2
  role: Aligns basecalled reads to hg38 prior to variant calling
- name: seq2squiggle
  role: Simulates R10.4.1 (250,000 reads) and R9.4.1 (50,000 reads) signals from the
    variant-carrying chr22 for one arm of the benchmark
  repo: github.com/ZKI-PH-ImageAnalysis/seq2squiggle
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioinformatics/btae744
    title: End-to-end simulation of nanopore sequencing signals with feed-forward
      transformers
  source_content_hash: sha256:3888fd585d3f1e5ac85374f4464e91360fc5bf924b5dd67cc5347a39131d408b
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Clair3
  - DeepSimulator
  - RTGtools
  - mamba
  - minimap2
  - seq2squiggle
  - squigulator
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# Variant callset truth-matching with RTGtools

## Summary

Scores a called VCF (e.g. Clair3 output from simulated nanopore reads) against a curated truth VCF (Genome in a Bottle NA12878) with RTGtools, compiling true positives, false positives, recall, and precision per simulator or pipeline variant.

## When to use

Use when you need to quantify SNV/indel variant-calling accuracy — precision and recall — of one or more callsets against a benchmark truth set such as GIAB NA12878.

## When NOT to use

- Input is a raw sequencing signal file (BLOW5/POD5) — basecalling, alignment, and variant calling must precede truth matching
- No curated truth variant set exists for the sample/genome — precision and recall cannot be computed
- Goal is signal-similarity assessment rather than downstream variant-detection performance

## Inputs

- Clair3 variant callset (VCF) from aligned basecalled reads
- Genome in a Bottle NA12878 truth variant VCF (SNVs and indels)
- Reference genome (hg38 chr22 with GIAB variants applied)
- Alignments of basecalled reads (minap2 BAM) from R9.4.1 and R10.4.1 simulated signal data

## Outputs

- Per-callset TP, FP, recall, and precision metrics for SNVs and indels
- variant_calling_benchmark.tsv — compiled per-simulator benchmark table

## How to apply

Build a modified reference by applying the Genome in a Bottle NA12878 variant set to hg38 chr22, simulate or process reads, basecall the signal to FASTQ, and align with minimap2 to hg38. Call variants from each alignment with Clair3, then score each callset against the NA12878 truth variants with RTGtools, tallying true positives and false positives separately for SNVs and indels. Compile recall and precision per condition (here, per simulator: seq2squiggle, squigulator, DeepSimulator) into a single table such as variant_calling_benchmark.tsv so simulators can be compared on the same truth set. Judge success by higher TP counts and jointly higher recall and precision relative to baselines — in the source benchmark, seq2squiggle-derived reads yielded more true-positive SNVs and higher recall/precision than squigulator and DeepSimulator (Fig. 4).

## Related tools

- **RTGtools** (Scores each Clair3 variant callset against the NA12878 truth variants, producing TP/FP/recall/precision)
- **Clair3** (Calls SNV and indel variants from minimap2 alignments of basecalled simulated reads)
- **minimap2** (Aligns basecalled reads to hg38 prior to variant calling)
- **seq2squiggle** (Simulates R10.4.1 (250,000 reads) and R9.4.1 (50,000 reads) signals from the variant-carrying chr22 for one arm of the benchmark) — github.com/ZKI-PH-ImageAnalysis/seq2squiggle

## Examples

```
rtg eval --baseline na12878_chr22_truth.sdf --call clair3_calls.vcf -o rtg_eval_output.txt
```

## Evaluation signals

- variant_calling_benchmark.tsv contains one row per simulator with non-negative TP, FP and recall/precision in [0, 1]
- TP counts for SNVs and indels are reported separately and sum consistently with the truth set size on chr22
- seq2squiggle arm shows higher true-positive SNV counts and recall/precision than squigulator and DeepSimulator arms (expected direction from Fig. 4)
- All arms used the same modified chr22 reference, read counts (250,000 R10.4.1; 50,000 R9.4.1), basecaller, aligner (minimap2), and caller (Clair3), so differences are attributable to the simulator

## Limitations

- Benchmark is restricted to chr22 with GIAB NA12878 variants; genome-wide or non-NA12878 performance is not directly established
- Results depend on the specific read counts (250,000 R10.4.1, 50,000 R9.4.1) and the two DeepSimulator configurations chosen; other settings may shift rankings
- Simulator quality affects basecall fidelity, so metric differences reflect the full simulate→basecall→call chain, not RTGtools itself

## Evidence

- [methods] RTGtools benchmark workflow: [evidence span withheld — anchor status: not_found]
- [results] Comparative result: "Reads generated with seq2squiggle showed a higher number of true-positive SNVs, along with an overall higher recall and precision compared to squigulator and DeepSimulator (Fig. 4)."
- [full_text] Benchmark research question: [evidence span withheld — anchor status: not_found]
