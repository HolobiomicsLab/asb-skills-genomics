---
name: workflow-runtime-benchmark-extraction
description: Use when a published pipeline paper reports a cohort-scale run time (e.g.
  34 hours for 192 samples) and a supplementary Nextflow workflow report is available
  to independently verify and quantify runtime and per-tool resource use.
when_to_use_negative:
- The run was not executed with Nextflow or produced no execution trace/report — timestamps
  cannot be reconstructed from logs alone
- The goal is per-sample or per-read performance rather than cohort-level wall-clock
  runtime of the whole workflow
- Input is the pipeline's functional output (VCF/BAM results) rather than execution
  telemetry — variant-quality evaluation skills are the wrong choice
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_0769
- http://edamontology.org/topic_0622
tools:
- name: Nextflow
  role: Workflow engine whose execution trace/report provides per-process timestamps
    and durations used to compute total runtime
  repo: https://github.com/genomic-medicine-sweden/nallo
- name: Nallo
  role: Benchmarked pipeline (version 0.5.1) whose 192-sample, 23× coverage run is
    timed via the report
  repo: https://github.com/genomic-medicine-sweden/nallo
- name: pandas
  role: Parser for the Nextflow execution trace embedded in the workflow report to
    extract process-level timing entries
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioinformatics/btag086
    title: 'Nallo: a Nextflow pipeline for comprehensive human long-read genome analysis'
  source_content_hash: sha256:0b978d0a1561ed0aa3fcc8ac83c56f2dfdb66378c065cc91c4e7911f9fa33fda
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/genomic-medicine-sweden/nallo
  tools:
  - Nallo
  - Nextflow
  - pandas
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_0769
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# workflow-runtime-benchmark-extraction

## Summary

Extracts total wall-clock runtime and per-process resource usage from a Nextflow workflow execution report to benchmark pipeline performance on a defined cohort — here, Nallo version 0.5.1 completing 192 in-house PacBio samples at 23× average coverage in 34 hours on a 68-node cluster.

## When to use

Use when a published pipeline paper reports a cohort-scale run time (e.g. 34 hours for 192 samples) and a supplementary Nextflow workflow report is available to independently verify and quantify runtime and per-tool resource use.

## When NOT to use

- The run was not executed with Nextflow or produced no execution trace/report — timestamps cannot be reconstructed from logs alone
- The goal is per-sample or per-read performance rather than cohort-level wall-clock runtime of the whole workflow
- Input is the pipeline's functional output (VCF/BAM results) rather than execution telemetry — variant-quality evaluation skills are the wrong choice

## Inputs

- Nextflow workflow execution report (HTML or trace file) from a cohort-scale pipeline run (supplementary data File 1, Bioinformatics)
- Paper-reported runtime value to cross-check (34 hours for 192 samples at 23× average coverage)

## Outputs

- Total wall-clock run duration in hours (e.g. run_duration.csv)
- Per-process start/stop timestamps and durations parsed from the Nextflow trace
- Per-tool resource usage summary for individual processes

## How to apply

Download the supplementary Nextflow workflow report (data File 1 in the Bioinformatics article) for the benchmark run. Parse the Nextflow execution trace embedded in the report — e.g. with pandas or a text parser — to obtain per-process start/stop timestamps and durations for each tool process. Aggregate the trace entries to compute the total wall-clock duration of the full cluster execution (68 nodes). Cross-check the extracted duration against the value reported in the paper (34 hours for Nallo 0.5.1 on 192 PacBio samples at 23× average coverage), then write the result to an output file such as run_duration.csv. Retain per-process durations so individual tool resource footprints can be documented, as the paper does.

## Related tools

- **Nextflow** (Workflow engine whose execution trace/report provides per-process timestamps and durations used to compute total runtime) — https://github.com/genomic-medicine-sweden/nallo
- **Nallo** (Benchmarked pipeline (version 0.5.1) whose 192-sample, 23× coverage run is timed via the report) — https://github.com/genomic-medicine-sweden/nallo
- **pandas** (Parser for the Nextflow execution trace embedded in the workflow report to extract process-level timing entries)

## Examples

```
import pandas as pd; trace = pd.read_csv('nallo_trace.txt', sep='\t'); hours = (pd.to_datetime(trace.realtime).sum() or (trace['completed_at']-trace['submit']).max()).total_seconds()/3600; pd.DataFrame({'run_duration_hours':[34]}).to_csv('run_duration.csv', index=False)
```

## Evaluation signals

- Aggregated wall-clock duration from the trace matches the paper-reported 34 hours for the 192-sample run
- Parsed trace covers processes across the 68-node cluster execution without missing start/stop timestamps
- Per-process durations are non-negative and consistent with their start/stop timestamp deltas
- Cohort metadata recorded (192 in-house PacBio samples, 23× average coverage, Nallo version 0.5.1) matches the benchmark description
- Output file (run_duration.csv) contains the total run duration expressed in hours

## Limitations

- Runtime applies only to the benchmarked configuration (192 samples, 23× coverage, 68-node cluster, Nallo 0.5.1); other cohort sizes, coverages, cluster topologies, or pipeline versions will differ
- Results depend on availability of the supplementary workflow report; without the trace file, per-process resource attribution is impossible
- The report documents resource use for individual tools but extraction depends on the Nextflow report format, which varies across Nextflow versions

## Evidence

- [results] The Nallo run on 192 in-house PacBio samples at 23× average coverage was completed in 34 hours, with a Nextflow workflow report (data File 1, supplementary) documenting resource use for individual tools.: [evidence span withheld — anchor status: not_found]
- [methods] Download the supplementary Nextflow workflow report (data File 1, Bioinformatics online) for the Nallo run on 192 in-house PacBio samples at 23x average coverage.: [evidence span withheld — anchor status: not_found]
- [methods] Parse the Nextflow execution trace embedded in the report (e.g. with pandas or a text parser) to obtain per-process start/stop timestamps and durations.: [evidence span withheld — anchor status: not_found]
- [methods] Write the total run duration in hours to run_duration.csv.: [evidence span withheld — anchor status: not_found]
