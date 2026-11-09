---
name: runtime-memory-profiling
description: Use when benchmarking whether a pruned deep learning model (e.g., DeepMod2
  BiLSTM) runs efficiently on CPU-only hardware, and you need measured runtime, peak
  memory, and accuracy trade-offs.
when_to_use_negative:
- When the goal is GPU inference benchmarking—the reported benefit (5-fold CPU speedup)
  is specific to pruned-model CPU-only execution
- When evaluating methylation accuracy alone without resource constraints; runtime/memory
  profiling adds overhead unnecessary for pure F1 comparison
- When the input signal files are not POD5/FAST5 basecalled with move table output
  (Guppy --bam_out --moves_out or Dorado --emit-moves), since DeepMod2 cannot align
  signals to basecalls
edam_operation: http://edamontology.org/operation_0334
edam_topics:
- http://edamontology.org/topic_3168
- http://edamontology.org/topic_0654
tools:
- name: DeepMod2
  role: Runs the detect module with the pruned BiLSTM model on CPU-only hardware to
    produce per-read methylation calls being profiled
  repo: https://github.com/WGLab/DeepMod2
- name: Dorado
  role: Basecalls POD5 files with --emit-moves (model dna_r10.4.1_e8.2_400bps_hac@v4.3.0,
    v0.5.3) to generate the BAM input with move tables
  repo: https://github.com/nanoporetech/dorado
- name: samtools
  role: Used in the alignment step (samtools fastq -T mv,ts; samtools view) to preserve
    move-table tags when aligning reads with minimap2
  repo: https://github.com/nanoporetech/dorado
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-024-45778-y
    title: A signal processing and deep learning framework for methylation detection
      using Oxford Nanopore sequencing
  source_content_hash: sha256:83520cac16d7b155311273d0081694f1686fa05c3eb3f14dbe8d917d580db080
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/WGLab/DeepMod2
  tools:
  - DeepMod2
  - Dorado
  - samtools
  edam_operation: http://edamontology.org/operation_0334
  edam_topics:
  - http://edamontology.org/topic_3168
  - http://edamontology.org/topic_0654
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 10
---

# runtime-memory-profiling

## Summary

Profile wall-clock runtime and peak memory usage of a computational pipeline—in this context, CPU-only DeepMod2 methylation inference with a pruned BiLSTM model—to quantify whether model pruning enables efficient CPU execution.

## When to use

Use when benchmarking whether a pruned deep learning model (e.g., DeepMod2 BiLSTM) runs efficiently on CPU-only hardware, and you need measured runtime, peak memory, and accuracy trade-offs.

## When NOT to use

- When the goal is GPU inference benchmarking—the reported benefit (5-fold CPU speedup) is specific to pruned-model CPU-only execution
- When evaluating methylation accuracy alone without resource constraints; runtime/memory profiling adds overhead unnecessary for pure F1 comparison
- When the input signal files are not POD5/FAST5 basecalled with move table output (Guppy --bam_out --moves_out or Dorado --emit-moves), since DeepMod2 cannot align signals to basecalls

## Inputs

- ON T POD5 sample dataset from github.com/WGLab/DeepMod2/releases/tag/v0.3.0
- Pruned DeepMod2 BiLSTM model configuration and checkpoint files (model.cfg, model_checkpoint)
- GRCh38 reference FASTA (GRCh38.fa)
- Dorado basecalled BAM with move tables (dorado basecaller ... --emit-moves)
- Minimap2-aligned BAM retaining mv tags
- Per-read methylation benchmark ground truth

## Outputs

- cpu_runtime_accuracy. containing wall-clock runtime (seconds), peak memory usage (GB), per-read F1-score, and pruning configuration metadata
- Per-read BED file of methylation calls from DeepMod2 detect
- Methylation-tagged BAM (MM/ML tags)

## How to apply

