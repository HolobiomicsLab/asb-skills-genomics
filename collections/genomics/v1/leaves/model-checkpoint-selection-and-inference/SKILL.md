---
name: model-checkpoint-selection-and-inference
description: Use when a trained methylation model checkpoint must be chosen from a
  training run and applied to held-out POD5/FAST5 signal plus move-table BAM to generate
  per-read 5mC probability calls.
when_to_use_negative:
- The goal is per-site methylation frequency aggregation rather than per-read probabilities
  — use a post-processing step that combines per-read calls instead
- Detection of modifications other than 5mCpG is needed, since DeepMod2 currently
  provides models only for 5mCpG
- Input is raw un-basecalled signal without a move-table BAM, which DeepMod2 requires
  to align signal to basecalls
edam_operation: http://edamontology.org/operation_3919
edam_topics:
- http://edamontology.org/topic_3295
- http://edamontology.org/topic_0736
tools:
- name: DeepMod2
  role: Provides the BiLSTM inference pipeline, checkpoint format, --motif CG 0 CpG
    restriction, and per-read TSV output
  repo: https://github.com/WGLab/DeepMod2
- name: Dorado
  role: Basecalls held-out POD5 files with --emit-moves to produce the move-table
    BAM consumed at inference
  repo: https://github.com/nanoporetech/dorado
- name: PyTorch
  role: Framework for loading the trained BiLSTM checkpoint and running forward passes
    over feature matrices
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-024-45778-y
    title: A signal processing and deep learning framework for methylation detection
      using Oxford Nanopore sequencing
  source_content_hash: sha256:e8807aa32effef9268c06bb562a720bf06de96a5f0f4d3137564732026711451
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/WGLab/DeepMod2
  tools:
  - DeepMod2
  - Dorado
  - PyTorch
  - samtools
  edam_operation: http://edamontology.org/operation_3919
  edam_topics:
  - http://edamontology.org/topic_3295
  - http://edamontology.org/topic_0736
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 10
---

# model-checkpoint-selection-and-inference

## Summary

Select the final trained checkpoint (e.g. model.epoch10 from the DeepMod2 BiLSTM run) and run inference on held-out ONT signal data to produce per-read 5mC methylation probabilities at CpG motifs, written as a per-read BED/TSV with scores in [0,1].

## When to use

Use when a trained methylation model checkpoint must be chosen from a training run and applied to held-out POD5/FAST5 signal plus move-table BAM to generate per-read 5mC probability calls.

## When NOT to use

- The goal is per-site methylation frequency aggregation rather than per-read probabilities — use a post-processing step that combines per-read calls instead
- Detection of modifications other than 5mCpG is needed, since DeepMod2 currently provides models only for 5mCpG
- Input is raw un-basecalled signal without a move-table BAM, which DeepMod2 requires to align signal to basecalls

## Inputs

- Trained model checkpoint files (model.cfg + model.epoch10) from the BiLSTM run
- Held-out POD5 signal files basecalled with Dorado --emit-moves into a move-table BAM aligned to the GRCh38 reference (ref.fa with .fai index)
- CpG motif specification (--motif CG 0)

## Outputs

- Per-read tab-separated BED file with columns read_name, chromosome, ref_position, read_position, strand, methylation_score, read_phase, ref_cpg, with methylation_score in [0,1]
- BAM annotated with MM/ML methylation modification tags

## How to apply

After training the BiLSTM classifier (batch_size 128, 10 epochs, lr 0.01, L2 0.01, seed 0, 50% validation split), pick the final-epoch checkpoint — for DeepMod2 this is the model.cfg plus final model.epoch10 checkpoint, which reached 78.46% validation accuracy. Run inference on the held-out ONT slice, restricting predictions to CpG motif positions via --motif CG 0, with the BAM basecalled by Dorado (dna_r10.4.1_e8.2_400bps_hac@v4.3.0, --emit-moves) and aligned to the GRCh38 reference. The model outputs a sigmoid methylation probability per CpG site; a threshold of 0.5 defines a methylated per-read call, though user-specified thresholds can exclude intermediate probabilities. Write per-read predictions as a tab-separated BED matching the DeepMod2 format (read_name, chromosome, ref_position, read_position, strand, methylation_score, read_phase, ref_cpg) and optionally add MM/ML tags to the BAM.

## Related tools

- **DeepMod2** (Provides the BiLSTM inference pipeline, checkpoint format, --motif CG 0 CpG restriction, and per-read TSV output) — https://github.com/WGLab/DeepMod2
- **Dorado** (Basecalls held-out POD5 files with --emit-moves to produce the move-table BAM consumed at inference) — https://github.com/nanoporetech/dorado
- **PyTorch** (Framework for loading the trained BiLSTM checkpoint and running forward passes over feature matrices)

## Examples

```
python PATH_TO_DEEPMOD2/deepmod2 detect --bam reads.bam --input INPUT_DIR --model MODEL --file_type pod5 --threads 8 --ref ref.fa --output MOD_CALLS
```

## Evaluation signals

- Per-read classification accuracy on the validation split approaches the reported 78.46% at epoch 10
- Per-read output file exists and every methylation_score lies in [0,1]
- Predictions are emitted only at CpG motif positions (--motif CG 0)
- At the default 0.5 threshold, methylated/unmethylated calls can be cross-checked against ground-truth sites defined as ≥90% or <10% methylation with ≥10X coverage

## Limitations

- A fixed 0.5 threshold predicts methylated calls; filtering intermediate predictions reduces coverage and statistical power, especially for low-coverage samples
- Move tables only approximately demarcate signal boundaries between consecutive bases, which can produce uninformative signal summary statistics feeding inference
- DeepMod2 models show a slight bias to overestimate methylation at some CpG sites, possibly from residual heterogeneity in training labels

## Evidence

- [results] A feature matrix is fed to a BiLSTM or Transformer model that makes per-read predictions for each CpG site on the read: "A feature matrix is fed to a BiLSTM or Transformer model that makes per-read predictions for each CpG site on the read"
- [results] A threshold of 0.5 for probability score is used to predict a CpG site as methylated, however, user specified thresholds can be used: "A threshold of 0.5 for probability score is used to predict a CpG site as methylated, however, user specified thresholds can be used to exclude intermediate probability predictions"
- [readme] Basecall your FAST5/POD5 files with Dorado (using --emit-moves) or Guppy (using --bam_out --moves_out) parameters to get a BAM file with move tables: [evidence span withheld — anchor status: external_doc]
- [results] DeepMod2 adds per-read predictions to the input BAM file as modification tags and also produces a tab separated text file output: "DeepMod2 adds per-read predictions to the input BAM file as modification tags and also produces a tab separated text file output"
