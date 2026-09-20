---
name: ont-fast5-signal-data-processing
description: Use when raw nanopore .fast5 files (single-read format) must be prepared
  as input for ONT signal-level methylation motif detection, e.g. before running Snapper
  on H.pylori J99 sample versus WGA-control reads.
when_to_use_negative:
- Input reads are already in multi-read fast5 format and resquiggled — reconverting
  or re-resquiggling is unnecessary and risks corrupting the signal index
- Analysis relies on modern basecalling-native POD5/BAM inputs with Remora, since
  Tombo is deprecated and unsupported by Oxford Nanopore
- Data is not Oxford Nanopore signal data (e.g. PacBio or Illumina reads), which lack
  .fast5 raw signal and cannot be resquiggled
edam_operation: http://edamontology.org/operation_3218
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: Tombo
  role: Resquiggles raw nanopore signal in .fast5 files against the reference genome
    before downstream motif analysis
  repo: https://github.com/nanoporetech/tombo
- name: ont_fast5_api
  role: Converts single-read .fast5 files to multi-read fast5 format required by Snapper
  repo: https://github.com/nanoporetech/ont_fast5_api
- name: Snapper
  role: Downstream consumer of the processed multi-fast5 sample and control directories
    for methylation motif detection
  repo: https://github.com/DNKonanov/Snapper
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioinformatics/btad702
    title: 'Snapper: high-sensitive detection of methylation motifs based on Oxford
      Nanopore reads'
  source_content_hash: sha256:59b1597f7a6fc44057d9a900bcf58a4ad13c5aa02815fdae88cb7333cbf91758
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/nanoporetech/tombo
  tools:
  - Nanodisco
  - Snapper
  - Tombo
  - ont_fast5_api
  edam_operation: http://edamontology.org/operation_3218
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# ont-fast5-signal-data-processing

## Summary

Preprocess Oxford Nanopore raw .fast5 signal data for methylation motif analysis by resquiggling reads against a reference genome with Tombo, then converting to the multi-read fast5 format with ont_fast5_api before running Snapper.

## When to use

Use when raw nanopore .fast5 files (single-read format) must be prepared as input for ONT signal-level methylation motif detection, e.g. before running Snapper on H.pylori J99 sample versus WGA-control reads.

## When NOT to use

- Input reads are already in multi-read fast5 format and resquiggled — reconverting or re-resquiggling is unnecessary and risks corrupting the signal index
- Analysis relies on modern basecalling-native POD5/BAM inputs with Remora, since Tombo is deprecated and unsupported by Oxford Nanopore
- Data is not Oxford Nanopore signal data (e.g. PacBio or Illumina reads), which lack .fast5 raw signal and cannot be resquiggled

## Inputs

- Raw single-read .fast5 nanopore files (e.g. H.pylori J99 sample and WGA-control reads)
- Reference genome in FASTA format (e.g. J99.fasta)

## Outputs

- Resquiggled .fast5 files with raw signal aligned to the reference
- Multi-read fast5 directory (batch_0.fast5, batch_1.fast5, ...) ready for Snapper

## How to apply

First resquiggle the raw .fast5 files with Tombo so that raw nanopore signal is aligned to the reference genome sequence (this creates the index and signal-to-sequence alignments needed for downstream k-mer statistics). Then convert the resquiggled files to multi-read fast5 format using ont_fast5_api's single_to_multi_fast5 script, since Snapper consumes a sample multi-fast5 directory and a control multi-fast5 directory. Install the environment with conda (python 3.7, ont-tombo, ont-fast5-api) and pip install snapper-ont, noting that python 3.7 is required because later versions may break inner biopython dependencies. Rationale: Tombo's resquiggle step is the mandatory first stage of any Tombo/Snapper-style signal analysis, and the multi-fast5 conversion matches Snapper's -sample_fast5dir/-control_fast5dir input contract.

## Related tools

- **Tombo** (Resquiggles raw nanopore signal in .fast5 files against the reference genome before downstream motif analysis) — https://github.com/nanoporetech/tombo
- **ont_fast5_api** (Converts single-read .fast5 files to multi-read fast5 format required by Snapper) — https://github.com/nanoporetech/ont_fast5_api
- **Snapper** (Downstream consumer of the processed multi-fast5 sample and control directories for methylation motif detection) — https://github.com/DNKonanov/Snapper

## Examples

```
single_to_multi_fast5 --input_path /data/reads --save_path /data/multi_reads --filename_base batch --batch_size 4000 --recursive
```

## Evaluation signals

- Tombo resquiggle completes and produces an index of raw-signal-to-reference alignments without erroring on the fast5 inputs
- single_to_multi_fast5 produces the expected batched multi_read files (batch_0.fast5, batch_1.fast5, ...) in the output folder
- Snapper accepts the converted directories via -sample_fast5dir and -control_fast5dir and runs to completion
- Environment resolves: python 3.7 with ont-tombo and ont-fast5-api installed from bioconda; pip install snapper-ont succeeds

## Limitations

- Python 3.7 must be used; later python versions might be incompatible because of inner biopython dependencies
- Tombo is deprecated — Oxford Nanopore no longer supports it and recommends Remora instead
- ont_fast5_api requires h5py >= 2.6, NumPy >= 1.11, six and progressbar33, which constrain the environment

## Evidence

- [methods] fast5 files should be resquiggled using Tombo software: "Firstly, fast5 files should be resquiggled using [Tombo](https://github.com/nanoporetech/tombo) software."
- [methods] convert to multi-fast5 format: "After resquiggling, fast5 files should be converted to the multi-fast5 format using [ont_fast5_api](https://github.com/nanoporetech/ont_fast5_api)."
- [readme] python 3.7 incompatibility note: "python 3.7 (later versions might be incompatible because of inner biopython dependencies)"
- [readme] Tombo deprecation notice: [evidence span withheld — anchor status: external_doc]
