---
name: bilstm-model-training
description: Use when you need to (re)train DeepSignal-plant's BiLSTM 5mC model from
  Nanopore signal feature tables, or when building/curating the balanced, denoised
  sample set (Algorithm 1 + Algorithm 2) that precedes BRNN-LSTM training.
when_to_use_negative:
- Input is a basecalled FASTQ/BAM without raw signal FAST5s — training requires 13x16
  signal features from Tombo-resquiggled raw reads
- You only want per-read methylation calls from an already-trained model — use `deepsignal_plant
  call_mods` + `call_freq` instead of retraining
- The organism/data are not plant Nanopore reads in CpG/CHG/CHH contexts — existing
  general-context models such as DeepSignal were limited to CpG, motivating this plant-specific
  model
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_0654
- http://edamontology.org/topic_0102
tools:
- name: DeepSignal-plant
  role: Provides the BiLSTM model architecture, feature extraction (`deepsignal_plant
    extract`), denoising Algorithms 1 and 2, and the training loop; implemented in
    Python3 and PyTorch
  repo: https://github.com/PengNi/deepsignal-plant
- name: PyTorch
  role: Backend framework for training the BRNN-with-LSTM model (version >=1.2.0,
    <=1.11.0)
  repo: https://pytorch.org/
- name: Tombo
  role: Upstream re-squiggle of raw signals to genome-reference bases (version 1.5.1),
    producing the per-base signals that become training features
  repo: https://github.com/nanoporetech/tombo
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-021-26278-9
    title: Genome-wide detection of cytosine methylations in plant from Nanopore data
      using deep learning
  source_content_hash: sha256:ebe7586c55b224335da1aa253a11c1530600aa9597ee4d5ac4907ee93a709c75
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/PengNi/deepsignal-plant
  tools:
  - DeepSignal-plant
  - PyTorch
  - Tombo
  edam_operation: http://edamontology.org/operation_3935
  edam_topics:
  - http://edamontology.org/topic_0654
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7692307692307693
n_claims: 13
---

# bilstm-model-training

## Summary

Trains DeepSignal-plant's bidirectional-recurrent (BiLSTM) model to detect 5mC in CpG/CHG/CHH contexts from Nanopore signal features (13-mer sequence plus 13x16 normalized signal per cytosine), using a balanced, iteratively denoised training set and a cross-entropy-optimized training loop.

## When to use

Use when you need to (re)train DeepSignal-plant's BiLSTM 5mC model from Nanopore signal feature tables, or when building/curating the balanced, denoised sample set (Algorithm 1 + Algorithm 2) that precedes BRNN-LSTM training.

## When NOT to use

- Input is a basecalled FASTQ/BAM without raw signal FAST5s — training requires 13x16 signal features from Tombo-resquiggled raw reads
- You only want per-read methylation calls from an already-trained model — use `deepsignal_plant call_mods` + `call_freq` instead of retraining
- The organism/data are not plant Nanopore reads in CpG/CHG/CHH contexts — existing general-context models such as DeepSignal were limited to CpG, motivating this plant-specific model

## Inputs

- Tombo-resquiggled Nanopore FAST5 reads (R9.4 1D), after Guppy >=3.6.1 basecalling
- Training feature file from deepsignal_plant extract: 13-mer sequence + 13x16 signal features per targeted cytosine
- Bisulfite-sequencing gold-standard methylation calls for labeling positive/negative sites
- Model checkpoint file (e.g. model.dp2.CNN.arabnrice2-1_120m_R9.4plus_tem.bn13_sn16.both_bilstm.epoch6.ckpt) if warm-starting or resuming

## Outputs

- Trained BiLSTM (BRNN-with-LSTM) model checkpoint in PyTorch (.ckpt) producing Pm/Pum methylation probabilities per sample
- Denoised, balanced training file of retained methylated/unmethylated samples (>=99% retention or 10 iterations) usable by deepsignal_plant train
- Validation split metrics (cross-entropy loss on the 1% held-out validation set)

