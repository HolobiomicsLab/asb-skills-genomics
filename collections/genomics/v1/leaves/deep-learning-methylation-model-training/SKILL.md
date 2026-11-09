---
name: deep-learning-methylation-model-training
description: Use when training or fine-tuning a Nanopore signal-based 5mC detection
  model for plant genomes and per-site accuracy for CHG/CHH must be improved over
  randomly sampled training data via balancing and denoising.
when_to_use_negative:
- Input data is not Nanopore raw-signal FAST5 (or POD5 with the R10 Dorado pipeline)
  re-squiggled to a plant reference — the 13×16 signal features depend on tombo resquiggle
  of ONT reads
- No matched bisulfite sequencing data exist to provide high-confidence methylated
  (≥0.9 frequency) and unmethylated (zero frequency) training labels
- Goal is only methylation calling with an existing pretrained model (e.g. the released
  arabnrice2-1 checkpoint) rather than retraining with balancing/denoising
edam_operation: http://edamontology.org/operation_3919
edam_topics:
- http://edamontology.org/topic_3295
- http://edamontology.org/topic_3474
tools:
- name: DeepSignal-plant
  role: Extracts sequence/signal features, trains the BiLSTM model (deepsignal_plant
    train), and calls per-read modifications (call_mods) and per-site frequencies
    (call_freq)
  repo: https://github.com/PengNi/deepsignal-plant
- name: Tombo
  role: Re-squiggles raw Nanopore signals to contiguous reference bases (v1.5.1) before
    feature extraction
  repo: https://github.com/nanoporetech/tombo
- name: Guppy
  role: Basecalls R9.4 FAST5 reads (version >=3.6.1) prior to re-squiggling
- name: PyTorch
  role: Deep-learning backend (version >=1.2.0, <=1.11.0) implementing the BRNN with
    LSTM units
  repo: https://pytorch.org/
- name: ont_fast5_api
  role: Multi-to-single FAST5 conversion required before resquiggling
  repo: https://github.com/nanoporetech/ont_fast5_api
- name: vbz_compression
  role: HDF5 plugin needed to read VBZ-compressed FAST5s during resquiggle and call_mods
  repo: https://github.com/nanoporetech/vbz_compression
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-021-26278-9
    title: Genome-wide detection of cytosine methylations in plant from Nanopore data
      using deep learning
  source_content_hash: sha256:59cd52fad916bc8defc9c1cad8d707a5efa6d24f7daae18e1c4698fb8b8a36c5
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/PengNi/deepsignal-plant
  tools:
  - DeepSignal-plant
  - Guppy
  - PyTorch
  - Tombo
  - ont_fast5_api
  - vbz_compression
  edam_operation: http://edamontology.org/operation_3919
  edam_topics:
  - http://edamontology.org/topic_3295
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6842105263157895
n_claims: 19
---

# deep-learning-methylation-model-training

## Summary

Train a BiLSTM (BRNN with LSTM units) DeepSignal-plant model to detect 5mC in CpG, CHG, and CHH contexts from Nanopore reads, using balanced and iteratively denoised bisulfite-labeled training samples to raise per-site Pearson correlation with bisulfite sequencing (e.g. CHH from 0.3696 to 0.7840).

## When to use

Use when training or fine-tuning a Nanopore signal-based 5mC detection model for plant genomes and per-site accuracy for CHG/CHH must be improved over randomly sampled training data via balancing and denoising.

## When NOT to use

- Input data is not Nanopore raw-signal FAST5 (or POD5 with the R10 Dorado pipeline) re-squiggled to a plant reference — the 13×16 signal features depend on tombo resquiggle of ONT reads
- No matched bisulfite sequencing data exist to provide high-confidence methylated (≥0.9 frequency) and unmethylated (zero frequency) training labels
- Goal is only methylation calling with an existing pretrained model (e.g. the released arabnrice2-1 checkpoint) rather than retraining with balancing/denoising

## Inputs

- ~500× A. thaliana R9.4 raw FAST5 reads (multi-read FAST5, e.g. fast5s.sample.tar.gz)
- Guppy-basecalled reads and tombo re-squiggled FAST5s (corrected group RawGenomeCorrected_000)
- A. thaliana genome reference GCF_000001735.4_TAIR10.1_genomic.fna
- Bisulfite sequencing replicates used to derive high-confidence methylated/unmethylated site labels
- Extracted per-cytosine feature table (13-mer sequence + 13×16 normalized signal features)

## Outputs

- Two trained BiLSTM model checkpoints per motif (random-baseline and denoised), e.g. model.dp2.CNN.arabnrice2-1_120m_R9.4plus_tem.bn13_sn16.both_bilstm.epoch6.ckpt
- Per-read modification calls from deepsignal_plant call_mods
- Per-site methylation frequency files from deepsignal_plant call_freq
- Per-site Pearson correlations of Nanopore vs bisulfite methylation frequencies for CHG and CHH (target: 0.9122 CHG, 0.7840 CHH)

## How to apply

