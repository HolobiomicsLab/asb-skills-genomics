---
name: nanopore-fast5-file-splitting
description: Use when raw Nanopore DRS FAST5 files contain multiple read IDs (multi-read
  format) and downstream tools such as Tombo resquiggle or Guppy require per-read
  single FAST5 files.
when_to_use_negative:
- Input FAST5 files are already single-read format (one read ID per file) — splitting
  is only necessary 'if necessary' per the RedNano workflow
- Downstream analysis does not require per-read FAST5 files, e.g. only FASTQ/BAM outputs
  are needed
edam_operation: http://edamontology.org/operation_3218
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: ont-fast5-api
  role: Provides the multi_to_single_fast5 command used to split multi-read FAST5
    files into single-read FAST5 files (version 4.0.2 in RedNano)
  repo: https://github.com/Derryxu/RedNano
- name: Guppy
  role: Downstream basecaller (v3.1.5) that consumes the single-read FAST5 files with
    --fast5_out using the rna_r9.4.1_70bps_hac.cfg config
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioinformatics/btae375
    title: RNA m6A detection using raw current signals and basecalling errors from
      Nanopore direct RNA sequencing reads
  source_content_hash: sha256:c12e7b21b1fc642525f3039df38eeeac38fab014a2bbb2c1f6d2dfea64c284b2
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Derryxu/RedNano
  tools:
  - Guppy
  - RedNano
  - minimap2
  - ont-fast5-api
  - ont-tombo
  - samtools
  edam_operation: http://edamontology.org/operation_3218
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.46153846153846156
n_claims: 13
---

# nanopore-fast5-file-splitting

## Summary

Splits Nanopore multi-read FAST5 files containing multiple read IDs into single-read FAST5 files using ont_fast5_api's multi_to_single_fast5, as the first preprocessing step of the RedNano m6A detection pipeline for Nanopore direct RNA sequencing (DRS) reads.

## When to use

Use when raw Nanopore DRS FAST5 files contain multiple read IDs (multi-read format) and downstream tools such as Tombo resquiggle or Guppy require per-read single FAST5 files.

## When NOT to use

- Input FAST5 files are already single-read format (one read ID per file) — splitting is only necessary 'if necessary' per the RedNano workflow
- Downstream analysis does not require per-read FAST5 files, e.g. only FASTQ/BAM outputs are needed

## Inputs

- Raw Nanopore multi-read FAST5 files (directory tree of .fast5 files with multiple read IDs)

## Outputs

- Directory of single-read FAST5 files (one FAST5 per read), e.g. /path/to/fast5_single

## How to apply

Run ont_fast5_api's multi_to_single_fast5 with --recursive over the input FAST5 directory so all nested multi-read files are found, specifying an output directory (e.g. /path/to/fast5_single) and multiple threads (-t 30 in RedNano). Each output single-read FAST5 file then feeds into Guppy basecalling with the rna_r9.4.1_70bps_hac.cfg configuration (--fast5_out). The rationale is that RedNano's resquiggle step (tombo resquiggle, corrected-group RawGenocorrected_001) operates per read in the Guppy workspace, so single-read files are a prerequisite for correct signal-to-sequence mapping.

## Related tools

- **ont-fast5-api** (Provides the multi_to_single_fast5 command used to split multi-read FAST5 files into single-read FAST5 files (version 4.0.2 in RedNano)) — https://github.com/Derryxu/RedNano
- **Guppy** (Downstream basecaller (v3.1.5) that consumes the single-read FAST5 files with --fast5_out using the rna_r9.4.1_70bps_hac.cfg config)

## Examples

```
multi_to_single_fast5 -i /path/to/fast5/ -s /path/to/fast5_single --recursive -t 30
```

## Evaluation signals

- Output directory contains exactly one FAST5 file per read ID, with no multi-read files remaining
- Each single-read FAST5 retains the raw signal data and read metadata needed by Guppy basecalling (guppy_basecaller completes without errors on the split files)
- The subsequent Guppy workspace (e.g. fast5_guppy) is populated with basecalled FAST5 files (--fast5_out) ready for tombo resquiggle
- Thread count and --recursive flag cover all nested input FAST5 files (30 threads in RedNano)

## Limitations

- Splitting is only required when the downloaded FAST5 files 'contains multiple reads-id'; applying it to already-single-read files adds unnecessary I/O overhead
- Version-sensitive: RedNano pins ont-fast5-api 4.0.2; other versions may behave differently
- Splitting alone does not basecall — reads are not usable for RedNano feature extraction until after Guppy basecalling and tombo resquiggle (RawGenomeCorrected_001 group)

## Evidence

- [methods] Converts multi read FAST5 file(s) into single read FAST5 files if necessary.: "**Converts multi read FAST5 file(s) into single read FAST5 files if necessary.**"
- [readme] multi_to_single_fast5 -i /path/to/fast5/ -s /path/to/fast5_single --recursive -t 30: "multi_to_single_fast5 -i /path/to/fast5/ -s /path/to/fast5_single --recursive -t 30"
- [readme] When the downloaded Fast5 file(s) of raw nanopore sequencing reads contains multiple reads-id, use ont_fast5_api to split: "When the downloaded Fast5 file(s) of raw nanopore sequencing reads contains multiple reads-id, use ont_fast5_api to split"
- [methods] each basecalled read is aligned to the reference transcriptome using minimap2 (v2.17): [evidence span withheld — anchor status: not_found]
