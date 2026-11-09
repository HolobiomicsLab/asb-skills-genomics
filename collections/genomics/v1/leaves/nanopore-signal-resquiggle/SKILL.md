---
name: nanopore-signal-resquiggle
description: Use when raw (uncorrected) Nanopore FAST5 reads must have their electrical
  signals aligned to genome-reference bases before signal-based methylation calling,
  e.g. prior to deepsignal_plant call_mods.
when_to_use_negative:
- Input is R10.4.1 POD5/BAM data basecalled with Dorado — the deepsignal-plant R10
  environment handles POD5/BAM I/O and does not use Tombo resquiggle
- FAST5 files are already genome-corrected (RawGenomeCorrected group present) — re-running
  resquiggle on corrected files is unnecessary and can corrupt event mappings
- The goal is basecalling-alphabet modified-base calling (e.g. Megalodon or Dorado
  remora models) that does not depend on Tombo signal alignment
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_0102
tools:
- name: Tombo
  role: Performs the resquiggle step, mapping raw signals of reads to contiguous genome-reference
    bases (version 1.5.1)
  repo: https://github.com/nanoporetech/tombo
- name: ont_fast5_api
  role: Converts multi-read FAST5 files to single-read FAST5 format required before
    resquiggle
  repo: https://github.com/nanoporetech/ont_fast5_api
- name: vbz_compression
  role: Provides the ont-vbz-hdf-plugin needed to read VBZ-compressed FAST5 files;
    set HDF5_PLUGIN_PATH when all fast5s fail in resquiggle
  repo: https://github.com/nanoporetech/vbz_compression
- name: DeepSignal-plant
  role: 'Downstream consumer: call_mods and call_freq run on the resquiggled FAST5s
    to produce per-read 5mC probabilities and per-site methylation frequencies'
  repo: https://github.com/PengNi/deepsignal-plant
- name: Guppy
  role: Basecalls the Nanopore reads (version >=3.6.1) before resquiggling and methylation
    calling
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-021-26278-9
    title: Genome-wide detection of cytosine methylations in plant from Nanopore data
      using deep learning
  source_content_hash: sha256:a6cf75cbbbc7a52d714673665acd9b8d4381d5057b8ba0811e2965f3a72cd77b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/nanoporetech/tombo
  tools:
  - DeepSignal-plant
  - Guppy
  - Oxford Nanopore Technologies (ONT)
  - Tombo
  - ont_fast5_api
  - vbz_compression
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.45454545454545453
n_claims: 11
---

# nanopore-signal-resquiggle

## Summary

Re-squiggle raw Nanopore FAST5 signals against a reference genome with Tombo 1.5.1 so that raw current values are mapped to contiguous genome bases, enabling signal-feature extraction for DeepSignal-plant 5mC detection.

## When to use

Use when raw (uncorrected) Nanopore FAST5 reads must have their electrical signals aligned to genome-reference bases before signal-based methylation calling, e.g. prior to deepsignal_plant call_mods.

## When NOT to use

- Input is R10.4.1 POD5/BAM data basecalled with Dorado — the deepsignal-plant R10 environment handles POD5/BAM I/O and does not use Tombo resquiggle
- FAST5 files are already genome-corrected (RawGenomeCorrected group present) — re-running resquiggle on corrected files is unnecessary and can corrupt event mappings
- The goal is basecalling-alphabet modified-base calling (e.g. Megalodon or Dorado remora models) that does not depend on Tombo signal alignment

## Inputs

- Raw (uncorrected) Oxford Nanopore FAST5 reads, ideally as single-read FAST5 files (R9.4 protocol)
- Reference genome FASTA matched to the reads (e.g. GCF_000001735.4_TAIR10.1_genomic.fna for A. thaliana)

## Outputs

- Re-squiggled FAST5 files with corrected signal events written to the RawGenomeCorrected_000 group, with per-read shift/scale parameters for median-shift and MAD-scale normalization
- Resquiggle success/failure report per FAST5 file

