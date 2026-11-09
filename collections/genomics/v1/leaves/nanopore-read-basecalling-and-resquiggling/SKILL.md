---
name: nanopore-read-basecalling-and-resquiggling
description: Use when starting from raw single-read FAST5 Nanopore files (e.g. R9.4
  plant reads) that need basecalls and raw-signal-to-reference alignment before deep-learning
  methylation detection.
when_to_use_negative:
- Input is already basecalled, re-squiggled FAST5 with a populated RawGenomeCorrected_000
  group (skip straight to deepsignal_plant call_mods)
- Data is R10.4.1 POD5/BAM from Dorado rather than R9.4 FAST5 + Guppy + Tombo, where
  the Tombo-based re-squiggle path does not apply
- Goal is methylation calling with statistical methods (e.g. Tombo detect_mods) rather
  than preparing inputs for DeepSignal-plant
edam_operation: http://edamontology.org/operation_3182
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3168
tools:
- name: Guppy
  role: Basecalls all Nanopore reads (v3.6.1+) to produce FASTQs and Basecall_1D_000
    groups in the FAST5s
- name: Tombo
  role: Annotates raw FAST5s with FASTQs and maps raw signals to contiguous genome
    reference bases (v1.5.1 re-squiggle)
  repo: https://github.com/nanoporetech/tombo
- name: ont_fast5_api
  role: Converts multi-read FAST5 to single-read FAST5 via multi_to_single_fast5 before
    processing
  repo: https://github.com/nanoporetech/ont_fast5_api
- name: vbz_compression
  role: Provides the ONT VBZ HDF5 plugin needed to read VBZ-compressed FAST5 signal
    data
  repo: https://github.com/nanoporetech/vbz_compression
- name: DeepSignal-plant
  role: Downstream consumer of re-squiggled FAST5s; call_mods reads the RawGenomeCorrected_000
    group to extract signal features
  repo: https://github.com/PengNi/deepsignal-plant
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-021-26278-9
    title: Genome-wide detection of cytosine methylations in plant from Nanopore data
      using deep learning
  source_content_hash: sha256:032df71ec9e609e5a8aa89b137cfac4cd340109b5dcb55e26ff33eabcc48e24c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/nanoporetech/tombo
  tools:
  - DeepSignal-plant
  - Guppy
  - Tombo
  - ont_fast5_api
  - vbz_compression
  edam_operation: http://edamontology.org/operation_3182
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# nanopore-read-basecalling-and-resquiggling

## Summary

Preprocess raw Oxford Nanopore FAST5 reads for methylation calling by basecalling with Guppy and re-squiggling raw signals to the genome reference with Tombo, producing signal-to-base aligned reads ready for DeepSignal-plant per-read 5mC probability calls.

## When to use

Use when starting from raw single-read FAST5 Nanopore files (e.g. R9.4 plant reads) that need basecalls and raw-signal-to-reference alignment before deep-learning methylation detection.

## When NOT to use

- Input is already basecalled, re-squiggled FAST5 with a populated RawGenomeCorrected_000 group (skip straight to deepsignal_plant call_mods)
- Data is R10.4.1 POD5/BAM from Dorado rather than R9.4 FAST5 + Guppy + Tombo, where the Tombo-based re-squiggle path does not apply
- Goal is methylation calling with statistical methods (e.g. Tombo detect_mods) rather than preparing inputs for DeepSignal-plant

## Inputs

- Raw single-read FAST5 files (R9.4 Nanopore reads, possibly VBZ-compressed)
- Genome reference FASTA (e.g. GCF_000001735.4_TAIR10.1_genomic.fna or IRGSP-1.0)
- Guppy FASTQ basecalls (dna_r9.4.1_450bps_hac_prom configuration)

## Outputs

- Re-squiggled FAST5 files with corrected signal groups (RawGenomeCorrected_000) and Basecall_1D_000 groups
- Signal-to-base aligned reads ready for deepsignal_plant call_mods feature extraction

