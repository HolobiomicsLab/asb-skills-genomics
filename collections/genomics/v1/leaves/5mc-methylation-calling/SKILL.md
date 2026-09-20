---
name: 5mc-methylation-calling
description: Use when raw Nanopore FAST5 reads from plant genomes (A. thaliana, O.
  sativa, B. nigra) must be converted into per-read 5mC probabilities and per-site
  methylation frequencies across CG/CHG/CHH contexts.
when_to_use_negative:
- Input is short-read bisulfite sequencing data (this skill consumes native Nanopore
  signal; bisulfite is used here only as the gold-standard reference for training/evaluation)
- Non-plant 5mC or other modifications (e.g. 6mA, 5hmC) — the pre-trained arabnrice2-1
  model was trained on A. thaliana/O. sativa 5mC only
- R10.4.1/POD5-era data handled with modern tools — Tombo (R9 path) is deprecated
  by ONT (see Remora); the R10 route uses Dorado basecalling instead of Guppy+Tombo
edam_topics:
- http://edamontology.org/topic_0160
- http://edamontology.org/topic_3474
- http://edamontology.org/topic_0157
tools:
- name: DeepSignal-plant
  role: Core BiLSTM model that outputs per-read 5mC probabilities (call_mods) and
    per-site methylation frequencies (call_freq) for all three contexts
  repo: https://github.com/PengNi/deepsignal-plant
- name: Tombo
  role: Re-squiggles raw FAST5 signals onto reference genome bases (RawGenomeCorrected_000)
    required before call_mods; version 1.5.1
  repo: https://github.com/nanoporetech/tombo
- name: Guppy
  role: Basecalls raw Nanopore reads to FASTQ before tombo annotation; version >=3.6.1
- name: ont_fast5_api
  role: Converts multi-read FAST5 to single-read FAST5 required by the pipeline
  repo: https://github.com/nanoporetech/ont_fast5_api
- name: vbz_compression
  role: HDF5 plugin to decompress VBZ-compressed FAST5 signal data when resquiggle/call_mods
    fail on all files
  repo: https://github.com/nanoporetech/vbz_compression
provenance:
  source_task_ids:
  - task_006
  - task_007
  source_papers:
  - doi: 10.1038/s41467-021-26278-9
    title: Genome-wide detection of cytosine methylations in plant from Nanopore data
      using deep learning
  source_content_hash: sha256:b74faac2f8d220213a48bd4f0abba2708c170dad5ab3040c16eb0d948862f4b7
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/PengNi/deepsignal-plant
  tools:
  - DeepSignal-plant
  - Guppy
  - Tombo
  - ont_fast5_api
  - vbz_compression
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0160
  - http://edamontology.org/topic_3474
  - http://edamontology.org/topic_0157
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# 5mc-methylation-calling

## Summary

Detects genome-wide 5mC cytosine methylation in all three sequence contexts (CG/CHG/CHH) from plant Oxford Nanopore reads using DeepSignal-plant, a BiLSTM deep-learning model trained on A. thaliana/O. sativa R9.4 data. It replaces CpG-only or statistics-based methods (Tombo, nanopolish, Megalodon) that miss non-CpG methylation abundant in plant genomes.

## When to use

Use when raw Nanopore FAST5 reads from plant genomes (A. thaliana, O. sativa, B. nigra) must be converted into per-read 5mC probabilities and per-site methylation frequencies across CG/CHG/CHH contexts.

## When NOT to use

- Input is short-read bisulfite sequencing data (this skill consumes native Nanopore signal; bisulfite is used here only as the gold-standard reference for training/evaluation)
- Non-plant 5mC or other modifications (e.g. 6mA, 5hmC) — the pre-trained arabnrice2-1 model was trained on A. thaliana/O. sativa 5mC only
- R10.4.1/POD5-era data handled with modern tools — Tombo (R9 path) is deprecated by ONT (see Remora); the R10 route uses Dorado basecalling instead of Guppy+Tombo

## Inputs

- Raw Oxford Nanopore FAST5 files (single-read format; convert multi-read FAST5 with ont_fast5_api multi_to_single_fast5)
- Guppy FASTQ basecalls (Guppy >=3.6.1, dna_r9.4.1_450bps_hac_prom.cfg)
- Reference genome FASTA (e.g. GCF_000001735.4_TAIR10.1_genomic.fna for A. thaliana, IRGSP-1.0/GCA_001433935.1 for O. sativa)
- Pre-trained DeepSignal-plant checkpoint (model.dp2.CNN.arabnrice2-1_120m_R9.4plus_tem.bn13_sn16.both_bilstm.epoch6.ckpt)

## Outputs

- Per-read 5mC probability file from deepsignal_plant call_mods (features TSV)
- Per-site methylation frequency file from deepsignal_plant call_freq (optionally bedMethyl via --bed --sort)
- Context-split CG/CHG/CHH frequency files from split_freq_file_by_5mC_motif.py
- Summary tables of detected 5mC sites per context/species (e.g. excess-vs-bisulfite counts)

## How to apply

