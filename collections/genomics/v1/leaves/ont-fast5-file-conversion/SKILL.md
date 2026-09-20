---
name: ont-fast5-file-conversion
description: Use when raw ONT direct RNA sequencing output consists of multi-read
  FAST5 files that must be split into single-read FAST5 files prior to Tombo resquiggle
  or signal-based modification analysis.
when_to_use_negative:
- Input data are already single-read FAST5 files (e.g. previously converted output)
  — re-conversion is unnecessary
- The downstream tool does not require single-read FAST5 files (e.g. basecalled FASTQ/BAM
  analysis pipelines)
- Data are not ONT FAST5 files (e.g. POD5 or other formats), since multi_to_single_fast5
  expects the ont_fast5 API FAST5 layout
edam_operation: http://edamontology.org/operation_0222
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: multi_to_single_fast5
  role: Core conversion command from the Oxford Nanopore ont_fast5 API that splits
    multi-read FAST5 files into single-read FAST5 files with --recursive, --save_path,
    --flowcell, --kit, --fast5_out, and --cpu_threads_per_Caller
  repo: github.com/physnano/rRNA_nanoSHAPE
provenance:
  source_task_ids:
  - task_006
  - task_001
  source_papers:
  - doi: 10.1016/j.xgen.2022.100097
    title: Direct detection of RNA modifications and structure using single-molecule
      nanopore sequencing
  source_content_hash: sha256:70483ba9379ba9d516572548ca4c843f5cdbcffd619ff9e4ed7528e2aae4363f
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
  edam_operation: http://edamontology.org/operation_0222
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# ont-fast5-file-conversion

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Converts Oxford Nanopore multi-read FAST5 files into per-read single FAST5 files using the ont_fast5 API's multi_to_single_fast5, the required first step before Tombo resquiggle and modification detection in the nanoSHAPE workflow.

## When to use

Use when raw ONT direct RNA sequencing output consists of multi-read FAST5 files that must be split into single-read FAST5 files prior to Tombo resquiggle or signal-based modification analysis.

## When NOT to use

- Input data are already single-read FAST5 files (e.g. previously converted output) — re-conversion is unnecessary
- The downstream tool does not require single-read FAST5 files (e.g. basecalled FASTQ/BAM analysis pipelines)
- Data are not ONT FAST5 files (e.g. POD5 or other formats), since multi_to_single_fast5 expects the ont_fast5 API FAST5 layout

## Inputs

- Directory of ONT multi-read FAST5 files from direct RNA nanopore sequencing
- Flowcell specification for the sequencing run
- Kit specification for the sequencing run

## Outputs

- Directory of per-read single FAST5 files (--fast5_out) ready for Tombo resquiggle

## How to apply

Run multi_to_single_fast5 from the Oxford Nanopore ont_fast5 API with -i pointing at the directory of multi fast5 files and --recursive to traverse subdirectories. Set --save_path to the output directory and supply the --flowcell and --kit specifications matching the sequenced run, which Tombo needs to correctly model the raw signal. Enable --fast5_out so per-read single FAST5 files are written, and set --cpu_threads_per_Caller to the available thread count for parallel conversion. Both the modified (e.g. AcIm-treated) and control/unmodified read sets must be converted before resquiggling. Validate that the output directory contains one usable single-read FAST5 file per input read for downstream 'tombo resquiggle'.

## Related tools

- **multi_to_single_fast5** (Core conversion command from the Oxford Nanopore ont_fast5 API that splits multi-read FAST5 files into single-read FAST5 files with --recursive, --save_path, --flowcell, --kit, --fast5_out, and --cpu_threads_per_Caller) — github.com/physnano/rRNA_nanoSHAPE

## Examples

```
multi_to_single_fast5 -i <directory of multi fast5s> --recursive --save_path <output path> --flowcell <flowcell spec> --kit <kit spec> --fast5_out --cpu_threads_per_Caller <num threads>
```

## Evaluation signals

- Output save_path directory contains one single-read FAST5 file per input read
- Converted single-read FAST5 files load successfully in the downstream 'tombo resquiggle' step without file-format errors
- Both modified and control/unmodified read sets were converted before resquiggling, as the workflow requires
- --flowcell and --kit values match the actual sequencing run so Tombo can model the raw signal correctly

## Limitations

- Conversion must be repeated separately for each read set (modified and unmodified) before comparison
- Correct --flowcell and --kit specifications are required; mismatched values can render files unusable for Tombo signal modeling
- This is only a prerequisite step — it does not perform resquiggling, modification detection, or any signal analysis itself

## Evidence

- [other] Convert multi-read files to per-read format with multi_to_single_fast5 (--recursive, --fast5_out, specifying --flowcell and --kit for the sequenced run).: [evidence span withheld — anchor status: not_found]