## How to apply

Convert multi-read FAST5 files to single-read format with the _multi_to_single_fast5_ command from ont_fast5_api, since resquiggle requires one read per file. Run tombo resquiggle (version 1.5.1) with the appropriate genome FASTA (e.g. GCF_000001735.4_TAIR10.1_genomic.fna for A. thaliana), writing corrected signal events to the RawGenomeCorrected_000 group. If all FAST5s fail during resquiggle (or subsequent call_mods), install the ONT VBZ HDF5 plugin and set HDF5_PLUGIN_PATH, as VBZ-compressed FAST5s cannot otherwise be read. Tombo also reports the shift/scale used for median-shift and MAD-scale normalization, which DeepSignal-plant uses downstream for feature extraction (k=13, m=16) before the BiLSTM model computes per-read 5mC probabilities.

## Related tools

- **Tombo** (Performs the resquiggle step, mapping raw signals of reads to contiguous genome-reference bases (version 1.5.1)) — https://github.com/nanoporetech/tombo
- **ont_fast5_api** (Converts multi-read FAST5 files to single-read FAST5 format required before resquiggle) — https://github.com/nanoporetech/ont_fast5_api
- **vbz_compression** (Provides the ont-vbz-hdf-plugin needed to read VBZ-compressed FAST5 files; set HDF5_PLUGIN_PATH when all fast5s fail in resquiggle) — https://github.com/nanoporetech/vbz_compression
- **DeepSignal-plant** (Downstream consumer: call_mods and call_freq run on the resquiggled FAST5s to produce per-read 5mC probabilities and per-site methylation frequencies) — https://github.com/PengNi/deepsignal-plant
- **Guppy** (Basecalls the Nanopore reads (version >=3.6.1) before resquiggling and methylation calling)

## Examples

```
tombo resquiggle fast5s/ GCF_000001735.4_TAIR10.1_genomic.fna
```

## Evaluation signals

- Resquiggle completes without the 'all fast5s failed' VBZ error; if it occurs, setting HDF5_PLUGIN_PATH to the extracted ont-vbz-hdf-plugin directory resolves it
- Corrected FAST5 files contain a RawGenomeCorrected_000 group with genome-aligned events and per-read normalization (shift/scale) parameters
- Downstream deepsignal_plant call_mods yields per-read 5mC probabilities without signal-extraction errors on the resquiggled files
- Aggregated per-site methylation frequencies from call_freq reach the published correlation with bisulfite sequencing (e.g. recomputed CG read-level accuracy ~0.9257 for B. nigra within tolerance)

## Limitations

- Tombo is deprecated: the nanoporetech/tombo repository is unsupported and the project recommends Remora instead, so version pinning to 1.5.1 is fragile
- VBZ-compressed FAST5 files fail silently across all reads unless the ONT VBZ HDF5 plugin is installed and HDF5_PLUGIN_PATH is set
- The R9 environment requires strict h5py<3 and numpy<1.23 pins for Tombo compatibility, limiting dependency flexibility

## Evidence

- [methods] tombo resquiggle fast5s/ GCF_000001735.4_TAIR10.1_genomic.fna: "tombo resquiggle fast5s/ GCF_000001735.4_TAIR10.1_genomic.fna"
- [discussion] Tombo maps raw signals to genome reference: "We use Tombo21 (version 1.5.1) to map raw signals of reads to contiguous bases in the genome reference"
- [discussion] Median shift and MAD scale normalization: "raw signals of each read are normalized by using median shift and median absolute deviation (MAD) scale"
- [readme] VBZ plugin fix for failed resquiggle: "Please try adding ont-vbz-hdf-plugin to your environment as follows when all fast5s failed in `tombo resquiggle` and/or `deepsignal_plant call_mods`."
- [readme] Tombo deprecated upstream: [evidence span withheld — anchor status: external_doc]
- [readme] ont_fast5_api multi-to-single conversion: [evidence span withheld — anchor status: external_doc]
