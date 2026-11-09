---
name: cumulative-classification-fraction-visualization
description: Use when benchmarking a read-until classifier against sequence-based
  adaptive sampling and you need a speed comparison table (riser_vs_as_timing.csv)
  and a cumulative-classified-vs-time plot (speed_comparison.png).
when_to_use_negative:
- The goal is to measure RISER's enrichment/depletion performance on read counts rather
  than its classification speed
- Signals have not been adapter/poly(A)-trimmed with BoostNano, so RISER and adaptive
  sampling would operate on unequal effective input lengths
edam_operation: http://edamontology.org/operation_0564
edam_topics:
- http://edamontology.org/topic_0102
tools:
- name: RISER
  role: CNN-based classifier whose per-signal mRNA classification times (threshold
    0.9) are benchmarked against adaptive sampling
  repo: https://github.com/comprna/riser
- name: BoostNano
  role: Pre-trims the sequencing adapter and poly(A) tail from raw signals so both
    methods classify equal-length inputs
  repo: https://github.com/haotianteng/BoostNano
- name: Dorado
  role: Basecalling server (RNA R9.4.1 fast configuration) used in the adaptive-sampling
    arm of the speed comparison
- name: minimap2
  role: Maps basecalled reads (-ax map-ont) against the GENCODE protein_coding transcriptome
    to decide mRNA status and stop the AS timer
- name: PyTorch
  role: Framework in which the RISER model is run on a single NVIDIA Tesla V100 GPU
    during timing
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-024-48673-8
    title: Biochemical-free enrichment or depletion of RNA classes in real-time during
      direct RNA sequencing with RISER
  source_content_hash: sha256:f5fa7f63027e99071346c4ad0be425fc7ffbf60fd4886aae43a0486cbf6ddbc2
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
  edam_operation: http://edamontology.org/operation_0564
  edam_topics:
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7692307692307693
n_claims: 13
---

# cumulative-classification-fraction-visualization

## Summary

Produces per-read and summary classification timings plus a cumulative fraction-of-signals-classified-versus-time plot to compare RISER's real-time raw-signal classification speed against GPU-accelerated Dorado basecalling plus minimap2 mapping in the RISER speed benchmark.

## When to use

Use when benchmarking a read-until classifier against sequence-based adaptive sampling and you need a speed comparison table (riser_vs_as_timing.csv) and a cumulative-classified-vs-time plot (speed_comparison.png).

## When NOT to use

- The goal is to measure RISER's enrichment/depletion performance on read counts rather than its classification speed
- Signals have not been adapter/poly(A)-trimmed with BoostNano, so RISER and adaptive sampling would operate on unequal effective input lengths

## Inputs

- 1000 fixed-length (6.2 s) DRS raw nanopore signals sampled from the mRNA model test set
- BoostNano-trimmed fast5 signals (adapter and poly(A) tail removed)
- GENCODE v34 (GRCh38.p13) reference filtered to protein_coding transcripts
- RISER mRNA classification model (PyTorch CNN)
- Dorado basecalling server with RNA R9.4.1 fast configuration

## Outputs

- riser_vs_as_timing.csv with per-read and summary classification times and RISER-vs-AS speed ratios
- speed_comparison.png showing the cumulative fraction of signals classified versus time

## How to apply

Randomly sample 1000 reads with signals at least 6.2 s long from the mRNA model test set and trim every signal to 6.2 s so both methods see equal-length inputs. Pre-trim the sequencing adapter and poly(A) tail from all test signals with BoostNano to remove internal-trimming differences between the two methods. Run RISER configured to identify mRNA (PyTorch CNN, probability threshold 0.9) and Dorado-based adaptive sampling with the RNA R9.4.1 fast configuration mapping against GENCODE v34 (GRCh38.p13) protein_coding transcripts via minimap2 -ax map-ont, stopping when a signal maps to an mRNA; repeat with both restricted to a single CPU and with RISER on a single GPU. Record per-read classification time for each method, then compute RISER-vs-AS speed ratios (44× on GPU, 371× CPU-RISER vs GPU-AS, with GPU AS averaging 5.8 s per signal) and plot the cumulative fraction of signals classified against elapsed time.

## Related tools

- **RISER** (CNN-based classifier whose per-signal mRNA classification times (threshold 0.9) are benchmarked against adaptive sampling) — https://github.com/comprna/riser
- **BoostNano** (Pre-trims the sequencing adapter and poly(A) tail from raw signals so both methods classify equal-length inputs) — https://github.com/haotianteng/BoostNano
- **Dorado** (Basecalling server (RNA R9.4.1 fast configuration) used in the adaptive-sampling arm of the speed comparison)
- **minimap2** (Maps basecalled reads (-ax map-ont) against the GENCODE protein_coding transcriptome to decide mRNA status and stop the AS timer)
- **PyTorch** (Framework in which the RISER model is run on a single NVIDIA Tesla V100 GPU during timing)

## Examples

```
python3 riser.py --target mRNA --mode deplete --duration 0.1 --kit RNA004
```

## Evaluation signals

- RISER classification time per signal is shorter than Dorado-basecalling-plus-minimap2-mapping time on both GPU and CPU configurations
- GPU-AS average time per signal is around 5.8 s; RISER-vs-AS ratios reproduce ~44× (GPU vs GPU) and 371× (CPU RISER vs GPU AS)
- The cumulative fraction curve for RISER rises to 1.0 faster than the adaptive-sampling curve at equal hardware settings
- All 1000 sampled reads are ≤6.2 s in length and were adapter/poly(A)-trimmed by BoostNano before timing
- riser_vs_as_timing.csv contains per-read times for all 1000 signals for both methods under both hardware configurations

## Limitations

- Results depend on hardware; the benchmark used a single NVIDIA Tesla V100 GPU and single-CPU comparisons, and other configurations may yield different ratios
- Adaptive sampling time depends on the reference transcriptome and mapping stop condition (first mRNA mapping), which can bias per-signal times for reads that never map
- RISER identification accuracy on the 3′ end signal is class-dependent; accurate lncRNA identification from the 3′ end signal remains challenging

## Evidence

- [abstract] RISER is faster than GPU-accelerated basecalling and mapping on both GPU and CPU: "Furthermore, using a GPU or a CPU, RISER is faster than GPU-accelerated basecalling and mapping"
- [methods] BoostNano pre-trimming of adapter and poly(A) tail: "the sequencing adapter and poly(A) tail were removed from the raw nanopore signals using BoostNano"
- [methods] minimap2 mapping against GENCODE transcriptome: "mapped to the GENCODE reference transcriptome (release 34, assembly GRCh38.p13) with minimap2"
- [methods] PyTorch model execution on GPU: "All models were built, trained, and tested using PyTorch (v1.9.0)36 with a single NVIDIA Tesla V100 graphics processing unit (GPU)"
- [discussion] RISER classifies directly from raw signals without basecalling: "it performs the reject decision by directly classifying the raw signal rather than using real-time basecalling and mapping as in ONT’s sequence-based adaptive sampling"
- [readme] README invocation of RISER for mRNA depletion: "python3 riser.py --target mRNA --mode deplete --duration 0.1 --kit RNA004"