## How to apply

Basecall raw single-read FAST5 files with Guppy (v3.6.1+, e.g. dna_r9.4.1_450bps_hac_prom.cfg); convert multi-read to single-read FAST5 with ont_fast5_api if needed. Annotate raw reads with the FASTQ basecalls using tombo preprocess annotate_raw_with_fastqs, then run tombo resquiggle against the species reference (e.g. TAIR10 fna for A. thaliana, IRGSP-1.0 for O. sativa) with --corrected-group RawGenomeCorrected_000 and --basecall-group Basecall_1D_000. If resquiggle or call_mods fails on all FAST5s, install the ONT VBZ HDF5 plugin and set HDF5_PLUGIN_PATH to its plugin directory. The corrected-group output stores the signal-to-base mapping that DeepSignal-plant call_mods reads to extract normalized signal features (median shift and MAD scale) for 5mC prediction.

## Related tools

- **Guppy** (Basecalls all Nanopore reads (v3.6.1+) to produce FASTQs and Basecall_1D_000 groups in the FAST5s)
- **Tombo** (Annotates raw FAST5s with FASTQs and maps raw signals to contiguous genome reference bases (v1.5.1 re-squiggle)) — https://github.com/nanoporetech/tombo
- **ont_fast5_api** (Converts multi-read FAST5 to single-read FAST5 via multi_to_single_fast5 before processing) — https://github.com/nanoporetech/ont_fast5_api
- **vbz_compression** (Provides the ONT VBZ HDF5 plugin needed to read VBZ-compressed FAST5 signal data) — https://github.com/nanoporetech/vbz_compression
- **DeepSignal-plant** (Downstream consumer of re-squiggled FAST5s; call_mods reads the RawGenomeCorrected_000 group to extract signal features) — https://github.com/PengNi/deepsignal-plant

## Examples

```
tombo resquiggle fast5s/ GCF_000001735.4_TAIR10.1_genomic.fna --corrected-group RawGenomeCorrected_000 --basecall-group Basecall_1D_000 --processes 10
```

## Evaluation signals

- Re-squiggled FAST5s contain a populated RawGenomeCorrected_000 group with Basecall_1D_000 basecall groups
- tombo resquiggle completes without failing on all FAST5 files (all-failure indicates a missing VBZ HDF5 plugin)
- Reference matches the species: TAIR10 (GCF_000001735.4) for A. thaliana, IRGSP-1.0 (GCA_001433935.1) for O. sativa
- Guppy basecalls use a config consistent with the flowcell chemistry, e.g. dna_r9.4.1_450bps_hac_prom.cfg
- Downstream deepsignal_plant call_mods produces per-read 5mC probabilities covering the expected number of reads

## Limitations

- VBZ-compressed FAST5 files fail tombo resquiggle and call_mods unless the ont-vbz-hdf-plugin is installed and HDF5_PLUGIN_PATH is set
- Tombo (v1.5.1) is deprecated/unsupported by ONT, and requires strict h5py<3 / numpy<1.23 pins in the R9 environment
- Re-squiggling is compute-bound; the article's benchmark used --processes 10 and GPU-accelerated call_mods (--nproc 30, --nproc_gpu 6)

## Evidence

- [discussion] Guppy basecalling: "We use Guppy (version 3.6.1 + 249406c) for the basecalling of all the Nanopore reads"
- [discussion] Tombo re-squiggle: "We use Tombo21 (version 1.5.1) to map raw signals of reads to contiguous bases in the genome reference"
- [methods] resquiggle command: "tombo resquiggle fast5s/ GCF_000001735.4_TAIR10.1_genomic.fna"
- [readme] VBZ plugin issue: "Please try adding ont-vbz-hdf-plugin to your environment as follows when all fast5s failed in `tombo resquiggle` and/or `deepsignal_plant call_mods`."
- [methods] tombo installation: "be re-squiggled by [tombo (version 1.5.1)](https://github.com/nanoporetech/tombo)"
