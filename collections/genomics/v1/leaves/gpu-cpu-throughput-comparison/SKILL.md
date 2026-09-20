---
name: gpu-cpu-throughput-comparison
description: Use when comparing the real-time decision speed of a raw-signal classifier
  versus basecalling-plus-mapping pipelines, or when benchmarking the same model on
  GPU versus CPU to justify CPU-only deployment.
when_to_use_negative:
- The goal is to measure enrichment/depletion yield (read-count changes) during live
  sequencing runs rather than decision-time throughput
- Inputs are already basecalled FASTQ/BAM sequences rather than raw nanopore signals,
  so the direct-signal classifier cannot be applied
- The comparison targets sequencing kit RNA004 or non-R9.4.1 chemistry where the Dorado
  RNA R9.4.1 fast baseline configuration no longer matches the data
edam_operation: http://edamontology.org/operation_3898
edam_topics:
- http://edamontology.org/topic_3168
- http://edamontology.org/topic_0622
tools:
- name: RISER
  role: Direct-signal deep learning classifier whose per-read mRNA/non-mRNA classification
    time is benchmarked on GPU and CPU
  repo: https://github.com/comprna/riser
- name: Dorado
  role: Basecalling server (RNA R9.4.1 fast configuration) forming the basecalling
    half of the adaptive sampling baseline
- name: minimap2
  role: Maps Dorado-basecalled reads against protein_coding GENCODE transcripts (-ax
    map-ont) to complete the adaptive sampling baseline
- name: BoostNano
  role: Pre-trims sequencing adapter and poly(A) tail from raw signals so both methods
    see identically processed inputs
  repo: https://github.com/haotianteng/BoostNano
- name: PyTorch
  role: Framework in which the RISER CNN is run on the single NVIDIA Tesla V100 GPU
    for the timing benchmark
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-024-48673-8
    title: Biochemical-free enrichment or depletion of RNA classes in real-time during
      direct RNA sequencing with RISER
  source_content_hash: sha256:365d10f946268f36af218d514a1b7404ad57676138b37752d76e6ab4802a26b5
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/comprna/riser
  tools:
  - BoostNano
  - Dorado
  - PyTorch
  - RISER
  - minimap2
  edam_operation: http://edamontology.org/operation_3898
  edam_topics:
  - http://edamontology.org/topic_3168
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# gpu-cpu-throughput-comparison

## Summary

Benchmark the per-read classification speed of a direct-signal deep learning classifier (RISER) against sequence-based adaptive sampling (Dorado basecalling plus minimap2 mapping) on matched GPU and CPU hardware, quantifying speed ratios on fixed-length inputs.

## When to use

Use when comparing the real-time decision speed of a raw-signal classifier versus basecalling-plus-mapping pipelines, or when benchmarking the same model on GPU versus CPU to justify CPU-only deployment.

## When NOT to use

- The goal is to measure enrichment/depletion yield (read-count changes) during live sequencing runs rather than decision-time throughput
- Inputs are already basecalled FASTQ/BAM sequences rather than raw nanopore signals, so the direct-signal classifier cannot be applied
- The comparison targets sequencing kit RNA004 or non-R9.4.1 chemistry where the Dorado RNA R9.4.1 fast baseline configuration no longer matches the data

## Inputs

- 1000 randomly sampled raw DRS signals (fast5) with signal length >= 6.2 s, trimmed to 6.2 s
- BoostNano-processed fast5 files with adapter and poly(A) tail trimmed from the raw signal
- GENCODE v34 (GRCh38.p13) transcriptome reference filtered to protein_coding transcripts
- Trained RISER mRNA classification model (PyTorch, v1.9.0)

## Outputs

- riser_vs_as_timing.csv with per-read and summary classification times for RISER (GPU and CPU) and adaptive sampling (GPU)
- speed_comparison.png plotting RISER-vs-AS speed ratios and cumulative fraction of signals classified versus time
- Summary speed ratios (e.g., 44x RISER-GPU vs AS-GPU; 371x RISER-CPU vs AS-GPU; 16x RISER-CPU vs AS-GPU, with AS averaging 5.8 s per signal)

