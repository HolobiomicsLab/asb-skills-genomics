---
name: kmer-sequence-feature-construction
description: Use when you have re-squiggled single-read FAST5 files (Tombo, corrected_group
  RawGenomeCorrected_000) and need per-site 13-mer sequence+signal features to train
  or run DeepSignal-plant 5mC detection.
when_to_use_negative:
- The features TSV already exists and the goal is methylation calling — proceed directly
  to `deepsignal_plant call_mods` instead of re-extracting.
- Data are from the R10.4.1 POD5/BAM + Dorado protocol where FAST5/Tombo re-squiggle
  is not the input path.
- Input is bisulfite sequencing data, which has no Nanopore signal events to build
  base-level signal features from.
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0102
tools:
- name: DeepSignal-plant
  role: Provides the `extract` command that builds the 13-mer sequence and 13x16 signal
    features per targeted cytosine into fast5s.C.features.tsv.
  repo: https://github.com/PengNi/deepsignal-plant
- name: Tombo
  role: Re-squiggles raw signals to contiguous genome reference bases, producing the
    corrected signal events (corrected_group RawGenomeCorrected_000) from which per-base
    statistics are computed.
  repo: https://github.com/nanoporetech/tombo
- name: ont_fast5_api
  role: Converts multi-read FAST5 files to single-read FAST5 with multi_to_single_fast5
    before re-squiggling and extraction.
  repo: https://github.com/nanoporetech/ont_fast5_api
- name: Guppy
  role: Basecalls the Nanopore reads (version >= 3.6.1) to provide the base sequences
    required before re-squiggling.
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-021-26278-9
    title: Genome-wide detection of cytosine methylations in plant from Nanopore data
      using deep learning
  source_content_hash: sha256:b2c4929ea92e1201d27db12e9416f856b0e594202ea95ff02d2515608fbdb32d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/PengNi/deepsignal-plant
  tools:
  - DeepSignal-plant
  - Guppy
  - Tombo
  - ont_fast5_api
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 12
---

# kmer-sequence-feature-construction

## Summary

Constructs the per-cytosine feature matrix for DeepSignal-plant: sequence features from the centered 13-mer (base identity, per-base signal mean, standard deviation, and signal length) plus 13x16 raw signal features extracted from Tombo re-squiggled reads, written to a tab-delimited features TSV that feeds BiLSTM methylation calling.

## When to use

Use when you have re-squiggled single-read FAST5 files (Tombo, corrected_group RawGenomeCorrected_000) and need per-site 13-mer sequence+signal features to train or run DeepSignal-plant 5mC detection.

## When NOT to use

- The features TSV already exists and the goal is methylation calling — proceed directly to `deepsignal_plant call_mods` instead of re-extracting.
- Data are from the R10.4.1 POD5/BAM + Dorado protocol where FAST5/Tombo re-squiggle is not the input path.
- Input is bisulfite sequencing data, which has no Nanopore signal events to build base-level signal features from.

## Inputs

- Single-read FAST5 files re-squiggled by Tombo (version 1.5.1) against the genome reference (e.g. GCF_000001735.4_TAIR10.1_genomic.fna for A. thaliana)
- Genome reference FASTA used for Tombo resquiggle
- Motif specification (e.g. C for cytosines in all CpG/CHG/CHH contexts)

## Outputs

- Tab-delimited features TSV (e.g. fast5s.C.features.tsv) with columns chrom, pos, strand, readname, read_strand, k_mer, base_means, base_stds, base_signal_lens, raw_signals, methy_label
- Per-base signal statistics for the 13-mer: means, standard deviations, and signal lengths
- 13x16 raw signal feature blocks joined with ';' in the raw_signals column

## How to apply

Run `deepsignal_plant extract` on the single-read FAST5 directory (converted with multi_to_single_fast5 from ont_fast5_api) with --corrected_group RawGenomeCorrected_000, --motifs C, and --nproc 30, writing fast5s.C.features.tsv. For each targeted cytosine in each read, locate the 13-mer sequence context (k=13 by default) centered on the target base; from the Tombo-aligned signal events compute four per-base features: base identity, signal mean, signal standard deviation, and number of signal values per base. Signals must first be normalized by median shift and median absolute deviation (MAD) scale. Additionally collect the 13x16 raw signal features, joining per-base blocks with ';' in the raw_signals column. Record read metadata (chrom, pos, strand, readname, read_strand) and the 0/1 methy_label column used for training.

## Related tools

- **DeepSignal-plant** (Provides the `extract` command that builds the 13-mer sequence and 13x16 signal features per targeted cytosine into fast5s.C.features.tsv.) — https://github.com/PengNi/deepsignal-plant
- **Tombo** (Re-squiggles raw signals to contiguous genome reference bases, producing the corrected signal events (corrected_group RawGenomeCorrected_000) from which per-base statistics are computed.) — https://github.com/nanoporetech/tombo
- **ont_fast5_api** (Converts multi-read FAST5 files to single-read FAST5 with multi_to_single_fast5 before re-squiggling and extraction.) — https://github.com/nanoporetech/ont_fast5_api
- **Guppy** (Basecalls the Nanopore reads (version >= 3.6.1) to provide the base sequences required before re-squiggling.)

## Examples

```
deepsignal_plant extract fast5s/ --corrected_group RawGenomeCorrected_000 --motifs C --nproc 30 --out_file_path fast5s.C.features.tsv
```

## Evaluation signals

- The output TSV is tab-delimited and contains all declared columns including chrom, pos, strand, readname, read_strand, k_mer, raw_signals, and methy_label.
- Every k_mer field is exactly a 13-mer centered on the targeted cytosine.
- Every raw_signals field contains a signal block for every base of the k-mer, with per-base blocks joined by ';' (13x16 signal features).
- Per-base features (means, standard deviations, signal lengths) are computed from median-shift / MAD-scaled signals, matching the article's normalization.
- methy_label values are binary 0/1 training labels consistent with the high-confidence site selection.

## Limitations

- Tombo (1.5.1) is deprecated and unsupported by Oxford Nanopore, replaced by Remora; it requires strict dependency pins (e.g. h5py<3, numpy<1.23).
- VBZ-compressed FAST5 files fail in `tombo resquiggle` and `deepsignal_plant call_mods` unless the ont-vbz-hdf-plugin is installed and HDF5_PLUGIN_PATH is set.
- Multi-read FAST5 inputs must first be converted to single-read format with multi_to_single_fast5 before extraction works.
- Feature extraction depends on the quality of the Tombo re-squiggle; misalignment of signal events corrupts the per-base signal statistics.

## Evidence

- [discussion] Tombo resquiggle basis: "We use Tombo21 (version 1.5.1) to map raw signals of reads to contiguous bases in the genome reference"
- [discussion] Signal normalization: "raw signals of each read are normalized by using median shift and median absolute deviation (MAD) scale"
- [readme] Re-squiggle prerequisite: "are required to basecall and re-squiggle the raw signals from nanopore reads before running deepsignal-plant"
- [readme] Tombo version pin: "[tombo (version 1.5.1)](https://github.com/nanoporetech/tombo) is required to be installed"
- [methods] 13-mer construction: [evidence span withheld — anchor status: not_found]
