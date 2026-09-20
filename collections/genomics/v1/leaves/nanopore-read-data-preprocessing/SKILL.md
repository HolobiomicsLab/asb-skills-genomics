---
name: nanopore-read-data-preprocessing
description: Use when input data are ONT multi-read fast5 files that must be split
  into per-read single fast5 files before raw-signal tools like tombo resquiggle can
  process them.
when_to_use_negative:
- Input files are already per-read single fast5 files and can be passed directly to
  tombo resquiggle
- The downstream analysis does not use Tombo or other raw-signal fast5 tools, so single-read
  splitting is unnecessary
- Data are basecalled FASTQ/BAM reads rather than raw-signal fast5 files
edam_operation: http://edamontology.org/operation_0232
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: multi_to_single_fast5
  role: Performs the multi-fast5 to single-read fast5 conversion, traversing the input
    directory recursively and writing single-read fast5 files with flowcell/kit metadata
  repo: github.com/physnano/rRNA_nanoSHAPE
- name: tombo
  role: Downstream consumer of the single fast5 files; tombo resquiggle performs the
    raw-signal-to-sequence alignment that requires single-read fast5 input
  repo: github.com/physnano/rRNA_nanoSHAPE
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1016/j.xgen.2022.100097
    title: Direct detection of RNA modifications and structure using single-molecule
      nanopore sequencing
  source_content_hash: sha256:62213b0e71f7f81654837086a014eba519a5630201c56f3990939e676a4ec48b
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/physnano/rRNA_nanoSHAPE
  tools:
  - multi_to_single_fast5
  - tombo
  edam_operation: http://edamontology.org/operation_0232
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# nanopore-read-data-preprocessing

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Converts ONT multi-read fast5 files into per-read single fast5 files using the multi_to_single_fast5 command from the Oxford Nanopore ont_fast5 API, a required first step before Tombo resquiggle and modification detection in the nanoSHAPE workflow.

## When to use

Use when input data are ONT multi-read fast5 files that must be split into per-read single fast5 files before raw-signal tools like tombo resquiggle can process them.

## When NOT to use

- Input files are already per-read single fast5 files and can be passed directly to tombo resquiggle
- The downstream analysis does not use Tombo or other raw-signal fast5 tools, so single-read splitting is unnecessary
- Data are basecalled FASTQ/BAM reads rather than raw-signal fast5 files

## Inputs

- Directory of ONT multi-read fast5 files (from direct RNA nanopore sequencing runs)
- Flowcell specification for the sequencing run
- Sequencing kit specification for the sequencing run

## Outputs

- Directory of per-read single fast5 files at the specified --save_path, one file per input read

## How to apply

Run multi_to_single_fast5 from the Oxford Nanopore ont_fast5 API with -i pointing at the directory containing multi fast5 files and --recursive to traverse subdirectories. Set --save_path to the output directory, and supply the matching --flowcell and --kit specifications for the sequencing run so read metadata is correctly assigned. Enable --fast5_out so per-read single fast5 files are written, and set --cpu_threads_per_Caller to the available thread count for parallel conversion. Both the MODIFIED and CONTROL/UNMODIFIED read sets (e.g., AcIm-treated and native RNA in nanoSHAPE) must be converted, since the downstream tombo detect_modifications model_sample_compare step requires both. Validation: confirm the output directory contains one single-read fast5 file per input read usable for tombo resquiggle.

## Related tools

- **multi_to_single_fast5** (Performs the multi-fast5 to single-read fast5 conversion, traversing the input directory recursively and writing single-read fast5 files with flowcell/kit metadata) — github.com/physnano/rRNA_nanoSHAPE
- **tombo** (Downstream consumer of the single fast5 files; tombo resquiggle performs the raw-signal-to-sequence alignment that requires single-read fast5 input) — github.com/physnano/rRNA_nanoSHAPE

## Examples

```
multi_to_single_fast5 -i <directory of multi fast5s> --recursive --save_path <output path> --flowcell <flowcell spec> --kit <kit spec> --fast5_out --cpu_threads_per_Caller <num threads>
```

## Evaluation signals

- Output directory at --save_path contains one single-read fast5 file per input read from the multi fast5 source
- Single fast5 files carry the correct --flowcell and --kit specifications matching the sequencing run
- Downstream tombo resquiggle runs successfully on the converted single fast5 directory without missing-read errors
- Both MODIFIED and CONTROL/UNMODIFIED read sets have been converted, since sample-compare modification detection requires both

## Limitations

- This is a required preprocessing step specific to Tombo-based processing; reads must be converted before Tombo can resquiggle them
- Flowcell and kit specifications must match the sequencing run, or downstream Tombo processing may fail
- Conversion efficiency is bounded by --cpu_threads_per_Caller; underprovisioning threads slows conversion of large multi fast5 directories

## Evidence

- [methods] multi_to_single_fast5 -i <directory of multi fast5s> --recursive --save_path <output path>: "multi_to_single_fast5 
		-i <directory of multi fast5s> 
		--recursive 
		--save_path <output path>"
- [readme] First the multi-fast5 files must be converted to single read fast5 files prior to processing with tombo.: "First the multi-fast5 files must be converted to single read fast5 files prior to processing with tombo."