Basecall the ONT POD5 sample dataset (github.com/WGLab/DeepMod2/releases/tag/v0.3.0) with Dorado v0.5.3 using model dna_r10.4.1_e8.2_400bps_hac@v4.3.0 and --emit-moves, then align to the GRCh38 reference with minimap2 -ax map-ont while copying move-table tags (samtools fastq -T mv,ts). Run DeepMod2's detect module on CPU-only hardware with the pruned model checkpoint supplied as --model model.cfg,model_checkpoint, --file_type pod5, an aligned BAM input, --ref GRCh38.fa, and --threads 8. Wrap the command in /usr/bin/time -v to record wall-clock runtime (seconds) and peak memory (GB) during methylation calling. Compute the per-read F1-score from the per-read BED output using the default 0.5 probability threshold for calling a CpG site methylated, against the benchmark ground truth. Write runtime, peak memory, F1-score, and pruning configuration metadata into cpu_runtime_accuracy. so the speedup (~5-fold: 13 h pruned vs 60 h unpruned CPU inference) and accuracy retention can be validated together.

## Related tools

- **DeepMod2** (Runs the detect module with the pruned BiLSTM model on CPU-only hardware to produce per-read methylation calls being profiled) — https://github.com/WGLab/DeepMod2
- **Dorado** (Basecalls POD5 files with --emit-moves (model dna_r10.4.1_e8.2_400bps_hac@v4.3.0, v0.5.3) to generate the BAM input with move tables) — https://github.com/nanoporetech/dorado
- **samtools** (Used in the alignment step (samtools fastq -T mv,ts; samtools view) to preserve move-table tags when aligning reads with minimap2) — https://github.com/nanoporetech/dorado

## Examples

```
/usr/bin/time -v python deepmod2 detect --bam aligned.bam --input POD5_DIR --model model.cfg,model_checkpoint --file_type pod5 --threads 8 --ref GRCh38.fa --output mod_calls
```

## Evaluation signals

- Recorded per-read F1-score falls in the ~0.95 range reported for DeepMod2 on human cell-line benchmarks (e.g., 95.7–97.1% on R9.4.1 Ashkenazim trio)
- cpu_runtime_accuracy. contains non-empty runtime, memory, F1, and pruning-config fields
- CPU inference with the pruned model completes in ~13 h vs ~60 h unpruned, confirming the ~5-fold speedup
- Peak memory usage (GB) is captured via /usr/bin/time -v around the deepmod2 detect command
- Alignment BAM retains move-table (mv) tags, verified via samtools fastq -T mv,ts round-trip, so methylation calling proceeds without signal-to-basecall alignment failures

## Limitations

- Move tables provide only approximate signal-to-basecall alignment, which can cause uninformative signal summary statistics in some cases
- DeepMod2 models may slightly overestimate methylation at some CpG sites, possibly from residual heterogeneity in training labels
- DeepMod2 only provides models for 5mCpG methylation due to lack of available datasets for other modification types
- Users can set read length or mean quality score thresholds to filter reads, which will change the reads profiled and downstream F1

## Evidence

- [abstract] DeepMod2 can run efficiently on central processing unit (CPU) through model pruning: "DeepMod2 can run efficiently on central processing unit (CPU) through model pruning"
- [results] It requires POD5/FAST5 files to be basecalled using Guppy or Dorado with move table output: "It requires POD5/FAST5 files to be basecalled using Guppy or Dorado with move table output to allow signal alignment with the basecalls"
- [results] A threshold of 0.5 for probability score is used to predict a CpG site as methylated: "A threshold of 0.5 for probability score is used to predict a CpG site as methylated, however, user specified thresholds can be used"
- [readme] dorado basecaller MODEL INPUT_DIR --emit-moves > basecall.bam: [evidence span withheld — anchor status: external_doc]
- [results] For R9.4.1 datasets of the Ashkenazim trio HG002, HG003 and HG004, the F1-score of DeepMod2 models is in the 95.7–97.1% range: "For R9.4.1 datasets of the Ashkenazim trio HG002, HG003 and HG004, the F1-score of DeepMod2 models is in the 95.7–97.1% range"