First prepare features: basecall ~500× A. thaliana R9.4 FAST5 reads with Guppy (>=3.6.1, dna_r9.4.1_450bps_hac_prom.cfg), re-squiggle raw signals against GCF_000001735.4_TAIR10.1_genomic.fna with tombo resquiggle (v1.5.1, --corrected-group RawGenomeCorrected_000), then extract 13-mer sequence plus 13×16 median-shift/MAD-normalized signal features per cytosine with deepsignal_plant extract (--motifs C). Label cytosines from bisulfite replicates: methylated if ≥5 reads and ≥0.9 methylation frequency, unmethylated if ≥5 reads and zero frequency. Build two training sets per motif capped at 20 million samples (half positive, half negative): a random baseline and a balanced set refined by the iterative denoising algorithm (I=10, R=3, E=3), which removes positive samples whose mean predicted methylation probability is below 0.5; for CHG/CHH take the union of high-confidence methylated sites across three bisulfite replicates. Train the BiLSTM twice with deepsignal_plant train (Adam optimizer, cross-entropy loss, 99:1 train/validation split, dropout, early stopping) — once on the random set, once on the denoised set — then call mods on held-out reads with deepsignal_plant call_mods and aggregate per-site frequencies with call_freq. Judge success by per-site Pearson correlation against each bisulfite replicate: the denoised model must improve from 0.8464 to 0.9122 (CHG) and from 0.3696 to 0.7840 (CHH), reproducing Fig. 1d.

## Related tools

- **DeepSignal-plant** (Extracts sequence/signal features, trains the BiLSTM model (deepsignal_plant train), and calls per-read modifications (call_mods) and per-site frequencies (call_freq)) — https://github.com/PengNi/deepsignal-plant
- **Tombo** (Re-squiggles raw Nanopore signals to contiguous reference bases (v1.5.1) before feature extraction) — https://github.com/nanoporetech/tombo
- **Guppy** (Basecalls R9.4 FAST5 reads (version >=3.6.1) prior to re-squiggling)
- **PyTorch** (Deep-learning backend (version >=1.2.0, <=1.11.0) implementing the BRNN with LSTM units) — https://pytorch.org/
- **ont_fast5_api** (Multi-to-single FAST5 conversion required before resquiggling) — https://github.com/nanoporetech/ont_fast5_api
- **vbz_compression** (HDF5 plugin needed to read VBZ-compressed FAST5s during resquiggle and call_mods) — https://github.com/nanoporetech/vbz_compression

## Examples

```
deepsignal_plant train --train_file train.features.tsv --valid_file valid.features.tsv --model_dir train_model_c --motifs C --nproc 30 --epochs 10
```

## Evaluation signals

- Denoised model raises per-site Pearson correlation vs bisulfite from 0.8464 to 0.9122 for CHG and from 0.3696 to 0.7840 for CHH, reproducing Fig. 1d
- Training set per motif contains at most 20 million samples, exactly half positive and half negative
- Denoising (I=10, R=3, E=3) removes positive samples whose mean predicted methylation probability is <0.5
- All 99:1 train/validation splits converge with early stopping; cross-entropy loss decreases without divergence
- Called methylation frequencies on held-out ~100× reads correlate highly with each of the three bisulfite replicates

## Limitations

- Gold-standard training labels depend on bisulfite sequencing, so model performance is bounded by bisulfite quality (incomplete conversion, DNA degradation, poor coverage in repetitive regions)
- CHG/CHH high-confidence methylated labels use the union of three bisulfite replicates, a lower-stringency choice that may introduce label noise
- VBZ-compressed FAST5s fail in tombo resquiggle and call_mods unless the ont-vbz-hdf-plugin is installed and HDF5_PLUGIN_PATH is set
- Tombo is deprecated/unsupported by ONT; PyTorch must be pinned between 1.2.0 and 1.11.0
- No accession numbers are given for the Nanopore and bisulfite training/evaluation data, limiting full reproduction

## Evidence

- [discussion] Iterative denoising of training samples (Algorithm 2, I=10, R=3, E=3): "In the denoising algorithm, we set I = 10, R = 3, E = 3 as default"
- [discussion] Denoising removes low-probability positive samples: "if mean(score(s)) < 0.5 thenSpos.remove(s)"
- [discussion] Training subsample cap: "we randomly subsample at most 20 million (half positive and half negative) samples as the training dataset for each motif"
- [discussion] Bisulfite label thresholds: "cytosines covered with at least five reads and had at least 0.9 methylation frequency as high-confidence methylated sites"
- [discussion] Optimizer and loss: "We use Adam optimizer48 to learn model parameters on the training dataset by minimizing the loss calculated by cross-entropy"
- [intro] Balancing and denoising improves CHG/CHH performance: "we develop a sample selection strategy to balance and denoise training samples, which can significantly improve the performances of the trained models, especially for CHH and CHG"
- [readme] README installation requirements: "Guppy (version>=3.6.1) is also required, which can be downloaded from [Nanopore Community (login required)]"
- [readme] VBZ plugin failure mode: "Please try adding ont-vbz-hdf-plugin to your environment as follows when all fast5s failed in `tombo resquiggle` and/or `deepsignal_plant call_mods`"
