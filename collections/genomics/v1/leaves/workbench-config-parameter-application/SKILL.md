---
name: workbench-config-parameter-application
description: Use when running miRPursuit on sRNA fasta/fastq libraries and you must
  set or adjust workbench filtering parameters (min_length=18, max_length=26, min_abundance=5,
  trrna=true) before genome mapping.
when_to_use_negative:
- Reads should skip filtering and enter at a later stage (e.g. miRCat or PAREsnip)
  — jump steps with -s|--step instead of altering filter parameters
- You need genome-hit, normalized-abundance or kill-list filtering at this stage —
  these are disabled by default (filter_genome_hits=false, filter_norm_abund=false,
  filter_kill_list=false) and belong to the later 'Filter Genome & miRBase' step
- The task is tool-specific parameter tuning outside the workflow (e.g. miRCat or
  PAREsnip settings in wbench_mircat.cfg or wbench_paresnip.cfg) — this skill covers
  the Filter/wbench_filter.cfg parameter set
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: miRPursuit
  role: Shell-script pipeline whose config directory centralises all workbench tool
    parameters; FILTER_SUF in workdirs.cfg selects the predefined filter settings
  repo: https://github.com/forestbiotech-lab/miRPursuit
- name: UEA sRNA workbench Filter
  role: Tool applied at the Filtering stage whose parameters (length, abundance, low-complexity,
    t/rRNA) are stored in wbench_filter.cfg
- name: Java 8
  role: Runtime required by the workbench tools whose parameters are configured; memory
    set via the MEMORY variable
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1002/1873-3468.12746
    title: mi<scp>RP</scp>ursuit—a pipeline for automated analyses of small <scp>RNA</scp>s
      in model and nonmodel plants
  source_content_hash: sha256:a7e33c898d5809370aff674acc9806bef83c00493a1b81a2825d644e64cc475e
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/forestbiotech-lab/miRPursuit
  tools:
  - Java
  - Java 8
  - UEA sRNA workbench Filter
  - miRPursuit
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# workbench-config-parameter-application

## Summary

Applies UEA sRNA workbench tool parameters (notably the Wbench Filter settings for sRNA length, abundance, low-complexity and t/rRNA removal) through miRPursuit's config/*.cfg files, so that filtering behaviour is defined once and reused across all libraries.

## When to use

Use when running miRPursuit on sRNA fasta/fastq libraries and you must set or adjust workbench filtering parameters (min_length=18, max_length=26, min_abundance=5, trrna=true) before genome mapping.

## When NOT to use

- Reads should skip filtering and enter at a later stage (e.g. miRCat or PAREsnip) — jump steps with -s|--step instead of altering filter parameters
- You need genome-hit, normalized-abundance or kill-list filtering at this stage — these are disabled by default (filter_genome_hits=false, filter_norm_abund=false, filter_kill_list=false) and belong to the later 'Filter Genome & miRBase' step
- The task is tool-specific parameter tuning outside the workflow (e.g. miRCat or PAREsnip settings in wbench_mircat.cfg or wbench_paresnip.cfg) — this skill covers the Filter/wbench_filter.cfg parameter set

## Inputs

- fasta sRNA library files (e.g. test_dataset-1.fa, test_dataset-2.fa) sharing a common name stem with sequential numbering, pointed to via INSERTS_DIR
- fastq or fastq.gz raw library files (when starting the pipeline before filtering)
- config directory files: wbench_filter.cfg, workdirs.cfg (FILTER_SUF, INSERTS_DIR, workdir, ADAPTOR), software_dirs.cfg

## Outputs

- Filtered fasta sRNA libraries restricted to 18–26 nt, abundance >=5, low-complexity and t/rRNA reads removed
- Read-count statistics in the counts directory (space-separated .tsv-style count files)
- Execution logs under [workdir]/logs recording each filtering run

## How to apply

Edit the configuration files in miRPursuit's config directory rather than the scripts themselves: wbench_filter.cfg holds the broad-range defaults (min_length=18, max_length=26, min_abundance=5) with filter_low_comp=true and filter_invalid=true enabled, plus trrna=true and trrna_sense_only=false to remove t/rRNA in both orientations; filter_genome_hits, filter_norm_abund and filter_kill_list are left false at this stage. In workdirs.cfg, set FILTER_SUF to the filter suffix that selects the predefined parameter set (e.g. 18_26_5) and INSERTS_DIR to the library input directory. Because miRPursuit is a series of UNIX shell scripts connecting workbench tools, parameters set here propagate automatically to every library processed in the run; re-analysis with different parameters can restart from the Filtering stage via the -s|--step option of sRNAWorkflow.sh.

## Related tools

- **miRPursuit** (Shell-script pipeline whose config directory centralises all workbench tool parameters; FILTER_SUF in workdirs.cfg selects the predefined filter settings) — https://github.com/forestbiotech-lab/miRPursuit
- **UEA sRNA workbench Filter** (Tool applied at the Filtering stage whose parameters (length, abundance, low-complexity, t/rRNA) are stored in wbench_filter.cfg)
- **Java 8** (Runtime required by the workbench tools whose parameters are configured; memory set via the MEMORY variable)

## Examples

```
bash pathToMirPursuit/miRPursuit.sh -f 1 -l 2 --fasta test_dataset-
```

## Evaluation signals

- Every output fasta sequence is within the configured 18–26 nt window and has abundance >= min_abundance=5
- t/rRNA and low-complexity reads are absent from filtered libraries (trrna=true, filter_low_comp=true took effect)
- FILTER_SUF suffix (e.g. 18_26_5) matches the values actually written in wbench_filter.cfg
- Per-library kept-read counts in the counts directory are consistent with the filter discard log under [workdir]/logs
- Re-running the pipeline with the same config reproduces identical filtered outputs (config, not script, is the single source of parameters)

## Limitations

- Only one parameter set is active per run; switching settings requires editing wbench_filter.cfg and restarting from the Filtering stage (-s 1)
- t/rRNA filtering uses trrna_sense_only=false by default, so both sense and antisense t/rRNA are removed — verify this matches the analysis intent
- The README notes the count output 'is not really a tsv it is in fact a space separated values', so downstream parsers must not assume strict tab separation

## Evidence

- [readme] FILTER_SUF selects predefined filter settings: [evidence span withheld — anchor status: external_doc]
