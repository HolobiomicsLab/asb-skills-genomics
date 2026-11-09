---
name: nanopore-methylation-feature-handling
description: Use when raw Nanopore FAST5 signals must be turned into balanced, denoised
  training samples for 5mC detection (e.g. building DeepSignal-plant training sets
  with CpG/CHG/CHH motifs from re-squiggled reads).
when_to_use_negative:
- Input is bisulfite sequencing data rather than native Nanopore signal — this skill
  builds features from raw electrical signals, not converted reads
- Goal is per-site methylation frequency reporting from an existing call_mods output
  (use call_freq instead of re-extracting and denoising features)
- A pretrained 5mC checkpoint already exists (e.g. model.dp2.CNN.arabnrice2-1_120m_R9.4plus_tem.bn13_sn16.both_bilstm.epoch6.ckpt)
  and no retraining is needed
tools:
- name: DeepSignal-plant
  role: Extracts sequence/signal features per cytosine, performs balancing and iterative
    denoising, and trains the BiLSTM methylation model
  repo: https://github.com/PengNi/deepsignal-plant
- name: Tombo
  role: Re-squiggles raw signals of reads to contiguous bases in the genome reference
    before feature extraction (version 1.5.1)
  repo: https://github.com/nanoporetech/tombo
- name: Guppy
  role: Basecalls all Nanopore reads (version >=3.6.1) prior to re-squiggling
- name: PyTorch
  role: Implements the BRNN-with-LSTM model and Adam-optimized training used in the
    denoising cross-prediction loop
- name: ont_fast5_api
  role: Converts multi-read FAST5 to single-read FAST5 (multi_to_single_fast5) so
    Tombo and deepsignal_plant can read them
  repo: https://github.com/nanoporetech/ont_fast5_api
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-021-26278-9
    title: Genome-wide detection of cytosine methylations in plant from Nanopore data
      using deep learning
  source_content_hash: sha256:29b2481fde4ff5b4966d2c29158ba6fdf3c4d00929b4c95e0ecc33bd3589af48
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
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9230769230769231
n_claims: 13
---

# nanopore-methylation-feature-handling

## Summary

Preparation of raw Nanopore signal features for plant 5mC model training: re-squiggled signals are converted into 13-mer sequence plus 13x16 signal features per targeted cytosine, balanced per k-mer motif, and iteratively denoised by two-fold cross-prediction before training DeepSignal-plant.

## When to use

Use when raw Nanopore FAST5 signals must be turned into balanced, denoised training samples for 5mC detection (e.g. building DeepSignal-plant training sets with CpG/CHG/CHH motifs from re-squiggled reads).

## When NOT to use

- Input is bisulfite sequencing data rather than native Nanopore signal — this skill builds features from raw electrical signals, not converted reads
- Goal is per-site methylation frequency reporting from an existing call_mods output (use call_freq instead of re-extracting and denoising features)
- A pretrained 5mC checkpoint already exists (e.g. model.dp2.CNN.arabnrice2-1_120m_R9.4plus_tem.bn13_sn16.both_bilstm.epoch6.ckpt) and no retraining is needed

## Inputs

- Multi-read FAST5 Nanopore files (converted to single-read with ont_fast5_api multi_to_single_fast5)
- Genome reference FASTA (e.g. GCF_000001735.4_TAIR10.1_genomic.fna for A. thaliana)
- Feature table of 13-mer sequence plus 13x16 signal features per targeted cytosine (e.g. fast5s.C.features.tsv from deepsignal_plant extract)
- High-confidence methylated/unmethylated labels from bisulfite sequencing (>=5 reads and >=0.9 methylation frequency; >=5 reads and zero methylation frequency)

## Outputs

- Re-squiggled FAST5 reads with signals mapped to reference bases
- Per-cytosine feature table (13-mer sequence + 13x16 normalized signal features)
- Balanced, denoised positive/negative training sample file usable by deepsignal_plant train

## How to apply