First basecall raw FAST5 with Guppy (>=3.6.1, dna_r9.4.1_450bps_hac_prom.cfg) and, if files are in multi-read format, convert with ont_fast5_api's multi_to_single_fast5. Annotate reads with the FASTQ basecalls (tombo preprocess annotate_raw_with_fastqs) and re-squiggle signals against the reference genome (e.g. TAIR10, IRGSP-1.0) with tombo 1.5.1, writing to the RawGenomeCorrected_000 group with Basecall_1D_000. Run deepsignal_plant call_mods with the pre-trained checkpoint model.dp2.CNN.arabnrice2-1_120m_R9.4plus_tem.bn13_sn16.both_bilstm.epoch6.ckpt and --motifs C to obtain per-read 5mC probabilities for all three contexts at once (one model handles CG/CHG/CHH); the model uses median shift and MAD scale signal normalization and k=13, m=16 sequence/signal features. Aggregate per-read calls to per-site methylation frequencies with deepsignal_plant call_freq (optionally --bed --sort for bedMethyl), then split by context with split_freq_file_by_5mC_motif.py. If all FAST5s fail in tombo resquiggle or call_mods, resolve VBZ compression by installing the ont-vbz-hdf-plugin and setting HDF5_PLUGIN_PATH. For comparison against bisulfite, restrict evaluation to cytosines with >=5 mapped Nanopore reads and >=10 mapped bisulfite reads, and report per-context Pearson correlation, RMSE, and read-level accuracy.

## Related tools

- **DeepSignal-plant** (Core BiLSTM model that outputs per-read 5mC probabilities (call_mods) and per-site methylation frequencies (call_freq) for all three contexts) — https://github.com/PengNi/deepsignal-plant
- **Tombo** (Re-squiggles raw FAST5 signals onto reference genome bases (RawGenomeCorrected_000) required before call_mods; version 1.5.1) — https://github.com/nanoporetech/tombo
- **Guppy** (Basecalls raw Nanopore reads to FASTQ before tombo annotation; version >=3.6.1)
- **ont_fast5_api** (Converts multi-read FAST5 to single-read FAST5 required by the pipeline) — https://github.com/nanoporetech/ont_fast5_api
- **vbz_compression** (HDF5 plugin to decompress VBZ-compressed FAST5 signal data when resquiggle/call_mods fail on all files) — https://github.com/nanoporetech/vbz_compression

## Examples

```
tombo resquiggle fast5s/ GCF_000001735.4_TAIR10.1_genomic.fna --corrected-group RawGenomeCorrected_000 --basecall-group Basecall_1D_000 --processes 10
```

## Evaluation signals

- Per-context Pearson correlation with bisulfite methylation levels on cytosines with >=5 Nanopore reads and >=10 bisulfite reads
- Reproduced context-wise metrics: CHG RMSE 0.1355 and CG read-level accuracy 0.9257 for B. nigra within a small tolerance
- Excess detected 5mC sites over bisulfite at 100x coverage: ~1.1% in A. thaliana and ~5.3% in O. sativa (Supplementary Table 1)
- Frequency aggregation sanity: per-site frequency equals methylated reads / total mapped reads at each cytosine
- Corrected-group invariants: resquiggle output stored in RawGenomeCorrected_000 with basecalls in Basecall_1D_000

## Limitations

- All FAST5s may fail in tombo resquiggle and/or deepsignal_plant call_mods due to VBZ compression; must install ont-vbz-hdf-plugin and export HDF5_PLUGIN_PATH
- Tombo is unsupported/deprecated by Oxford Nanopore (superseded by Remora), constraining the R9.4/Guppy/Tombo pipeline
- Training gold standard is bisulfite sequencing, so model performance is bounded by bisulfite quality; CHG/CHH high-confidence methylated labels for A. thaliana use a union of three replicates (potential label noise)
- Pre-trained model is specific to R9.4 1D reads of A. thaliana/O. sativa; comparisons with Megalodon require re-training Megalodon on the same combined dataset to be fair

## Evidence

- [readme] BiLSTM-based 5mC detection built on Python3/PyTorch: "deepsignal-plant applies BiLSTM to detect methylation from Nanopore reads. It is built on **Python3** and **PyTorch**."
- [readme] Guppy >=3.6.1 is required for basecalling: "Guppy (version>=3.6.1) is also required, which can be downloaded from [Nanopore Community (login required)]"
- [readme] VBZ compression failure mode and fix: "Please try adding ont-vbz-hdf-plugin to your environment as follows when all fast5s failed in `tombo resquiggle` and/or `deepsignal_plant call_mods`."
- [discussion] Per-site methylation frequency computation: "by counting the number of reads where the site is called methylated and the total number of reads mapped to the site, DeepSignal-plant calculates a methylation frequency"
- [discussion] Signal normalization and feature parameters: "raw signals of each read are normalized by using median shift and median absolute deviation (MAD) scale"
- [discussion] High-confidence methylated site threshold used for training/evaluation labels: "cytosines covered with at least five reads and had at least 0.9 methylation frequency as high-confidence methylated sites"
- [methods] Tombo resquiggle requirement with version: "be re-squiggled by [tombo (version 1.5.1)](https://github.com/nanoporetech/tombo)"
