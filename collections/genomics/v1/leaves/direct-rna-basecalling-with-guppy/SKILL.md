---
name: direct-rna-basecalling-with-guppy
description: Use when starting RedNano preprocessing from raw multi-read Nanopore
  DRS FAST5 files and basecalled sequences (FASTQ) plus basecall-group FAST5 signals
  are needed for re-squiggling and m6A feature extraction.
when_to_use_negative:
- Input FAST5 files are already basecalled and only re-squiggling or alignment is
  pending — proceed directly to tombo resquiggle / minimap2 instead
- Reads are DNA Nanopore reads or non-r9.4.1 RNA chemistry, where the rna_r9.4.1_70bps_hac.cfg
  configuration would not match the sequencing kit
edam_operation: http://edamontology.org/operation_3182
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: Guppy
  role: Basecaller run with the rna_r9.4.1_70bps_hac.cfg RNA configuration; --fast5_out
    emits the basecalled FAST5 workspace needed for re-squiggling
  repo: https://community.nanoporetech.com/getting_started
- name: ont-fast5-api 4.0.2
  role: Splits multi-read FAST5 files into single-read FAST5 files prior to basecalling
    via multi_to_single_fast5
  repo: https://github.com/Derryxu/RedNano
- name: RedNano
  role: Pipeline whose preprocessing step this basecalling procedure is part of; basecalled
    outputs feed Tombo re-squiggling and feature extraction
  repo: https://github.com/Derryxu/RedNano
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioinformatics/btae375
    title: RNA m6A detection using raw current signals and basecalling errors from
      Nanopore direct RNA sequencing reads
  source_content_hash: sha256:bf65a1c9f5231b1c60ba1166058f57e7cd5a598f7711d076eabdb9ef0081fcca
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Guppy
  - RedNano
  - minimap2
  - ont-fast5-api 4.0.2
  - ont-tombo
  - samtools
  edam_operation: http://edamontology.org/operation_3182
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 15
---

# direct-rna-basecalling-with-guppy

## Summary

Basecalls raw Nanopore direct RNA FAST5 signals with Guppy (v3.1.5) using the rna_r9.4.1_70bps_hac.cfg configuration, producing both FASTQ sequences and basecalled FAST5 files that downstream Tombo re-squiggling in the RedNano pipeline depends on.

## When to use

Use when starting RedNano preprocessing from raw multi-read Nanopore DRS FAST5 files and basecalled sequences (FASTQ) plus basecall-group FAST5 signals are needed for re-squiggling and m6A feature extraction.

## When NOT to use

- Input FAST5 files are already basecalled and only re-squiggling or alignment is pending — proceed directly to tombo resquiggle / minimap2 instead
- Reads are DNA Nanopore reads or non-r9.4.1 RNA chemistry, where the rna_r9.4.1_70bps_hac.cfg configuration would not match the sequencing kit

## Inputs

- Single-read FAST5 files of raw Nanopore direct RNA reads (produced via multi_to_single_fast5 from multi-read FAST5)
- GPU device for Guppy basecalling (e.g. CUDA:0)

## Outputs

- Basecalled FASTQ files (merged into a single test.fastq)
- Basecalled FAST5 workspace from Guppy (--fast5_out), containing the Basecall_1D_001 basecall group consumed by tombo resquiggle

## How to apply

First split multi-read FAST5 files into single-read FAST5 files with ont_fast5_api's multi_to_single_fast5 (--recursive, e.g. 30 threads), since Guppy expects single-read input. Then run guppy_basecaller on the FAST5 directory with -c rna_r9.4.1_70bps_hac.cfg and --fast5_out, tuning GPU usage with --gpu_runners_per_device 2 --chunks_per_runner 2500 --device CUDA:0. The --fast5_out flag is essential because RedNano's subsequent tombo resquiggle step consumes the Guppy workspace against basecall-group Basecall_1D_001. Finally, concatenate all per-run FASTQ files (extracting them with poretools or nanopolish first if necessary) into a single test.fastq used for transcriptome alignment with minimap2. Use the RNA-specific r9.4.1 70bps high-accuracy configuration; a DNA or generic config would misbasecall direct RNA reads.

## Related tools

- **Guppy** (Basecaller run with the rna_r9.4.1_70bps_hac.cfg RNA configuration; --fast5_out emits the basecalled FAST5 workspace needed for re-squiggling) — https://community.nanoporetech.com/getting_started
- **ont-fast5-api 4.0.2** (Splits multi-read FAST5 files into single-read FAST5 files prior to basecalling via multi_to_single_fast5) — https://github.com/Derryxu/RedNano
- **RedNano** (Pipeline whose preprocessing step this basecalling procedure is part of; basecalled outputs feed Tombo re-squiggling and feature extraction) — https://github.com/Derryxu/RedNano

## Examples

```
guppy_basecaller -i /path/to/fast5/ -r -s /path/to/fast5_guppy --fast5_out -c rna_r9.4.1_70bps_hac.cfg --gpu_runners_per_device 2 --chunks_per_runner 2500 --device CUDA:0; cat /path/to/fast5_guppy/*.fastq > test.fastq
```

## Evaluation signals

- test.fastq exists and is non-empty after concatenating /path/to/fast5_guppy/*.fastq
- The Guppy output workspace (/path/to/fast5_guppy/workspace/) contains basecalled FAST5 files with the Basecall_1D_001 group, which tombo resquiggle requires
- Guppy runs without GPU errors using --gpu_runners_per_device 2 --chunks_per_runner 2500 --device CUDA:0
- Downstream tombo resquiggle with --basecall-group Basecall_1D_001 successfully produces the RawGenomeCorrected_001 corrected-group workspace, confirming basecalled FAST5 outputs are well-formed

## Limitations

- Guppy version matters: the RedNano README specifies Guppy (v3.1.5); other versions may change basecall group naming expected by Tombo
- If FASTQ files are not directly produced, poretools or nanopolish must first extract FASTQ from FAST5 files before merging
- GPU resources constrain throughput: only 2 GPU runners per device with 2500 chunks per runner are specified in the documented configuration

## Evidence

- [readme] Basecalling using guppy (v3.1.5): "2. **Basecalling using guppy (v3.1.5)**"
- [readme] guppy_basecaller command with RNA config and GPU settings: "guppy_basecaller -i /path/to/fast5/ -r -s /path/to/fast5_guppy --fast5_out -c rna_r9.4.1_70bps_hac.cfg --gpu_runners_per_device 2 --chunks_per_runner 2500 --device CUDA:0"
- [readme] Merging FASTQ files into a single file: "# Merge all fastq files into one file, use poretools or nanopolish to extract the fastq files from fast5 files first if necessary"
- [methods] Converting multi-read FAST5 into single-read FAST5: "**Converts multi read FAST5 file(s) into single read FAST5 files if necessary.**"
- [readme] Tombo resquiggle depends on the basecall group from Guppy output: "tombo resquiggle --overwrite /path/to/fast5_guppy/workspace/ --basecall-group Basecall_1D_001 referance.transcript.fa --fit-global-scale --include-event-stdev --corrected-group RawGenomeCorrected_001"