First basecall with Guppy (>=3.6.1) and re-squiggle raw signals to the genome reference with Tombo 1.5.1, normalizing each read's raw signal by median shift and MAD scale. Extract per-cytosine samples containing the 13-mer sequence and 13x16 signal features (deepsignal_plant extract), then balance positive and negative samples per k-mer motif and cap read coverage per k-mer (Algorithm 1), subsampling at most 20 million samples (half positive, half negative) per motif. Run the iterative denoising loop (Algorithm 2, defaults I=10, R=3, E=3): per iteration, split the training set into two folds, train the BRNN-with-LSTM model (Adam optimizer, cross-entropy loss) on each fold and cross-predict the held-out fold for R=3 rounds, then remove positive samples whose mean predicted methylation probability mean(score(s)) < 0.5. Re-balance and re-cap samples after each removal round. Terminate when retention reaches at least 99% of samples or after 10 iterations, and write the retained balanced positive/negative samples for deepsignal_plant train.

## Related tools

- **DeepSignal-plant** (Extracts sequence/signal features per cytosine, performs balancing and iterative denoising, and trains the BiLSTM methylation model) — https://github.com/PengNi/deepsignal-plant
- **Tombo** (Re-squiggles raw signals of reads to contiguous bases in the genome reference before feature extraction (version 1.5.1)) — https://github.com/nanoporetech/tombo
- **Guppy** (Basecalls all Nanopore reads (version >=3.6.1) prior to re-squiggling)
- **PyTorch** (Implements the BRNN-with-LSTM model and Adam-optimized training used in the denoising cross-prediction loop)
- **ont_fast5_api** (Converts multi-read FAST5 to single-read FAST5 (multi_to_single_fast5) so Tombo and deepsignal_plant can read them) — https://github.com/nanoporetech/ont_fast5_api

## Examples

```
tombo resquiggle fast5s/ GCF_000001735.4_TAIR10.1_genomic.fna
```

## Evaluation signals

- The denoising loop terminates with >=99% sample retention or after the maximum I=10 iterations with R=3, E=3 defaults
- Only positive (methylated) samples with mean predicted methylation probability mean(score(s)) < 0.5 are removed between iterations
- Positive and negative sample counts are balanced per k-mer motif and read coverage is re-capped per k-mer after every removal round
- Each feature sample contains a 13-mer sequence and 13x16 signal features for the targeted cytosine after median-shift/MAD-scale normalization
- All FAST5s resquiggle successfully; if every fast5 fails, verify the ont-vbz-hdf-plugin is installed and HDF5_PLUGIN_PATH is set

## Limitations

- Training gold standard is bisulfite sequencing, so the trained model's performance is bounded by bisulfite quality; CHG/CHH high-confidence methylated labels for A. thaliana use a union of three replicates (lower stringency, potential label noise)
- Tombo is deprecated and unsupported by Oxford Nanopore, which recommends Remora instead; it requires strict h5py<3 / numpy<1.23 pins in the R9 environment
- VBZ-compressed FAST5 files require the ont-vbz-hdf-plugin and a correct HDF5_PLUGIN_PATH, otherwise all fast5s fail in tombo resquiggle and/or deepsignal_plant call_mods
- PyTorch version must be constrained (>=1.2.0, <=1.11.0); an inappropriate auto-installed version must be re-installed for the OS

## Evidence

- [intro] we develop a sample selection strategy to balance and denoise training samples, which can significantly improve the performances of the trained models, especially for CHH and CHG: "Therefore, we develop a sample selection strategy to balance and denoise training samples, which can significantly improve the performances of the trained models, especially for CHH and CHG"
- [discussion] In the denoising algorithm, we set I = 10, R = 3, E = 3 as default: "In the denoising algorithm, we set I = 10, R = 3, E = 3 as default"
- [discussion] we balance the types of k-mer in positive and negative samples, as well as the number of reads covered for each k-mer: "we balance the types of k-mer in positive and negative samples, as well as the number of reads covered for each k-mer"
- [discussion] raw signals of each read are normalized by using median shift and median absolute deviation (MAD) scale: "raw signals of each read are normalized by using median shift and median absolute deviation (MAD) scale"
- [readme] Guppy (version>=3.6.1) is also required, which can be downloaded from [Nanopore Community (login required)]: "Guppy (version>=3.6.1) is also required, which can be downloaded from [Nanopore Community (login required)]"