## How to apply

Randomly sample reads (e.g., 1000 DRS signals with signals at least 6.2 s long) and trim every signal to the same fixed length (6.2 s) so both methods operate on equal-length inputs. Pre-trim the sequencing adapter and poly(A) tail from all raw nanopore signals with BoostNano to remove differences in internal trimming between methods. Run RISER (PyTorch CNN, probability threshold 0.9) on a single NVIDIA Tesla V100 GPU and on a single CPU, recording per-read time to classify each read as mRNA or non-mRNA; run the sequence-based adaptive sampling baseline on a single GPU by basecalling with the Dorado server (RNA R9.4.1 fast configuration) and mapping against GENCODE v34 (GRCh38.p13) protein_coding transcripts with minimap2 -ax map-ont, stopping when a signal maps to an mRNA. Compute per-read and summary classification times, speed ratios (RISER/AS), and the cumulative fraction of signals classified versus time; write a timing CSV and a speed comparison plot. The rationale is that direct signal classification avoids the computationally heavier basecalling step, so the benchmark validates the claim that RISER is faster than GPU-accelerated basecalling and mapping on both GPU and CPU.

## Related tools

- **RISER** (Direct-signal deep learning classifier whose per-read mRNA/non-mRNA classification time is benchmarked on GPU and CPU) — https://github.com/comprna/riser
- **Dorado** (Basecalling server (RNA R9.4.1 fast configuration) forming the basecalling half of the adaptive sampling baseline)
- **minimap2** (Maps Dorado-basecalled reads against protein_coding GENCODE transcripts (-ax map-ont) to complete the adaptive sampling baseline)
- **BoostNano** (Pre-trims sequencing adapter and poly(A) tail from raw signals so both methods see identically processed inputs) — https://github.com/haotianteng/BoostNano
- **PyTorch** (Framework in which the RISER CNN is run on the single NVIDIA Tesla V100 GPU for the timing benchmark)

## Examples

```
python boostnano_eval.py -i INPUT_FAST5_FOLDER -o OUTPUT_FOLDER -m MODEL_FOLDER --replace
```

## Evaluation signals

- RISER classification time must be shorter than Dorado-plus-minimap2 time on both GPU and CPU hardware
- Expected speed ratios in the reported regime: ~44x (RISER-GPU vs AS-GPU), ~371x (RISER-CPU vs AS-GPU), and CPU-based RISER ~16x faster than GPU-based AS averaging 5.8 s per signal
- All compared signals are exactly 6.2 s long, so per-read timing distributions are on equal-length inputs
- Cumulative fraction of signals classified versus time curve for RISER should dominate the adaptive sampling curve across the full time range
- riser_vs_as_timing.csv contains one timing row per read per method (1000 reads) with no missing classification decisions at threshold 0.9

## Limitations

- Timing results are tied to the specific hardware (single NVIDIA Tesla V100 GPU, single CPU) and may vary on other configurations
- Speed comparisons assume BoostNano pre-trimming removes adapter and poly(A) tail differences; without it, internal trimming behavior of the two methods differs
- The benchmark is run offline on fixed 6.2 s signals and does not capture the lag between a reject command and voltage reversal on live hardware, which is beyond RISER's control and varies across platforms

## Evidence

- [abstract] using a GPU or a CPU, RISER is faster than GPU-accelerated basecalling and mapping: "Furthermore, using a GPU or a CPU, RISER is faster than GPU-accelerated basecalling and mapping"
- [readme] boostnano replaces raw signal for basecalling: [evidence span withheld — anchor status: external_doc]
- [methods] mapping options used for the baseline: "mapped to the GENCODE reference transcriptome (release 34, assembly GRCh38.p13) with minimap233 (v2.17) using options -ax map-ont --secondary=no"
- [methods] models run on a single GPU: "All models were built, trained, and tested using PyTorch (v1.9.0)36 with a single NVIDIA Tesla V100 graphics processing unit (GPU)"
