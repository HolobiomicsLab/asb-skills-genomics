---
name: nextflow-profile-configuration
description: Use when launching an nf-core pipeline (e.g. nf-core/isoseq) on a specific
  HPC, container engine, or the bundled nf-core 'test' dataset, or when offline/custom
  institutional configuration is needed.
when_to_use_negative:
- The workflow is not Nextflow-based (e.g. a standalone SMRT Link / PacBio IsoSeq
  GUI analysis), so profiles and nf-core/configs do not apply
- You only need to run individual tools (ccs, lima, isoseq3) outside a pipeline rather
  than a configured end-to-end Nextflow launch
- The compute environment already has its own working pipeline config and you would
  risk overriding it with a duplicate custom profile
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: Nextflow
  role: Workflow engine that executes nf-core/isoseq and consumes the -profile / -c
    configuration mechanism
  repo: https://github.com/nf-core/isoseq
- name: nf-core/configs
  role: Repository of institutional Nextflow config profiles applied via `-profile
    <config_name>`
  repo: https://github.com/nf-core/configs
- name: nf-core/isoseq
  role: Target pipeline launched with a test or institutional profile to reproduce
    the Iso-Seq annotation workflow
  repo: https://github.com/nf-core/isoseq
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioinformatics/btad150
    title: 'nf-core/isoseq: simple gene and isoform annotation with PacBio Iso-Seq
      long-read sequencing'
  source_content_hash: sha256:1cf87644a6213fc0202162e6976f23274d2a5ab0154dd113da3416f174d8c1e4
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/nf-core/isoseq
  tools:
  - BAMTools
  - BioPerl
  - CCS
  - IsoSeq (isoseq3 refine)
  - LIMA
  - Nextflow
  - TAMA
  - nf-core/configs
  - nf-core/isoseq
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5454545454545454
n_claims: 11
---

# nextflow-profile-configuration

## Summary

Configure and launch nf-core pipelines such as nf-core/isoseq using Nextflow profiles (-profile), including institutional configs from nf-core/configs and bundled test profiles, so the pipeline is correctly parameterized for a given compute environment and dataset.

## When to use

Use when launching an nf-core pipeline (e.g. nf-core/isoseq) on a specific HPC, container engine, or the bundled nf-core 'test' dataset, or when offline/custom institutional configuration is needed.

## When NOT to use

- The workflow is not Nextflow-based (e.g. a standalone SMRT Link / PacBio IsoSeq GUI analysis), so profiles and nf-core/configs do not apply
- You only need to run individual tools (ccs, lima, isoseq3) outside a pipeline rather than a configured end-to-end Nextflow launch
- The compute environment already has its own working pipeline config and you would risk overriding it with a duplicate custom profile

## Inputs

- Nextflow pipeline name or local pipeline directory (e.g. nf-core/isoseq)
- Profile name string (institutional HPC name, container engine such as docker/singularity, or 'test')
- Custom Nextflow config file (.config) with params section for institutional settings
- Samplesheet of samples/runs (for the isoseq pipeline, PacBio subreads plus reference genome)

## Outputs

- A configured Nextflow run of the pipeline for the chosen execution environment
- On the 'test' profile: full nf-core/isoseq test-dataset outputs including per-sample merged annotations at 10_GSTAMA_MERGE/<sample>.bed
- Pipeline run reports (nextflow execution artifacts, software versions)

## How to apply

Launch the pipeline with `nextflow run nf-core/isoseq -profile <name>`; nf-core/configs hosts institutional config files that set executor, Singularity scope, and pipeline-specific params so a single -profile flag configures the whole environment. For offline clusters, download the pipeline with the `nf-core download` helper so the config profiles are shipped with the files, then point at them via `params.custom_config_base` or a `-c` custom config. Use the 'test' profile to run the bundled nf-core test dataset (PacBio subreads plus reference genome) as an end-to-end smoke test of the isoseq workflow (CCS → LIMA → isoseq3 refine → BAMTools → TAMA polyA cleanup → minimap2/uLTRA → TAMA merge). Profiles can be combined with container engines (e.g. `-profile test,docker`). When defaults are insufficient, supply a custom config with the Nextflow `-c` parameter rather than editing pipeline files.

## Related tools

- **Nextflow** (Workflow engine that executes nf-core/isoseq and consumes the -profile / -c configuration mechanism) — https://github.com/nf-core/isoseq
- **nf-core/configs** (Repository of institutional Nextflow config profiles applied via `-profile <config_name>`) — https://github.com/nf-core/configs
- **nf-core/isoseq** (Target pipeline launched with a test or institutional profile to reproduce the Iso-Seq annotation workflow) — https://github.com/nf-core/isoseq

## Examples

```
nextflow run nf-core/isoseq -profile test,docker
```

## Evaluation signals

- The chosen profile resolves without 'Unknown config attribute' errors (e.g. institutional env vars like USER_SCRATCH have fallback values defined)
- With the 'test' profile, the run completes all stages: CCS, LIMA, isoseq refine, BAMTools convert, TAMA polyA cleanup, minimap2/uLTRA alignment, GSTAMA merge
- Each expected output 10_GSTAMA_MERGE/<sample>.bed exists, is non-empty, and conforms to BED12 (12 tab-delimited columns)
- A downloaded offline pipeline still loads its config profiles correctly via params.custom_config_base / relative file path
- Config files satisfy nf-core profile requirements: params.config_profile_description, config_profile_contact, and config_profile_url are set

## Limitations

- GitHub CI runners cannot see institutional shell environment variables, so profiles referencing e.g. env.USER_SCRATCH must define an internal fallback variable or testing fails
- nf-core/configs is moving toward Nextflow 'strict' syntax, so older custom configs may need updating
- No software version numbers are recorded in the pipeline changelog sections, making exact tool-version pinning via config alone a reproducibility gap

## Evidence

- [intro] The pipeline is built using [Nextflow](https://www.nextflow.io/) and processes data using the following steps: "The pipeline is built using [Nextflow](https://www.nextflow.io/) and processes data using the following steps"
- [abstract] Following nf-core guidelines, the pipeline has few dependencies and can be run on any of platforms.: "Following nf-core guidelines, the pipeline has few dependencies and can be run on any of platforms."
- [readme] The Nextflow [`-c`](https://www.nextflow.io/docs/latest/config.html) parameter can be used with nf-core pipelines in order to load custom config files: [evidence span withheld — anchor status: external_doc]
- [readme] simply run `nextflow run <nf-core pipeline> -profile <hpc_name>` and everything is configured for you!: [evidence span withheld — anchor status: external_doc]
- [readme] We recommend using the `nf-core download` helper tool. This will download both the pipeline files and also the config profiles: [evidence span withheld — anchor status: not_found]
