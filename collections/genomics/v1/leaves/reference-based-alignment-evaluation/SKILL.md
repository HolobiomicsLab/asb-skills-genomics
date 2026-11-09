---
name: reference-based-alignment-evaluation
description: 'Use when comparing overlap detection tools'' output files on datasets
  with a reference genome available, and you need objective graph-accuracy metrics
  (error rate, #COC, #CC) evaluated against filtered reference alignments.'
when_to_use_negative:
- No reference assembly exists for the organism, since error rate is computed against
  reference-based ground-truth alignments
- The goal is assembly quality rather than overlap graph quality — BUSCO/contiguity
  evaluation of the Fedrann-Shasta assemblies is a separate step
- Overlaps are already provided as alignment coordinates (e.g. PAF with identity already
  thresholded above 30%), where re-filtering with the 10% identity rule would change
  established baselines
edam_topics:
- http://edamontology.org/topic_0102
tools:
- name: Fedrann
  role: Produces the overlaps.tsv candidate overlap files being evaluated against
    ground truth
  repo: https://github.com/jzhang-dev/FEDRANN
- name: minimap2
  role: Generates reference-based PAF alignments used as ground truth (identity threshold
    raised to 30% for human PAFs)
- name: kNN-overlap-finder workflows
  role: Reproducible benchmarking and evaluation workflows that run the six baselines
    and score every overlap set
  repo: https://github.com/jzhang-dev/kNN-overlap-finder
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/gigascience/giag048
    title: 'FEDRANN: effective long-read overlap detection based on dimensionality
      reduction and approximate nearest neighbors'
  source_content_hash: sha256:0b3372bed747934c332438b64505427720fbf11977d323b8a09f73df9d0500c5
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/jzhang-dev/FEDRANN
  tools:
  - BLEND
  - Fedrann
  - MECAT2
  - MHAP
  - Python
  - kNN-overlap-finder workflows
  - minimap2
  - wtdbg2
  - xRead
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 18
---

# reference-based-alignment-evaluation

## Summary

Scores long-read overlap detection outputs (e.g. Fedrann overlaps.tsv, minimap2 PAF, MHAP files) against reference-based ground-truth alignments to compute overlap graph accuracy metrics — error rate, mean overlap size, #COC, #CC — plus runtime and peak memory per tool-dataset pair.

## When to use

Use when comparing overlap detection tools' output files on datasets with a reference genome available, and you need objective graph-accuracy metrics (error rate, #COC, #CC) evaluated against filtered reference alignments.

## When NOT to use

- No reference assembly exists for the organism, since error rate is computed against reference-based ground-truth alignments
- The goal is assembly quality rather than overlap graph quality — BUSCO/contiguity evaluation of the Fedrann-Shasta assemblies is a separate step
- Overlaps are already provided as alignment coordinates (e.g. PAF with identity already thresholded above 30%), where re-filtering with the 10% identity rule would change established baselines

## Inputs

- Overlap output files per tool (e.g. Fedrann overlaps.tsv with query_name, query_orientation, target_name, target_orientation, neighbor_rank, distance; minimap2 PAF; MHAP format)
- Reference genome assemblies (FASTA): WBcel235 GCA_000002985.3, Drosophila Release 6 GCF_000001215.4, T2T-CHM13 v2.0
- Long-read FASTQ/FASTA datasets (ONT SRR10028111, SRR13070625; PacBio HiFi and CycloneSEQ G400-ER HG002)

## Outputs

- Per tool-dataset metrics: error rate, mean overlap size, #COC, #CC, runtime, peak memory
- Aggregated CSV table with one row per tool-dataset combination
- Filtered ground-truth reference alignment sets

## How to apply

Align each long-read dataset to its reference assembly (e.g. WBcel235 GCA_000002985.3, Drosophila Release 6 GCF_000001215.4, T2T-CHM13 v2.0) to build ground truth, filtering reference reads by length ≥ 5 kb, aligned fraction ≥ 50%, and mapping quality ≥ 30. Filter the alignments themselves to ≥ 100 matched bases and ≥ 10% alignment identity; for minimap2 PAF files producing exceptionally large outputs (> 4 TB on human whole-genome reads), raise the identity threshold to 30%. Convert each tool's overlap output (Fedrann overlaps.tsv with query/target names, orientations, neighbor_rank, distance) into the evaluation format, then compute error rate, mean overlap size, number of connected components of the overlap graph (#COC and #CC) per tool-dataset pair. Record wall-clock runtime and peak memory during the tool runs so accuracy and efficiency can be compared in one aggregated CSV with one row per tool-dataset combination. Reproducible evaluation workflows live in github.com/jzhang-dev/kNN-overlap-finder.

## Related tools

- **Fedrann** (Produces the overlaps.tsv candidate overlap files being evaluated against ground truth) — https://github.com/jzhang-dev/FEDRANN
- **minimap2** (Generates reference-based PAF alignments used as ground truth (identity threshold raised to 30% for human PAFs))
- **kNN-overlap-finder workflows** (Reproducible benchmarking and evaluation workflows that run the six baselines and score every overlap set) — https://github.com/jzhang-dev/kNN-overlap-finder

## Evaluation signals

- Metrics table contains all required fields (error rate, mean overlap size, #COC, #CC, runtime, peak memory) for all 7 tools x 5 datasets
- Filtered ground-truth alignments satisfy all thresholds: read length ≥ 5 kb, aligned fraction ≥ 50%, mapping quality ≥ 30, ≥ 100 matched bases, ≥ 10% identity
- Reproduced result matches the paper's finding: Fedrann most accurate on 4 of 5 datasets, with MECAT2 slightly better on H4C
- Fedrann shows competitive runtime (fastest on H4C/H4O) while its memory consumption is expected to be higher than most existing tools

## Limitations

- Evaluation requires reference assemblies; ground truth is only as good as the reference alignment filters
- Human PAF files exceeded 4 TB, forcing a raised (30%) identity threshold for minimap2 that differs from other baselines
- Reproducibility depends solely on GitHub repositories; no DOI or Zenodo archive is provided

## Evidence

- [discussion] Read length >= 5 kb, aligned fraction >= 50%, mapping quality >= 30: [evidence span withheld — anchor status: not_found]
- [discussion] Alignment filtering criteria: matched bases and identity: [evidence span withheld — anchor status: not_found]
- [discussion] minimap2 PAF size threshold exception: "for minimap2 PAF files (which produced exceptionally large outputs exceeding 4TB for human whole-genome sequencing read alignments), we increased the identity threshold to 30%"
- [readme] Benchmark workflows reproducibility: [evidence span withheld — anchor status: external_doc]
- [discussion] Memory caveat for Fedrann: [evidence span withheld — anchor status: not_found]
