---
name: low-complexity-sequence-filtering
description: Use when raw fasta/fastq sRNA libraries must be reduced to high-confidence
  reads (18–26 nt, abundance ≥5, non-t/rRNA, non-low-complexity) prior to genome mapping
  and miRNA/tasiRNA annotation in the miRPursuit pipeline.
when_to_use_negative:
- Input reads have not been adaptor-trimmed and the --trim flag has not been applied;
  adaptor trimming (fastx_clipper) precedes the Wbench Filter in Step 1
- The goal is genome-based filtering — that is performed later in Step 2 ('Filter
  Genome & mirbase') with patman mapping at 0 mismatches, not by the Wbench Filter
- Sequences already filtered and only being re-analysed downstream from a later pipeline
  stage (miRPursuit allows restarting from a specific stage onward)
edam_operation: http://edamontology.org/operation_3218
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0659
tools:
- name: miRPursuit
  role: Shell-script pipeline that orchestrates the Filtering stage, passing filtered
    fasta libraries between the UEA sRNA workbench Filter and downstream patman/miRProf
    steps
  repo: https://github.com/forestbiotech-lab/miRPursuit
- name: Java
  role: Runtime environment required to execute the UEA sRNA workbench Filter tool
    (miRPursuit works best with Oracle Java v.8)
- name: patman
  role: 'Downstream in the same Filtering stage: reads are further filtered by mapping
    them to the setup genome file with 0 mismatches after length/abundance/complexity
    filtering'
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1002/1873-3468.12746
    title: mi<scp>RP</scp>ursuit—a pipeline for automated analyses of small <scp>RNA</scp>s
      in model and nonmodel plants
  source_content_hash: sha256:39cfbb8d16ad6364eda729685f3eaca2ab1f805e3b80169799806bfc469bfc29
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/forestbiotech-lab/miRPursuit
  tools:
  - Java
  - miRPursuit
  - patman
  edam_operation: http://edamontology.org/operation_3218
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0659
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# low-complexity-sequence-filtering

## Summary

In the miRPursuit Filtering stage, fasta sRNA sequences are filtered for length, abundance and low complexity, and t/rRNA reads are removed, using the UEA sRNA workbench Filter tool configured via wbench_filter.cfg before genome mapping with patman.

## When to use

Use when raw fasta/fastq sRNA libraries must be reduced to high-confidence reads (18–26 nt, abundance ≥5, non-t/rRNA, non-low-complexity) prior to genome mapping and miRNA/tasiRNA annotation in the miRPursuit pipeline.

## When NOT to use

- Input reads have not been adaptor-trimmed and the --trim flag has not been applied; adaptor trimming (fastx_clipper) precedes the Wbench Filter in Step 1
- The goal is genome-based filtering — that is performed later in Step 2 ('Filter Genome & mirbase') with patman mapping at 0 mismatches, not by the Wbench Filter
- Sequences already filtered and only being re-analysed downstream from a later pipeline stage (miRPursuit allows restarting from a specific stage onward)

## Inputs

- fasta sRNA library files (e.g. test_dataset-1.fa, test_dataset-2.fa) sharing a common string with sequential numbering, located in INSERTS_DIR
- wbench_filter.cfg configuration file with filtering parameters
- workdirs.cfg setting FILTER_SUF filter suffix (e.g. 18_26_5)

## Outputs

- filtered fasta sRNA libraries restricted to 18–26 nt reads with abundance ≥5, t/rRNA and low-complexity reads removed
- per-library kept-sequence count table (e.g. filter_kept_counts.tsv)
- filter discard log under [workdir]/logs and read statistics under [workdir]/counts

## How to apply

Load the input fasta sRNA libraries (e.g. test_dataset-1.fa, test_dataset-2.fa) from INSERTS_DIR, which share a common filename string with sequential numbering. Apply the UEA sRNA workbench Filter tool to each library using the broad-range defaults in wbench_filter.cfg: min_length=18, max_length=26, min_abundance=5, with filter_low_comp=true and filter_invalid=true. Enable t/rRNA removal with trrna=true and trrna_sense_only=false, while keeping genome-hit, normalized-abundance and kill-list filtering disabled (filter_genome_hits=false, filter_norm_abund=false, filter_kill_list=false) since genome filtering happens in the later patman-based 'Filter Genome & mirbase' step. Write filtered fasta libraries to the working directory, matching the FILTER_SUF=18_26_5 suffix convention chosen in config/workbench_filter_*.cfg. Tabulate per-library kept counts (e.g. filter_kept_counts.tsv) and cross-check against the filter discard log.

## Related tools

- **miRPursuit** (Shell-script pipeline that orchestrates the Filtering stage, passing filtered fasta libraries between the UEA sRNA workbench Filter and downstream patman/miRProf steps) — https://github.com/forestbiotech-lab/miRPursuit
- **Java** (Runtime environment required to execute the UEA sRNA workbench Filter tool (miRPursuit works best with Oracle Java v.8))
- **patman** (Downstream in the same Filtering stage: reads are further filtered by mapping them to the setup genome file with 0 mismatches after length/abundance/complexity filtering)

## Examples

```
bash pathToMirPursuit/miRPursuit.sh -f 1 -l 2 --fasta test_dataset-
```

## Evaluation signals

- Each filtered library contains only sequences within the 18–26 nt length range
- All retained reads have abundance ≥5 (min_abundance=5)
- t/rRNA reads (trrna=true, sense and antisense since trrna_sense_only=false) and low-complexity/invalid reads are absent from the output
- Per-library kept counts are consistent with the filter discard log and with counts written to [workdir]/counts
- Output filenames follow the FILTER_SUF=18_26_5 suffix convention from config/workbench_filter_*.cfg

## Limitations

- Parameters are pipeline defaults ('broad range default values'); non-model or unusual libraries may need re-tuning of min_length/max_length/min_abundance in wbench_filter.cfg
- Quality control with fastqc is not yet implemented in this stage, so raw-read quality issues are not assessed before filtering
- The t/rRNA filter relies on the UEA workbench's internal t/rRNA detection; no separate genome-hit or kill-list filtering is applied at this step (filter_genome_hits=false, filter_norm_abund=false, filter_kill_list=false)

## Evidence

- [intro] Filtering of fasta sequences based on length, abundance, low complexity, and removal of t/r RNA: "The fasta sequences are filtered based on their length, abundance, low complexity and t/r RNA are removed."
- [methods] Broad-range default filter parameters: "#Broad range default values
      min_length=18
      max_length=26
      min_abundance=5"
- [methods] trrna filter settings: "trrna=true
      trrna_sense_only=false"
- [methods] Disabled genome/norm/kill-list filters: "filter_genome_hits=false
      filter_norm_abund=false
      filter_kill_list=false"
- [intro] Reads further filtered by genome mapping with patman: "The reads are further filtered by mapping them to the setup genome file with '0' mismatches using patman."
- [readme] Wbench Filter is Step 1 after adaptor trimming: [evidence span withheld — anchor status: external_doc]
