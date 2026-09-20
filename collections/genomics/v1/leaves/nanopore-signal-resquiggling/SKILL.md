---
name: nanopore-signal-resquiggling
description: Use when raw nanopore fast5 signal must be aligned to a reference genome
  and reformatted to multi-fast5 before running methylation motif callers such as
  Snapper's sample-vs-WGA comparison.
when_to_use_negative:
- Input is already resquiggled multi-fast5 data produced by this pipeline — re-resquiggling
  is unnecessary
- Downstream analysis does not require raw signal alignment (e.g., basecall-level
  modification calling with newer tools such as Remora, which ONT now recommends over
  Tombo)
- The fast5 signal data is unavailable and only FASTQ basecalls are on hand
edam_operation: http://edamontology.org/operation_3218
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3168
tools:
- name: Tombo
  role: Resquiggles raw nanopore signal against the reference genome, the first step
    in any downstream Tombo/Snapper analysis
  repo: https://github.com/nanoporetech/tombo
- name: ont_fast5_api
  role: Converts resquiggled fast5 files between single-read and multi-read formats
    via single_to_multi_fast5
  repo: https://github.com/nanoporetech/ont_fast5_api
- name: snapper-ont
  role: Downstream consumer of the multi-fast5 sample and control directories for
    greedy motif enrichment
- name: Python
  role: Runtime for the python 3.7 conda environment in which tombo, ont-fast5-api,
    and snapper-ont are installed
provenance:
  source_task_ids:
  - task_001
  - task_004
  - task_003
  source_papers:
  - doi: 10.1093/bioinformatics/btad702
    title: 'Snapper: high-sensitive detection of methylation motifs based on Oxford
      Nanopore reads'
  source_content_hash: sha256:f01ef885c3c18d143f2b1758d7e2f4180c8b568331e18ab7e60df6b39b91fa59
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/nanoporetech/tombo
  tools:
  - Python
  - Snapper
  - Tombo
  - ont_fast5_api
  - pip
  - snapper-ont
  edam_operation: http://edamontology.org/operation_3218
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 14
---

# nanopore-signal-resquiggling

## Summary

Preprocessing skill for Oxford Nanopore methylation motif detection: raw fast5 files are resquiggled against a reference genome with Tombo (aligning raw signal to basecalls), then converted to the multi-read fast5 format with ont_fast5_api so Snapper can compare native sample and WGA control signal.

## When to use

Use when raw nanopore fast5 signal must be aligned to a reference genome and reformatted to multi-fast5 before running methylation motif callers such as Snapper's sample-vs-WGA comparison.

## When NOT to use

- Input is already resquiggled multi-fast5 data produced by this pipeline — re-resquiggling is unnecessary
- Downstream analysis does not require raw signal alignment (e.g., basecall-level modification calling with newer tools such as Remora, which ONT now recommends over Tombo)
- The fast5 signal data is unavailable and only FASTQ basecalls are on hand

## Inputs

- Raw single- or multi-read Oxford Nanopore fast5 files (native/native-modified sample)
- Raw fast5 files from a whole-genome amplification (WGA) control run
- Reference genome in FASTA format (e.g., J99.fasta)

## Outputs

- Resquiggled fast5 files with raw-signal-to-reference alignments indexed (Tombo)
- Multi-read fast5 directories for sample and control (e.g., J99_multi, J99_wga_multi) ready for snapper input
- Optionally, Tombo per-base modified-base statistics from model_sample_compare

## How to apply

First resquiggle the raw fast5 files against the reference genome (e.g., J99.fasta from the Zenodo deposit 10.5281/zenodo.10117651) using Tombo, which creates an index and stores the raw signal alignments needed for downstream per-base statistics. Then convert the resquiggled files to multi-fast5 format with ont_fast5_api's single_to_multi_fast5 (or verify existing files are multi-read), for both the native sample directory and the WGA control directory. Set up a python 3.7 conda environment (later versions may be incompatible because of inner biopython dependencies) and install ont-fast5-api and ont-tombo from bioconda. Run this preprocessing before invoking snapper with -sample_fast5dir and -control_fast5dir pointing at the multi-fast5 directories; Tombo's model_sample_compare can also be run afterward to obtain per-base modified-base statistics.

## Related tools

- **Tombo** (Resquiggles raw nanopore signal against the reference genome, the first step in any downstream Tombo/Snapper analysis) — https://github.com/nanoporetech/tombo
- **ont_fast5_api** (Converts resquiggled fast5 files between single-read and multi-read formats via single_to_multi_fast5) — https://github.com/nanoporetech/ont_fast5_api
- **snapper-ont** (Downstream consumer of the multi-fast5 sample and control directories for greedy motif enrichment)
- **Python** (Runtime for the python 3.7 conda environment in which tombo, ont-fast5-api, and snapper-ont are installed)

## Examples

```
single_to_multi_fast5 --input_path /data/reads --save_path /data/J99_multi --filename_base batch --batch_size 100 --recursive
```

## Evaluation signals

- Resquiggling completes without errors and Tombo's signal-to-reference index is created for all reads
- Both the native sample and WGA control exist as multi-read fast5 directories (e.g., J99_multi and J99_wga_multi) with batch_*.fast5 files produced by single_to_multi_fast5
- Downstream snapper run accepts the -sample_fast5dir and -control_fast5dir inputs and emits passed_motifs_ and final_motifs FASTA files per strand/contig
- The python environment is python 3.7 with ont-tombo and ont-fast5-api installed from bioconda, since later Python versions may break biopython dependencies

## Limitations

- Tombo is deprecated and unsupported by Oxford Nanopore, which recommends Remora as the replacement resource
- Snapper requires python 3.7; later versions might be incompatible because of inner biopython dependencies
- Snapper's comparison requires matched native and WGA (amplified) control read sets; without a WGA control the sample-vs-control statistics cannot be computed

## Evidence

- [methods] Tombo resquiggle prerequisite: "Firstly, fast5 files should be resquiggled using [Tombo](https://github.com/nanoporetech/tombo) software."
- [methods] multi-fast5 conversion prerequisite: "After resquiggling, fast5 files should be converted to the multi-fast5 format using [ont_fast5_api](https://github.com/nanoporetech/ont_fast5_api)."
- [readme] installation via conda/pip: "(snapper) $ conda install -c bioconda ont-fast5-api ont-tombo
(snapper) $ pip install snapper-ont"
- [readme] python 3.7 version constraint: "python 3.7 (later versions might be incompatible because of inner biopython dependencies)"
- [readme] Tombo resquiggle purpose: [evidence span withheld — anchor status: external_doc]
