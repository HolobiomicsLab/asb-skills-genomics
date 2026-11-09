---
name: sequence-length-abundance-thresholding
description: Use when preprocessing raw sRNA-seq (FASTQ/FASTA) libraries for small
  RNA discovery and you need to discard reads outside the miRNA/siRNA size window
  or below a minimum count threshold.
when_to_use_negative:
- Input is already a filtered FASTA set past the Wbench Filter step — instead restart
  the pipeline with -s|--step 2 (Filter Genome & miRBase) to avoid re-filtering
- The read type falls outside the 18–26 nt window (e.g. piRNA- or rRNA-focused analyses)
  — the hard-coded sRNA size defaults would discard the targets
- Quality control is the goal — miRPursuit explicitly notes fastqc-based QC is not
  yet implemented
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_0092
- http://edamontology.org/topic_0080
tools:
- name: miRPursuit
  role: Pipeline orchestrating the Wbench Filter step (Step 1) and storing read counts
    and logs per library
  repo: https://github.com/forestbiotech-lab/miRPursuit
- name: UEA sRNA workbench
  role: Provides the Wbench Filter tool implementing the length/abundance/complexity/trrna
    thresholds
  repo: https://github.com/forestbiotech-lab/sRNA-workflow
- name: FASTX-Toolkit (fastx_clipper)
  role: Performs adaptor trimming immediately before Wbench Filter when the --trim
    flag is set
  repo: http://hannonlab.cshl.edu/fastx_toolkit/
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1002/1873-3468.12746
    title: mi<scp>RP</scp>ursuit—a pipeline for automated analyses of small <scp>RNA</scp>s
      in model and nonmodel plants
  source_content_hash: sha256:aa27a64a508153d406d4626df6d0926d3d1e88dbf910bbe8f2d2cb2d6a576723
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/forestbiotech-lab/miRPursuit
  tools:
  - FASTX-Toolkit (fastx_clipper)
  - UEA sRNA workbench
  - miRProf
  - miRPursuit
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_0092
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 12
---

# sequence-length-abundance-thresholding

## Summary

A filtering step from miRPursuit's Wbench Filter stage that reduces raw sRNA FASTA reads by length (18–26 nt), abundance (min_abundance=5), low complexity, invalid reads, and t/rRNA contamination before genome mapping and miRNA identification.

## When to use

Use when preprocessing raw sRNA-seq (FASTQ/FASTA) libraries for small RNA discovery and you need to discard reads outside the miRNA/siRNA size window or below a minimum count threshold.

## When NOT to use

- Input is already a filtered FASTA set past the Wbench Filter step — instead restart the pipeline with -s|--step 2 (Filter Genome & miRBase) to avoid re-filtering
- The read type falls outside the 18–26 nt window (e.g. piRNA- or rRNA-focused analyses) — the hard-coded sRNA size defaults would discard the targets
- Quality control is the goal — miRPursuit explicitly notes fastqc-based QC is not yet implemented

## Inputs

- Raw sRNA-seq FASTQ files (optionally .fastq.gz archives matching a library-series pattern, e.g. Lib_1.fq)
- Raw sRNA-seq FASTA files (e.g. Lib_1.fa series)
- Adaptor sequence defined as ADAPTOR in config/workdirs.cfg

## Outputs

- Filtered FASTA sequences retained within the 18–26 nt length window at min_abundance=5, cleaned of low-complexity and t/rRNA reads
- Per-step read counts stored in the [workdir]/counts directory (space-separated %y%m%d:%h%m%s-type-lib[lib_first]-[lib_last].tsv files)
- Execution logs in [workdir]/logs (*.log / *.log.ok)

## How to apply

Run adaptor trimming first (fastx_clipper from FASTX-Toolkit, if flagged, with the ADAPTOR sequence set in workdirs.cfg), then apply the UEA sRNA workbench Wbench Filter as Step 1 of the miRPursuit workflow. Use the broad-range defaults from the filter config: min_length=18, max_length=26, min_abundance=5 to keep reads in the canonical sRNA size window with sufficient counts. Enable trrna=true to remove t/rRNA fragments and filter_low_comp=true with filter_invalid=true to drop low-complexity and invalid sequences, while leaving filter_genome_hits=false so genome filtering happens downstream via PatMan. Keep FILTER_SUF in workdirs.cfg pointing at the predefined filter settings so the Filtering stage can be re-run with different parameters independently of later Annotation and Reporting stages.

## Related tools

- **miRPursuit** (Pipeline orchestrating the Wbench Filter step (Step 1) and storing read counts and logs per library) — https://github.com/forestbiotech-lab/miRPursuit
- **UEA sRNA workbench** (Provides the Wbench Filter tool implementing the length/abundance/complexity/trrna thresholds) — https://github.com/forestbiotech-lab/sRNA-workflow
- **FASTX-Toolkit (fastx_clipper)** (Performs adaptor trimming immediately before Wbench Filter when the --trim flag is set) — http://hannonlab.cshl.edu/fastx_toolkit/

## Examples

```
bash pathToMirPursuit/miRPursuit.sh -f 1 -l 2 --fasta test_dataset-
```

## Evaluation signals

- Every retained read is within the 18–26 nt window and has abundance >= min_abundance=5 in the filtered FASTA
- Reads matching t/rRNA are absent from the filtered output (trrna=true) and low-complexity/invalid sequences are removed (filter_low_comp=true, filter_invalid=true)
- The count file in [workdir]/counts shows a monotonic decrease from raw to filtered read totals per library, consistent with filtering only
- The run log in [workdir]/logs ends in *.log.ok indicating the Filtering step completed without error
- Non-conserved reads produced downstream remain valid input for miRCat and the ta-si predictor, confirming the filter did not over-trim

## Limitations

- min_length/max_length/min_abundance are 'Broad range default values' (18–26 nt, abundance 5); analyses targeting sRNA classes outside this window need config changes
- Quality control with fastqc is not yet implemented in the workflow
- The count file is not really a tsv — it is in fact space separated values, which may break strict TSV parsers

## Evidence
