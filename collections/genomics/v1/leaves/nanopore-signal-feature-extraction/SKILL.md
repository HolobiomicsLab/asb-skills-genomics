---
name: nanopore-signal-feature-extraction
description: Use when raw Nanopore FAST5/POD5 reads have been basecalled and re-squiggled
  against a genome reference and per-base sequence-context plus raw-signal features
  are needed to train or run a methylation-calling model on cytosine sites.
when_to_use_negative:
- Input is already a DeepSignal-plant features TSV (e.g. fast5s.C.features.tsv) —
  downstream steps like balancing or training should consume it directly
- Reads are raw FAST5 that have not been basecalled and re-squiggled; run Guppy basecalling
  and tombo resquiggle first
- The goal is methylation calling with statistics-based methods alone (e.g. Tombo's
  model-free testing) rather than building deep-learning features
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3557
- http://edamontology.org/topic_0622
tools:
- name: DeepSignal-plant
  role: Provides the `deepsignal_plant extract` command that computes 13-mer sequence
    features and 13×16 signal features per cytosine into a TSV
  repo: https://github.com/PengNi/deepsignal-plant
- name: Tombo
  role: Re-squiggles raw Nanopore signals to genome reference bases (v1.5.1, --corrected-group
    RawGenomeCorrected_000) so per-base signal statistics can be computed
  repo: https://github.com/nanoporetech/tombo
- name: Guppy
  role: Basecalls raw FAST5 reads (version >=3.6.1) prior to re-squiggling and feature
    extraction
- name: ont_fast5_api
  role: Converts multi-read FAST5 to single-read FAST5 (`multi_to_single_fast5`) as
    required before resquiggling and extraction
  repo: https://github.com/nanoporetech/ont_fast5_api
- name: vbz_compression
  role: HDF5 VBZ plugin needed to read VBZ-compressed FAST5 files; set HDF5_PLUGIN_PATH
    to avoid failures in tombo resquiggle and call_mods
  repo: https://github.com/nanoporetech/vbz_compression
merged_aliases:
- nanopore-raw-signal-feature-extraction
merged_alias_records:
- alias: nanopore-signal-feature-table-manipulation
  slug: nanopore-signal-feature-table-manipulation
  cosine_score: 0.8388
  method: embedding-cosine
  decision: needs_review
provenance:
  source_task_ids:
  - task_003
  - task_005
  - task_001
  source_papers:
  - doi: 10.1038/s41467-021-26278-9
    title: Genome-wide detection of cytosine methylations in plant from Nanopore data
      using deep learning
  source_content_hash: sha256:50e4daa71bab228880676ecf5cb634f413febf13892af639b779232e95e1fcba
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/PengNi/deepsignal-plant
  tools:
  - DeepSignal-plant
  - Guppy
  - PyTorch
  - Python
  - RedNano
  - Tombo
  - h5py
  - ont-tombo
  - ont_fast5_api
  - vbz_compression
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3557
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7692307692307693
n_claims: 13
---

# nanopore-signal-feature-extraction

## Summary

Extract per-cytosine sequence and raw-signal features from re-squiggled Nanopore reads: a 13-mer k-mer centered on the target base plus 13×16 raw signal features (4 per-base features × 13 bases), as used by DeepSignal-plant for 5mC detection in plants.

## When to use

Use when raw Nanopore FAST5/POD5 reads have been basecalled and re-squiggled against a genome reference and per-base sequence-context plus raw-signal features are needed to train or run a methylation-calling model on cytosine sites.

## When NOT to use