## How to apply

Build training inputs with `deepsignal_plant extract`: per targeted cytosine, a 13-mer sequence plus a 13x16 signal-feature matrix from raw signals normalized by median shift and MAD scale (after Guppy basecalling and Tombo 1.5.1 resquiggle). Label positives/negatives from bisulfite gold standards (positive: >=5 reads and >=0.9 methylation frequency; negative: >=5 reads and zero methylation frequency), then balance k-mer types and per-k-mer read coverage (Algorithm 1) and cap at 20 million samples per motif (half positive, half negative). Denoise via Algorithm 2 with defaults I=10, R=3, E=3: per iteration run two-fold cross prediction (R=3 rounds) with the BiLSTM model, remove positive samples with mean predicted methylation probability mean(score(s)) < 0.5, re-balance, and stop when retention reaches >=99% or after 10 iterations. Train the BRNN-with-LSTM model (softmax outputs Pm/Pum) with the Adam optimizer minimizing cross-entropy loss on a 99:1 train/validation split, using dropout and early stopping; a one-model design covers all three contexts, as trained on A. thaliana and O. sativa R9.4 1D reads.

## Related tools

- **DeepSignal-plant** (Provides the BiLSTM model architecture, feature extraction (`deepsignal_plant extract`), denoising Algorithms 1 and 2, and the training loop; implemented in Python3 and PyTorch) — https://github.com/PengNi/deepsignal-plant
- **PyTorch** (Backend framework for training the BRNN-with-LSTM model (version >=1.2.0, <=1.11.0)) — https://pytorch.org/
- **Tombo** (Upstream re-squiggle of raw signals to genome-reference bases (version 1.5.1), producing the per-base signals that become training features) — https://github.com/nanoporetech/tombo

## Evaluation signals

- Denoising loop terminates with >=99% sample retention (or after I=10 iterations), yielding a balanced positive/negative training file
- After balancing, positive and negative counts are equal per k-mer motif and capped at <=10 million each (20M total, half positive/half negative per motif)
- All removed positives had mean(score(s)) < 0.5 under the two-fold cross prediction (R=3 rounds per iteration)
- Validation cross-entropy loss on the 1% held-out split decreases with early stopping; dropout active during training
- Downstream check: trained model's per-site methylation frequencies correlate with bisulfite sequencing across all three CpG/CHG/CHH contexts

## Limitations

- Training gold standard is bisulfite sequencing, so model performance is bounded by bisulfite quality (incomplete conversion, DNA degradation, no coverage in repetitive regions)
- For A. thaliana CHG/CHH, high-confidence methylated labels use the union of three replicates (lower stringency), a potential label-noise source
- PyTorch version must be >=1.2.0 and <=1.11.0; Tombo (required for resquiggling) is deprecated by Oxford Nanopore, so the R9 FAST5 pipeline depends on unsupported software
- The article gives no accession numbers for the Nanopore/bisulfite training data (~500x training reads, ~100x evaluation), limiting exact reproduction

## Evidence

- [discussion] Training with Adam optimizer and cross-entropy loss: "We use Adam optimizer48 to learn model parameters on the training dataset by minimizing the loss calculated by cross-entropy"
- [discussion] BRNN with LSTM units is the model: "BRNN29 with LSTM units30 is used in the model of DeepSignal-plant"
- [discussion] Denoising removal criterion: "if mean(score(s)) < 0.5 thenSpos.remove(s)"
- [discussion] Denoising default hyperparameters: "In the denoising algorithm, we set I = 10, R = 3, E = 3 as default"
- [discussion] Training sample cap per motif: "we randomly subsample at most 20 million (half positive and half negative) samples as the training dataset for each motif"
- [readme] README: BiLSTM on Python3/PyTorch: "deepsignal-plant applies BiLSTM to detect methylation from Nanopore reads. It is built on **Python3** and **PyTorch**."
