---
name: reference-genome-read-extraction
description: Use when building a simulated long-read benchmark where ground-truth
  overlaps must come from aligning reads back to a known reference chromosome with
  defined accuracy, coverage, and read-length grid conditions.
when_to_use_negative:
- The dataset consists of real sequencing reads where ground-truth overlaps cannot
  be derived by aligning to a known reference (use real-data benchmarks like the GIAB
  HG002 PacBio HiFi or ONT SRR datasets instead).
- The analysis goal is assembly quality assessment (BUSCO/contiguity on Shasta assemblies)
  rather than overlap graph error rate benchmarking.
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_0196
tools:
- name: PBSIM3
  role: Simulates long reads from the extracted reference chromosome across the read-length/coverage/accuracy
    parameter grid
- name: minimap2
  role: Aligns simulated reads back to the extracted reference chromosome to generate
    ground-truth overlaps under the paper's alignment filters
- name: Fedrann
  role: Produces candidate overlaps.tsv per simulated dataset for comparison against
    the reference-derived ground truth
  repo: https://github.com/jzhang-dev/FEDRANN
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/gigascience/giag048
    title: 'FEDRANN: effective long-read overlap detection based on dimensionality
      reduction and approximate nearest neighbors'
  source_content_hash: sha256:f248804168973114074c4a8314f83980fe862e9b1656c0d06be687048da3508a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/jzhang-dev/FEDRANN
  tools:
  - Fedrann
  - PBSIM3
  - Python
  - minimap2
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_0196
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# reference-genome-read-extraction

## Summary

Extracts a reference sequence (e.g., chromosome 22 from the human T2T-CHM13 v2.0 assembly) to serve as ground truth for PBSIM3 long-read simulation, enabling controlled benchmarking of overlap detection against known alignment-derived overlaps.

## When to use

Use when building a simulated long-read benchmark where ground-truth overlaps must come from aligning reads back to a known reference chromosome with defined accuracy, coverage, and read-length grid conditions.

## When NOT to use

- The dataset consists of real sequencing reads where ground-truth overlaps cannot be derived by aligning to a known reference (use real-data benchmarks like the GIAB HG002 PacBio HiFi or ONT SRR datasets instead).
- The analysis goal is assembly quality assessment (BUSCO/contiguity on Shasta assemblies) rather than overlap graph error rate benchmarking.

## Inputs

- Reference genome assembly in FASTA (T2T-CHM13 v2.0 human assembly)
- Simulation parameter grid specification (read length, coverage, accuracy)
- Simulated long-read FASTQ files from PBSIM3
- minimap2 alignment output (SAM/PAF) of simulated reads against the extracted reference

## Outputs

- Per-chromosome reference FASTA for simulation
- One simulated FASTQ per grid condition
- Ground-truth overlap set from filtered minimap2 alignments
- Per-condition overlap error rates (aggregated, e.g., overlap_error_rates.csv)

## How to apply

First extract the target chromosome (here chromosome 22) from the T2T-CHM13 v2.0 assembly to serve as the simulation reference. Then use PBSIM3 to simulate long reads across a parameter grid — read lengths of 10, 20, and 30 kb; coverages of 10, 30, and 50x; accuracies of 91%, 95%, and 99% — producing one FASTQ per grid condition. Generate ground truth by aligning the simulated reads back to the extracted chromosome with minimap2 and applying the paper's filters: >= 100 matched bases and >= 10% alignment identity, plus reference-read filters of length >= 5 kb, aligned fraction >= 50%, and mapping quality >= 30. This reference-backed ground truth is what allows downstream comparison against a tool's overlaps.tsv to compute false-positive/false-negative overlap graph error rates per condition.

## Related tools

- **PBSIM3** (Simulates long reads from the extracted reference chromosome across the read-length/coverage/accuracy parameter grid)
- **minimap2** (Aligns simulated reads back to the extracted reference chromosome to generate ground-truth overlaps under the paper's alignment filters)
- **Fedrann** (Produces candidate overlaps.tsv per simulated dataset for comparison against the reference-derived ground truth) — https://github.com/jzhang-dev/FEDRANN

## Evaluation signals

- Each simulated FASTQ corresponds to exactly one grid condition (read length 10/20/30 kb; coverage 10/30/50x; accuracy 91%/95%/99%).
- Ground-truth alignments pass the filters: >= 100 matched bases, >= 10% alignment identity; reference reads retained only if length >= 5 kb, aligned fraction >= 50%, mapping quality >= 30.
- Error rates fall monotonically (or near-monotonically) with rising simulated accuracy — the reported pattern is 8.34% at 91% accuracy to 5.55% at 99% accuracy at 30x coverage / 20 kb mean read length.
- Per-condition error rates aggregate cleanly into overlap_error_rates.csv with one row per grid condition.

## Limitations

- Ground-truth quality depends on minimap2 alignment filters; for human whole-genome PAF outputs exceeding 4 TB the identity threshold must be raised (to 30% in the paper).
- Simulated data from a single reference chromosome may not capture real-data complexity such as repeats and structural variation present in empirical ONT/PacBio datasets.
- No DOI or Zenodo archive is provided; reproducibility depends solely on the GitHub repositories (kNN-overlap-finder workflows).

## Evidence

- [other] 1. Extract chromosome 22 from the human T2T-CHM13 v2.0 assembly to serve as the simulation reference.: [evidence span withheld — anchor status: not_found]
- [other] Simulate long reads with PBSIM3 for each point of the parameter grid — read lengths 10, 20, and 30 kb; coverages 10, 30, and 50x; accuracies 91%, 95%, and 99%: [evidence span withheld — anchor status: not_found]
- [discussion] All alignments were filtered using two criteria: (1) ≥ 100 matched bases and (2) ≥ 10% alignment identity: [evidence span withheld — anchor status: not_found]
- [readme] PBSIM3-based simulation: Varied depth, accuracy, and read length configurations: [evidence span withheld — anchor status: external_doc]