- Input is already a DeepSignal-plant features TSV (e.g. fast5s.C.features.tsv) — downstream steps like balancing or training should consume it directly
- Reads are raw FAST5 that have not been basecalled and re-squiggled; run Guppy basecalling and tombo resquiggle first
- The goal is methylation calling with statistics-based methods alone (e.g. Tombo's model-free testing) rather than building deep-learning features

## Inputs

- Re-squiggled single-read FAST5 directory (corrected group RawGenomeCorrected_000 from tombo resquiggle v1.5.1)
- Genome reference FASTA (e.g. GCF_000001735.4_TAIR10.1_genomic.fna)
- Basecalled reads from Guppy >=3.6.1
- Motif specification (e.g. --motifs C for cytosine 5mC detection)

## Outputs

- Tab-delimited feature TSV (e.g. fast5s.C.features.tsv) with 11 columns: chrom, pos, strand, pos_in_strand, readname, read_strand, k_mer, signal_means, signal_stds, signal_lens, raw_signals, methy_label
- 13-mer k_mer strings centered on each targeted cytosine
- 13×16 raw signal features per site (per-base means, stds, lengths, and ';'-joined raw signal blocks)

## How to apply

First basecall reads (e.g. Guppy >=3.6.1 with dna_r9.4.1_450bps_hac_prom.cfg) and re-squiggle raw signals to the reference (tombo resquiggle v1.5.1, --corrected-group RawGenomeCorrected_000); normalize signals with median shift and MAD scale. Then run `deepsignal_plant extract` with --motifs C and the corrected group to produce a features TSV. For each targeted cytosine, locate the 13-mer sequence context (k=13 by default) centered on the target base and compute four per-base features: base identity, signal mean, signal standard deviation, and number of signal values per base, yielding 13×16 raw signal features joined with ';' in the raw_signals column. The output is a tab-delimited 11-column TSV (chrom, pos, strand, pos_in_strand, readname, read_strand, k_mer, signal_means, signal_stds, signal_lens, raw_signals, methy_label). Validate that every k_mer field is a 13-mer and every raw_signals field contains signal blocks for all 13 bases.

## Related tools

- **DeepSignal-plant** (Provides the `deepsignal_plant extract` command that computes 13-mer sequence features and 13×16 signal features per cytosine into a TSV) — https://github.com/PengNi/deepsignal-plant
- **Tombo** (Re-squiggles raw Nanopore signals to genome reference bases (v1.5.1, --corrected-group RawGenomeCorrected_000) so per-base signal statistics can be computed) — https://github.com/nanoporetech/tombo
- **Guppy** (Basecalls raw FAST5 reads (version >=3.6.1) prior to re-squiggling and feature extraction)
- **ont_fast5_api** (Converts multi-read FAST5 to single-read FAST5 (`multi_to_single_fast5`) as required before resquiggling and extraction) — https://github.com/nanoporetech/ont_fast5_api
- **vbz_compression** (HDF5 VBZ plugin needed to read VBZ-compressed FAST5 files; set HDF5_PLUGIN_PATH to avoid failures in tombo resquiggle and call_mods) — https://github.com/nanoporetech/vbz_compression

## Examples

```
deepsignal_plant extract --input_path fast5s/ --ref GCF_000001735.4_TAIR10.1_genomic.fna --corrected_group RawGenomeCorrected_000 --motifs C --nproc 30 --output_path fast5s.C.features.tsv
```

## Evaluation signals

- Output TSV is tab-delimited with all 11 declared columns (chrom, pos, strand, pos_in_strand, readname, read_strand, k_mer, signal_means, signal_stds, signal_lens, raw_signals, methy_label)
- Each k_mer field is exactly a 13-mer centered on the targeted cytosine
- Each raw_signals field contains signal blocks for every base of the 13-mer (13 blocks joined with ';')
- Per-base signal means, standard deviations, and lengths are computed from Tombo-aligned signal events of the re-squiggled reads
- read_strand and methy_label columns are populated (0/1 methy_label for training rows)

## Limitations

- VBZ-compressed FAST5 files fail in `tombo resquiggle` and `deepsignal_plant call_mods` unless the ont-vbz-hdf-plugin is installed and HDF5_PLUGIN_PATH is set
- Tombo is deprecated and unsupported; ONT recommends Remora, and Tombo requires strict h5py<3 / numpy<1.23 pins in the R9 environment
- Signal normalization and feature extraction depend on a contiguous genome reference alignment; mis-mapped events corrupt per-base signal statistics
- Runtime and memory usage of the pipeline are only documented in supplementary materials

## Evidence

- [results] For each targeted 5mC site, DeepSignal-plant builds sequence features from the centered k-mer (k = 13 by default) consisting of four per-base features: the base identity and the mean, standard deviation, and number of signal values of each base.: [evidence span withheld — anchor status: not_found]
- [discussion] raw signals of each read are normalized by using median shift and median absolute deviation (MAD) scale: "raw signals of each read are normalized by using median shift and median absolute deviation (MAD) scale"
- [methods] tombo resquiggle fast5s/ GCF_000001735.4_TAIR10.1_genomic.fna: "tombo resquiggle fast5s/ GCF_000001735.4_TAIR10.1_genomic.fna"
- [readme] Please try adding ont-vbz-hdf-plugin to your environment as follows when all fast5s failed in `tombo resquiggle` and/or `deepsignal_plant call_mods`.: "Please try adding ont-vbz-hdf-plugin to your environment as follows when all fast5s failed in `tombo resquiggle` and/or `deepsignal_plant call_mods`"
- [methods] please use _multi_to_single_fast5_ command from the [ont_fast5_api package](https://github.com/nanoporetech/ont_fast5_api): "please use _multi_to_single_fast5_ command from the [ont_fast5_api package](https://github.com/nanoporetech/ont_fast5_api)"
