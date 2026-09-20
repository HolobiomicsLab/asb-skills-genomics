---
name: model-pruning-cpu-efficiency-benchmarking
description: Use when you must run DeepMod2 methylation inference on CPU-only hardware,
  or when quantifying how model pruning trades inference wall-clock time and memory
  against per-read F1 accuracy.
when_to_use_negative:
- When GPU hardware is available — the pruning benchmark targets CPU-only inference,
  and unpruned models on GPU are the article's default performant configuration
- When input is FAST5 rather than POD5 — this skill's invocation is parameterized
  for --file_type pod5 with Dorado --emit-moves basecalls
- When the goal is per-site methylation frequency evaluation rather than CPU runtime/accuracy
  benchmarking with per-read F1
edam_operation: http://edamontology.org/operation_3208
edam_topics:
- http://edamontology.org/topic_3295
- http://edamontology.org/topic_3168
tools:
- name: DeepMod2
  role: Runs the detect module with the pruned BiLSTM model for CPU methylation calling
    and per-read predictions
  repo: github.com/WGLab/DeepMod2
- name: Dorado
  role: Basecalls POD5 with --emit-moves to produce BAM files with move tables required
    for signal alignment
  repo: https://github.com/nanoporetech/dorado
- name: minimap2
  role: Aligns basecalled reads to GRCh38 with -ax map-ont while preserving move-table
    tags
- name: samtools
  role: Copies move tags (mv,ts) through basecall-to-alignment conversion and BAM
    handling
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-024-45778-y
    title: A signal processing and deep learning framework for methylation detection
      using Oxford Nanopore sequencing
  source_content_hash: sha256:d9e7bf7370a951bc117fd6eafcab79990cef3ace95246e3f0ba8cdf72d1273dc
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - DeepMod2
  - Dorado
  - minimap2
  - samtools
  edam_operation: http://edamontology.org/operation_3208
  edam_topics:
  - http://edamontology.org/topic_3295
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# model-pruning-cpu-efficiency-benchmarking

## Summary

Benchmark the CPU-only runtime and accuracy of the pruned DeepMod2 BiLSTM model for 5mC methylation calling on ONT POD5 data, where pruning reduces inference from ~60 h to ~13 h (~5-fold speedup) while preserving a per-read F1-score in the ~0.95 range.

## When to use

Use when you must run DeepMod2 methylation inference on CPU-only hardware, or when quantifying how model pruning trades inference wall-clock time and memory against per-read F1 accuracy.

## When NOT to use

- When GPU hardware is available — the pruning benchmark targets CPU-only inference, and unpruned models on GPU are the article's default performant configuration
- When input is FAST5 rather than POD5 — this skill's invocation is parameterized for --file_type pod5 with Dorado --emit-moves basecalls
- When the goal is per-site methylation frequency evaluation rather than CPU runtime/accuracy benchmarking with per-read F1

## Inputs

- ON SIMPOD5 signal files from R10.4.1 flowcells (e.g. DeepMod2 v0.3.0 sample dataset)
- GRCh38 reference FASTA (GRCh38.fa)
- Pruned DeepMod2 model configuration and checkpoint files (model.cfg, model_checkpoint)
- CPU-only execution environment with >=8 threads

## Outputs

- Methylation-tagged BAM file with MM/ML tags from DeepMod2 detect
- Per-read BED methylation calls at the 0.5 probability threshold
- cpu_runtime_accuracy. with wall-clock runtime (seconds), peak memory (GB), per-read F1-score, and pruning configuration metadata

## How to apply

Basecall the ONT POD5 sample dataset (github.com/WGLab/DeepMod2/releases/tag/v0.3.0) with Dorado v0.5.3 using model dna_r10.4.1_e8.2_400bps_hac@v4.3.0 and --emit-moves so move tables are retained, then align reads to the GRCh38 reference with minimap2 -ax map-ont while copying the move tags (mv,ts). Launch DeepMod2's detect module in a CPU-only environment supplying the pruned model as --model model.cfg,model_checkpoint, --file_type pod5, the aligned BAM, --ref GRCh38.fa, and --threads 8. Profile the methylation-calling process with /usr/bin/time -v to capture wall-clock runtime and peak memory; the unpruned BiLSTM requires ~60 h CPU inference versus ~13 h with pruning (~5-fold speedup). Compute per-read F1-score from the per-read BED output using the default 0.5 probability threshold against benchmark ground truth, and record runtime, memory, F1, and pruning-config metadata in cpu_runtime_accuracy..

## Related tools

- **DeepMod2** (Runs the detect module with the pruned BiLSTM model for CPU methylation calling and per-read predictions) — github.com/WGLab/DeepMod2
- **Dorado** (Basecalls POD5 with --emit-moves to produce BAM files with move tables required for signal alignment) — https://github.com/nanoporetech/dorado
- **minimap2** (Aligns basecalled reads to GRCh38 with -ax map-ont while preserving move-table tags)
- **samtools** (Copies move tags (mv,ts) through basecall-to-alignment conversion and BAM handling)

## Examples

```
/usr/bin/time -v python deepmod2 detect --bam aligned.bam --input pod5_dir --model model.cfg,model_checkpoint --file_type pod5 --ref GRCh38.fa --threads 8 --output mod_calls
```

## Evaluation signals

- Recorded per-read F1-score falls in the ~0.95 range reported for DeepMod2 on human cell-line benchmarks
- cpu_runtime_accuracy. contains non-empty runtime, memory, F1, and pruning-config fields
- Pruned-model CPU inference time is markedly lower than the ~60 h unpruned baseline (~13 h, ~5-fold speedup)
- Basecalled BAM retains move tables (verified after minimap2 alignment with mv,ts tags copied via samtools)
- Peak memory usage is captured by /usr/bin/time -v and finite

## Limitations

- Move tables provide only approximate signal-to-basecall alignment, causing uninformative signal summary statistics in some cases
- DeepMod2 only provides models for 5mCpG methylation; no training data exists for other modification types
- Runtime/memory numbers are hardware- and thread-count-dependent (workflow specifies --threads 8), so absolute times may not transfer across CPUs
- A slight bias to overestimate methylation at some CpG sites exists, possibly from residual heterogeneity in training labels

## Evidence

- [abstract] DeepMod2 can run efficiently on central processing unit (CPU) through model pruning: "DeepMod2 can run efficiently on central processing unit (CPU) through model pruning"
- [intro] Per-read F1 ~95%: "DeepMod2 achieves ~95% and ~99% F1-score, respectively, with a correlation of r > 0.95 with short-read sequencing"
- [results] 0.5 threshold for methylation calls: "A threshold of 0.5 for probability score is used to predict a CpG site as methylated, however, user specified thresholds can be used"
- [readme] Dorado basecalling with move tables: [evidence span withheld — anchor status: external_doc]
- [methods] Sample dataset source: "We will use sample dataset from https://github.com/WGLab/DeepMod2/releases/tag/v0.3.0."
- [results] Move table alignment caveat: "the move table does not always accurately demarcate boundaries between signals associated with consecutive bases, which can lead to uninformative signal summary statistics"
