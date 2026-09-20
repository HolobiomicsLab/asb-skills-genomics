---
name: high-throughput-sequencing-pipeline-profiling
description: Use when you need to measure total wall-clock runtime and per-process
  resource use of a high-throughput Nextflow pipeline run, e.g. quantifying Nallo's
  34-hour analysis of 192 PacBio samples at 23× coverage from its execution report.
when_to_use_negative:
- You need the biological outputs of Nallo (variant calls, genome assembly, CpG methylation)
  rather than measurements of the run's speed and resource consumption.
- No Nextflow execution trace or workflow report exists for the run (e.g. the pipeline
  was executed with a different engine), so per-process timestamps and durations cannot
  be extracted.
- The run is a short single-sample or test-data execution whose duration is not representative
  of the high-throughput 68-node cluster scenario this skill profiles.
edam_operation: http://edamontology.org/operation_2409
tools:
- name: Nextflow
  role: Workflow engine whose execution trace/report supplies per-process start/stop
    timestamps, durations, and resource use used for profiling
  repo: https://www.nextflow.io/
- name: Nallo
  role: The Nextflow pipeline being profiled (version 0.5.1) for the 192-sample, 23×
    PacBio HiFi run
  repo: https://github.com/genomic-medicine-sweden/nallo
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioinformatics/btag086
    title: 'Nallo: a Nextflow pipeline for comprehensive human long-read genome analysis'
  source_content_hash: sha256:7173fe386506cd313d3469bbd606c3af815748b2a54a2f48b9002e28b4ddd76c
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Nallo
  - Nextflow
  edam_operation: http://edamontology.org/operation_2409
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# high-throughput-sequencing-pipeline-profiling

## Summary

Profiling the wall-clock runtime and per-tool resource use of a high-throughput Nextflow long-read genome pipeline (Nallo) by parsing the Nextflow execution trace embedded in its workflow report — e.g. confirming that Nallo 0.5.1 analyzed 192 in-house PacBio samples at 23× average coverage in 34 hours on a 68-node cluster.

## When to use

Use when you need to measure total wall-clock runtime and per-process resource use of a high-throughput Nextflow pipeline run, e.g. quantifying Nallo's 34-hour analysis of 192 PacBio samples at 23× coverage from its execution report.

## When NOT to use

- You need the biological outputs of Nallo (variant calls, genome assembly, CpG methylation) rather than measurements of the run's speed and resource consumption.
- No Nextflow execution trace or workflow report exists for the run (e.g. the pipeline was executed with a different engine), so per-process timestamps and durations cannot be extracted.
- The run is a short single-sample or test-data execution whose duration is not representative of the high-throughput 68-node cluster scenario this skill profiles.

## Inputs

- Supplementary Nextflow workflow report (data File 1, Bioinformatics online) for the Nallo run
- Nextflow execution trace with per-process start/stop timestamps, durations, and CPU/memory resource columns

## Outputs

- Total wall-clock run duration in hours, written to run_duration.csv
- Per-process resource-use table attributing runtime and CPU/memory to individual pipeline tools

## How to apply

Download the supplementary Nextflow workflow report (data File 1, Bioinformatics online) for the Nallo run of interest — here Nallo 0.5.1 on 192 in-house PacBio samples at 23× average coverage. Parse the Nextflow execution trace embedded in the report (e.g. with pandas or a text parser) to obtain per-process start/stop timestamps and durations for each of the pipeline's tools (Minimap2 mapping, samtools sorting/indexing, Hifiasm assembly, phasing, methylation pileup steps). Aggregate the trace entries to compute the total wall-clock duration of the 68-node cluster execution, using the trace's per-process resource columns to attribute CPU/memory use to individual tools. Cross-check the extracted duration against the 34 hours reported in the paper, and write the total run duration in hours to run_duration.csv.

## Related tools

- **Nextflow** (Workflow engine whose execution trace/report supplies per-process start/stop timestamps, durations, and resource use used for profiling) — https://www.nextflow.io/
- **Nallo** (The Nextflow pipeline being profiled (version 0.5.1) for the 192-sample, 23× PacBio HiFi run) — https://github.com/genomic-medicine-sweden/nallo

## Examples

```
python -c "import pandas as pd; t=pd.read_csv('trace.txt',sep='\t',parse_dates=['start','complete']); print((t.complete.max()-t.start.min()).total_seconds()/3600)"
```

## Evaluation signals

- Extracted total wall-clock duration matches the 34 hours reported for the 192-sample, 23× average coverage run (cross-check step in the workflow).
- Per-process start/complete timestamps parse as valid datetimes, and every individual process duration is ≤ the total run wall clock.
- Per-tool resource metrics (CPU/memory) are present in the trace, consistent with the report 'documenting resource use for individual tools'.
- run_duration.csv contains a single numeric duration value in hours consistent with the trace-derived aggregate.
- The compute context is recorded and reproducible: 192 in-house PacBio samples at 23× average coverage executed on the 68-node cluster.

## Limitations

- The 34-hour figure is specific to Nallo 0.5.1 analyzing 192 in-house PacBio samples at 23× average coverage on the 68-node cluster; other versions, sample counts, or coverages will not match.
- Run time varies with the analyses requested and with coverage (per the wf-human-variation README), so a profile is not transferable across pipeline options or datasets.
- Only resource use for tools captured in the supplementary Nextflow report (data File 1) is profiled; if the report is unavailable, the skill cannot be applied.

## Evidence

- [results] The Nallo run on 192 in-house PacBio samples at 23× average coverage was completed in 34 hours, with a Nextflow workflow report (data File 1, supplementary): [evidence span withheld — anchor status: not_found]
- [abstract] Here we present Nallo, a Nextflow pipeline for analysis of PacBio and Oxford Nanopore data: "Here we present Nallo, a Nextflow pipeline for analysis of PacBio and Oxford Nanopore data"
- [readme] Approximate run time: Variable depending on whether it is targeted sequencing or whole genome sequencing, as well as coverage and the individual analyses requested.: [evidence span withheld — anchor status: not_found]
- [other] Parse the Nextflow execution trace embedded in the report (e.g. with pandas or a text parser) to obtain per-process start/stop timestamps and durations.: [evidence span withheld — anchor status: not_found]
