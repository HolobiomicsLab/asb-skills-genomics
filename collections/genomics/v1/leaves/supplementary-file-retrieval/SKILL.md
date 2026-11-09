---
name: supplementary-file-retrieval
description: Use when an article claim (e.g. 'the 192-sample Nallo run completed in
  34 hours') is backed by a deposited supplementary file, and the claim must be independently
  verified from the file's raw contents.
when_to_use_negative:
- No supplementary workflow report or execution trace is deposited alongside the article,
  so the claim cannot be independently parsed.
- The question concerns biological/variant-call outputs of the pipeline rather than
  execution resource-use metrics.
- The run reported is single-sample or small-scale rather than the multi-node (68-node)
  cohort execution the trace aggregation assumes.
edam_operation: http://edamontology.org/operation_2422
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: Nallo
  role: Pipeline whose run on 192 in-house PacBio samples at 23× average coverage
    generated the supplementary Nextflow workflow report being retrieved and parsed
  repo: https://github.com/genomic-medicine-sweden/nallo
- name: Nextflow
  role: Workflow manager whose embedded execution trace in the report provides per-process
    timestamps and durations used to compute total run time
  repo: https://www.nextflow.io/
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioinformatics/btag086
    title: 'Nallo: a Nextflow pipeline for comprehensive human long-read genome analysis'
  source_content_hash: sha256:5653da122cb8f6e9de42237022c0a852e08b77327197209fe87739893835b199
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/genomic-medicine-sweden/nallo
  tools:
  - Nallo
  - Nextflow
  edam_operation: http://edamontology.org/operation_2422
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# supplementary-file-retrieval

## Summary

Retrieve and parse a supplementary file (here, the Nextflow workflow report deposited as data File 1 in Bioinformatics online) to extract quantitative evidence — e.g. total wall-clock run duration — that corroborates a claim made in the article text.

## When to use

Use when an article claim (e.g. 'the 192-sample Nallo run completed in 34 hours') is backed by a deposited supplementary file, and the claim must be independently verified from the file's raw contents.

## When NOT to use

- No supplementary workflow report or execution trace is deposited alongside the article, so the claim cannot be independently parsed.
- The question concerns biological/variant-call outputs of the pipeline rather than execution resource-use metrics.
- The run reported is single-sample or small-scale rather than the multi-node (68-node) cohort execution the trace aggregation assumes.

## Inputs

- Nextflow workflow report (supplementary data File 1, Bioinformatics online)
- Embedded Nextflow execution trace with per-process timestamps and durations
- Article-reported value to cross-check (34 hours for the 192-sample, 23× coverage run)

## Outputs

- Total wall-clock run duration in hours
- run_duration.csv containing the extracted duration
- Per-process start/stop timestamps and durations extracted from the trace

## How to apply

Locate the supplementary file cited in the article (data File 1, Bioinformatics online) for the specific run in question — here the Nallo 0.5.1 run on 192 in-house PacBio samples at 23× average coverage. Download it and parse the embedded Nextflow execution trace (e.g. with pandas or a text parser) to obtain per-process start/stop timestamps and durations. Aggregate the trace entries across the 68-node cluster execution to compute the total wall-clock run duration. Cross-check the extracted duration against the value reported in the paper (34 hours) before writing the result (e.g. hours into run_duration.csv). The rationale is that the workflow report documents resource use for individual tools, giving a reproducible, per-process ground truth for the aggregate claim.

## Related tools

- **Nallo** (Pipeline whose run on 192 in-house PacBio samples at 23× average coverage generated the supplementary Nextflow workflow report being retrieved and parsed) — https://github.com/genomic-medicine-sweden/nallo
- **Nextflow** (Workflow manager whose embedded execution trace in the report provides per-process timestamps and durations used to compute total run time) — https://www.nextflow.io/

## Examples

```
python -c "import pandas as pd; t = pd.read_csv('trace.txt', sep='\t'); print((pd.to_datetime(t.completed)-pd.to_datetime(t.submitted)).max())"
```

## Evaluation signals

- Aggregated trace duration across the 68-node cluster execution equals the 34 hours reported in the paper.
- Extracted per-process start/stop timestamps are internally consistent (stop after start; no negative durations).
- Parsed trace covers all processes documented in the report's resource-use section for individual tools.
- Output run_duration.csv contains the duration in hours and matches the cross-check value.

## Limitations

- Wall-clock duration depends on cluster configuration (68 nodes) and coverage (23×); values do not transfer to other datasets or hardware.
- The trace documents per-process resource use but does not capture queueing or scheduling delays outside Nextflow's accounting.
- Version-specific: the report documents a Nallo version 0.5.1 run; later pipeline versions may differ in runtime.

## Evidence

- [other] The Nallo run on 192 in-house PacBio samples at 23× average coverage was completed in 34 hours, with a Nextflow workflow report (data File 1, supplementary) documenting resource use for individual tools.: [evidence span withheld — anchor status: not_found]
- [supplementary] Parse the Nextflow execution trace embedded in the report (e.g. with pandas or a text parser) to obtain per-process start/stop timestamps and durations.: [evidence span withheld — anchor status: not_found]
- [full_text] Aggregate the trace entries to compute the total wall-clock run duration of the 68-node cluster execution.: [evidence span withheld — anchor status: not_found]
- [readme] The workflow uses Nextflow to manage compute and software resources.: "The workflow uses [Nextflow](https://www.nextflow.io/) to manage
compute and software resources"
