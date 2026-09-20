---
name: nextflow-multi-node-workflow-orchestration
description: Use when a long-read genome analysis (FASTQ inputs, GRCh38 reference)
  must be executed end-to-end on a cluster and you need reproducible resource accounting
  (allocated cpus, duration, core-hours) from the Nextflow trace report.
when_to_use_negative:
- You only need a single-tool, single-node run (e.g. one Sniffles2 or Minimap2 invocation)
  where Nextflow orchestration overhead is unnecessary
- The task is short-read or targeted sequencing analysis outside the long-read rare-disease
  scope of Nallo-style pipelines
- No cluster scheduler or container runtime is available and only a single workstation
  is in scope
edam_topics:
- http://edamontology.org/topic_0769
tools:
- name: Nallo
  role: Target Nextflow pipeline being orchestrated; performs mapping, variant detection,
    assembly, phasing and methylation reporting for PacBio/ONT data
  repo: https://github.com/genomic-medicine-sweden/nallo
- name: Nextflow
  role: Workflow engine that manages multi-node execution, containerization profiles
    and trace reporting used for core-hour accounting
  repo: https://github.com/genomic-medicine-sweden/nallo
- name: Docker
  role: Container runtime required so pipeline processes execute reproducibly across
    cluster nodes
- name: nf-core tools
  role: Used with nf-test in the developer environment to lint and test the Nextflow
    pipeline
- name: nf-test
  role: Test framework validating pipeline behaviour after orchestration changes
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioinformatics/btag086
    title: 'Nallo: a Nextflow pipeline for comprehensive human long-read genome analysis'
  source_content_hash: sha256:c6733365c5d066a07fe3f2078dcea81a89ba82dad90fd6c7ed0d652a5d9b3df7
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/genomic-medicine-sweden/nallo
  tools:
  - Docker
  - Nallo
  - Nextflow
  - nf-core tools
  - nf-test
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0769
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 10
---

# nextflow-multi-node-workflow-orchestration

## Summary

Orchestrate a multi-process bioinformatics pipeline (e.g. Nallo for PacBio/ONT long-read genomes) across a multi-node cluster using Nextflow with containerized execution, while recording per-process CPU allocation and runtime for core-hour and wall-clock benchmarking.

## When to use

Use when a long-read genome analysis (FASTQ inputs, GRCh38 reference) must be executed end-to-end on a cluster and you need reproducible resource accounting (allocated cpus, duration, core-hours) from the Nextflow trace report.

## When NOT to use

- You only need a single-tool, single-node run (e.g. one Sniffles2 or Minimap2 invocation) where Nextflow orchestration overhead is unnecessary
- The task is short-read or targeted sequencing analysis outside the long-read rare-disease scope of Nallo-style pipelines
- No cluster scheduler or container runtime is available and only a single workstation is in scope

## Inputs

- ~32x PacBio HiFi WGS FASTQ files for GIAB trio HG002/HG003/HG004
- GRCh38 reference genome and associated resource files
- Nextflow pipeline release (e.g. Nallo v0.5.1 from genomic-medicine-sweden/nallo)
- Cluster of 68 nodes with dual Intel Xeon Gold 6248R CPUs and a container runtime (Docker or Singularity)

## Outputs

- Nextflow trace report listing allocated cpus, duration and real-time per process
- Per-sample core-hours and total wall-clock completion time (aggregated e.g. to nallo_trio_runtime_benchmark.csv)
- Pipeline outputs: aligned BAM/CRAM, VCFs, haplotype-resolved assembly (GFA/FASTA), methylation pileups

## How to apply

Fetch the pipeline from GitHub (e.g. genomic-medicine-sweden/nallo) and launch it with Nextflow using a containerization profile (Docker or Singularity) so every process runs in an isolated container. Supply the input samplesheet — here ~32x PacBio HiFi WGS FASTQs for GIAB samples HG002, HG003 and HG004 plus GRCh38 reference resources — and enable Nextflow trace reporting so each process logs allocated cpus and real-time duration. After completion, parse the trace file (e.g. with pandas) and compute per-sample core-hours as allocated cpus x runtime summed over all processes; the published benchmark for Nallo v0.5.1 full analysis (alignment, variant calling, de novo assembly) is ~344 core hours per sample and 5 h 52 min wall-clock on a 68-node cluster of dual Intel Xeon Gold 6248R machines. Rationale for containerization and trace-based accounting comes from the pipeline's own developer documentation, which mandates Nextflow plus Docker/Singularity profiles and nf-test-based validation.

## Related tools

- **Nallo** (Target Nextflow pipeline being orchestrated; performs mapping, variant detection, assembly, phasing and methylation reporting for PacBio/ONT data) — https://github.com/genomic-medicine-sweden/nallo
- **Nextflow** (Workflow engine that manages multi-node execution, containerization profiles and trace reporting used for core-hour accounting) — https://github.com/genomic-medicine-sweden/nallo
- **Docker** (Container runtime required so pipeline processes execute reproducibly across cluster nodes)
- **nf-core tools** (Used with nf-test in the developer environment to lint and test the Nextflow pipeline)
- **nf-test** (Test framework validating pipeline behaviour after orchestration changes)

## Examples

```
nextflow run genomic-medicine-sweden/nallo -r v0.5.1 -profile docker -with-trace trace.txt --input samplesheet.csv --fasta GRCh38.fa
```

## Evaluation signals

- Per-sample core-hours computed from the trace (allocated cpus x runtime summed over all processes) are consistent with the published ~344 core hours per sample
- Total wall-clock completion time per sample is on the order of the published 5 hours 52 minutes on the 68-node Xeon Gold 6248R cluster
- Every process in the trace report has a non-zero allocated cpus value and a recorded real-time duration
- All processes executed under the chosen container profile (Docker/Singularity) rather than natively
- Pipeline completes all expected outputs (alignment, variant calling, assembly, phasing, methylation) without failed tasks in the trace

## Limitations

- Benchmark figures are specific to Nallo v0.5.1 on ~32x PacBio HiFi GIAB trio data and a 68-node dual-Xeon-Gold-6248R cluster; other data volumes or hardware will shift core-hours and wall-clock
- Docker daemon must be running when executing containerized runs, as noted in the developer setup instructions
- Similar ONT-oriented pipelines (e.g. epi2me-labs/wf-human-variation) require at least 20x coverage to run and recommend >30x, so coverage below thresholds can cause failure or unexpected outcomes

## Evidence

- [abstract] Here we present Nallo, a Nextflow pipeline for analysis of PacBio and Oxford Nanopore data: "Here we present Nallo, a Nextflow pipeline for analysis of PacBio and Oxford Nanopore data"
- [intro] Install Docker ... and make sure the daemon is running when you want to run the tests locally.: "Install Docker (https://www.docker.com/products/docker-desktop/) and make sure the daemon is running when you want to run the tests locally."
- [readme] The workflow uses Nextflow to manage compute and software resources: "The workflow uses [Nextflow](https://www.nextflow.io/) to manage
compute and software resources,"
- [readme] Containerization controlled by the -profile parameter: "This is controlled by the
[`-profile`](https://www.nextflow.io/docs/latest/config.html#config-profiles)
parameter as exemplified below."
- [readme] epi2me-labs/wf-human-variation minimum coverage requirement: [evidence span withheld — anchor status: not_found]
