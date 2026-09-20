---
name: short-read-variant-calling-with-clair3
description: Use when you have basecalled nanopore reads aligned to hg38 and need
  to detect SNVs/indels, especially when comparing variant-detection performance across
  read simulators on chr22 (NA12878) data.
when_to_use_negative:
- Input is raw nanopore signal that has not yet been basecalled and aligned with minimap2
  — Clair3 requires read sequences, not signal
- The goal is signal-quality simulation assessment itself (e.g. DTW distance to real
  squiggles) rather than downstream variant detection performance
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_3673
- http://edamontology.org/topic_0622
tools:
- name: seq2squiggle
  role: Simulates R9.4.1 and R10.4.1 nanopore signals from the variant-carrying chr22
    reference whose basecalled reads are then variant-called
  repo: github.com/ZKI-PH-ImageAnalysis/seq2squiggle
- name: minimap2
  role: Aligns basecalled reads to hg38 to produce the BAM input for Clair3
- name: Clair3
  role: Calls SNVs and indels from each minimap2 alignment
- name: RTGtools
  role: Scores each Clair3 callset against NA12878 truth variants to compute true
    positives, false positives, recall, and precision
- name: DeepSimulator
  role: Baseline simulator (run in two configurations) for comparison against seq2squiggle
- name: squigulator
  role: Baseline simulator for comparison against seq2squiggle
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioinformatics/btae744
    title: End-to-end simulation of nanopore sequencing signals with feed-forward
      transformers
  source_content_hash: sha256:e0b4eb5972401e1af46b454a326bab9aa167c6693e0df0dec987e968209ffb2c
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
  - http://edamontology.org/topic_3673
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# Short-read variant calling with Clair3

## Summary

Call SNVs and indels from minimap2-aligned basecalled nanopore reads using Clair3, then benchmark the callset against Genome in a Bottle NA12878 truth variants with RTGtools to score simulator-supported variant detection.

## When to use

Use when you have basecalled nanopore reads aligned to hg38 and need to detect SNVs/indels, especially when comparing variant-detection performance across read simulators on chr22 (NA12878) data.

## When NOT to use

- Input is raw nanopore signal that has not yet been basecalled and aligned with minimap2 — Clair3 requires read sequences, not signal
- The goal is signal-quality simulation assessment itself (e.g. DTW distance to real squiggles) rather than downstream variant detection performance

## Inputs

- Basecalled read sequences from simulated nanopore signals (BLOW5/POD5 from seq2squiggle, squigulator, or DeepSimulator)
- hg38 reference genome for read alignment
- Modified chr22 reference carrying Genome in a Bottle NA12878 variants
- NA12878 truth variant set (Genome in a Bottle)
- Pore chemistry specification (R9.4.1 or R10.4.1)

## Outputs

- Per-simulator variant callsets from Clair3 (VCF)
- variant_calling_benchmark.tsv with true positives, false positives, recall, and precision per simulator

## How to apply

Construct a modified chr22 reference by applying Genome in a Bottle NA12878 variants to hg38, then simulate reads (e.g. 250,000 R10.4.1 reads or 50,000 R9.4.1 reads with seq2squiggle). Basecall all simulated signal datasets (BLOW5/POD5 from seq2squiggle, squigulator, or DeepSimulator) into read sequences and align them to the hg38 reference with minimap2. Run Clair3 on each alignment to produce a variant callset, then score each callset against the NA12878 truth variants with RTGtools, compiling true positives, false positives, recall, and precision per simulator. In this benchmark, reads simulated with seq2squiggle yielded higher numbers of true-positive SNVs and overall higher recall and precision than squigulator and DeepSimulator, so per-simulator comparison is the key decision output.

## Related tools

- **seq2squiggle** (Simulates R9.4.1 and R10.4.1 nanopore signals from the variant-carrying chr22 reference whose basecalled reads are then variant-called) — github.com/ZKI-PH-ImageAnalysis/seq2squiggle
- **minimap2** (Aligns basecalled reads to hg38 to produce the BAM input for Clair3)
- **Clair3** (Calls SNVs and indels from each minimap2 alignment)
- **RTGtools** (Scores each Clair3 callset against NA12878 truth variants to compute true positives, false positives, recall, and precision)
- **DeepSimulator** (Baseline simulator (run in two configurations) for comparison against seq2squiggle)
- **squigulator** (Baseline simulator for comparison against seq2squiggle)

## Examples

```
minimap2 -ax map-ont hg38.fa basecalled.fastq | samtools sort -o aln.bam; clair3 --bam_fn aln.bam --ref_fn hg38.fa --output vcf_out; rtg vcfeval -b truth.vcf -c vcf_out/merge_output.vcf.gz
```

## Evaluation signals

- variant_calling_benchmark.tsv contains per-simulator TP, FP, recall, and precision rows for every simulator configuration
- seq2squiggle-derived callsets show higher true-positive SNV counts and higher recall/precision than squigulator and DeepSimulator (as in Fig. 4)
- RTGtools completes successfully against the NA12878 truth set without reference-mismatch errors (modified chr22 vs hg38 used consistently)
- Clair3 produces a valid VCF for each simulator's alignment, and alignment BAMs map to hg38

## Limitations

- Benchmark results depend on the simulation model quality — squigulator and DeepSimulator rely on static k-mer models, which limits their realism compared to seq2squiggle's learned sequential context
- Read counts are chemistry-specific (250,000 R10.4.1 vs 50,000 R9.4.1), so cross-chemistry comparisons are not at equal depth
- Only chr22 and the NA12878 truth set are covered; genome-wide variant calling performance is not established

## Evidence

- [results] Reads generated with seq2squiggle showed a higher number of true-positive SNVs, along with an overall higher recall and precision compared to squigulator and DeepSimulator: "Reads generated with seq2squiggle showed a higher number of true-positive SNVs, along with an overall higher recall and precision compared to squigulator and DeepSimulator (Fig. 4)."
- [intro] seq2squiggle is a deep learning-based tool for generating artifical nanopore signals from DNA sequence data.: "`seq2squiggle` is a deep learning-based tool for generating artifical nanopore signals from DNA sequence data."
- [methods] Simulate 250,000 R10.4.1 reads and 50,000 R9.4.1 reads from the modified chr22: [evidence span withheld — anchor status: not_found]
- [methods] Align the basecalled reads to hg38 with minimap2; call variants with Clair3: [evidence span withheld — anchor status: not_found]
- [readme] Generate 10,000 reads & signals using R9.4.1 chemistry: "# Generate 10,000 reads & signals using R9.4.1 chemistry
seq2squiggle predict example.fasta -o example.blow5 -n 10000 --profile dna_r9_min"
