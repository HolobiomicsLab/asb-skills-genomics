---
name: inc-seq-baseline-comparison-analysis
description: Use when reprocessing previously published INC-Seq consensus reads (e.g.,
  R7-chemistry Ladder replicate data from Li et al.) to test whether de novo chopSeq/nanoClust
  consensus calling beats the INC-Seq 97-98% median accuracy baseline.
when_to_use_negative:
- Input is raw noisy nanopore reads rather than INC-Seq consensus reads — chopSeq
  is designed for fragmentation and orientation correction of INC-Seq consensus reads,
  not raw read mapping
- The analysis goal is reference-based mapping of noisy long reads to databases like
  SILVA — the whole rationale here is a completely de novo consensus approach without
  reference mapping
- A quick classification-only assignment suffices — classification-only analysis is
  limited in differentiating closely related sequences and risks false detections
edam_operation: http://edamontology.org/operation_3192
edam_topics:
- http://edamontology.org/topic_0637
- http://edamontology.org/topic_3174
tools:
- name: chopSeq
  role: Performs fragmentation and read orientation correction of INC-Seq consensus
    reads prior to clustering
  repo: github.com/umerijaz/nanopore
- name: nanoClust
  role: Read partitioning-based de novo clustering and within-cluster consensus calling
    to obtain full-length 16S rRNA gene sequences
  repo: github.com/umerijaz/nanopore
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/gigascience/giy140
    title: 'NanoAmpli-Seq: a workflow for amplicon sequencing for mixed microbial
      communities on the nanopore sequencing platform'
  source_content_hash: sha256:2c92e44bacb9acea894b18416ebb3cd0241a4c4460f78eef05e1cf2e69d6dba8
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - chopSeq
  - nanoClust
  edam_operation: http://edamontology.org/operation_3192
  edam_topics:
  - http://edamontology.org/topic_0637
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9285714285714286
n_claims: 14
---

# inc-seq-baseline-comparison-analysis

## Summary

Benchmarks a reprocessed nanopore amplicon dataset (e.g., Li et al. INC-Seq R7 Ladder reads) against the published INC-Seq accuracy baseline by running the chopSeq → nanoClust workflow and comparing per-OTU consensus accuracies to the 97–98% median INC-Seq accuracy.

## When to use

Use when reprocessing previously published INC-Seq consensus reads (e.g., R7-chemistry Ladder replicate data from Li et al.) to test whether de novo chopSeq/nanoClust consensus calling beats the INC-Seq 97-98% median accuracy baseline.

## When NOT to use

- Input is raw noisy nanopore reads rather than INC-Seq consensus reads — chopSeq is designed for fragmentation and orientation correction of INC-Seq consensus reads, not raw read mapping
- The analysis goal is reference-based mapping of noisy long reads to databases like SILVA — the whole rationale here is a completely de novo consensus approach without reference mapping
- A quick classification-only assignment suffices — classification-only analysis is limited in differentiating closely related sequences and risks false detections

## Inputs

- Previously published INC-Seq consensus reads (e.g., Ladder replicate data from Li et al., R7 chemistry)
- Expected OTU reference sequences for the mock community (including the Fusobacterium OTU)

## Outputs

- Full-length 16S rRNA gene consensus sequences partitioned into the theoretical OTUs (8 for the 10-organism community)
- Per-OTU consensus accuracy values from alignment against OTU reference sequences
- Average consensus accuracy versus the INC-Seq 97-98% median accuracy baseline comparison

## How to apply

Obtain the published INC-Seq read data (e.g., the Ladder replicate from Li et al. via the umerijaz/nanopore repository), run chopSeq on the INC-Seq consensus reads for fragmentation and read orientation correction, then run nanoClust for read partitioning-based de novo clustering and within-cluster consensus calling to obtain full-length 16S rRNA gene sequences — for the 10-organism community this partitions the data into its 8 theoretical OTUs. Align each called consensus against the expected OTU reference sequences to compute per-OTU consensus accuracy, including the Fusobacterium OTU. The decision point is the comparison against the INC-Seq baseline: the reproduced run is accepted when average consensus accuracy reaches approximately 99.5% with the Fusobacterium OTU at approximately 98.75%, matching NanoAmpli-Seq 2D/1D2 accuracy on tested mock communities and exceeding the INC-Seq 97–98% median accuracy. This demonstrates the improvement is independent of nanopore sequencing chemistry (works on the older R7 chemistry).

## Related tools

- **chopSeq** (Performs fragmentation and read orientation correction of INC-Seq consensus reads prior to clustering) — github.com/umerijaz/nanopore
- **nanoClust** (Read partitioning-based de novo clustering and within-cluster consensus calling to obtain full-length 16S rRNA gene sequences) — github.com/umerijaz/nanopore

## Evaluation signals

- Average consensus sequence accuracy reaches approximately 99.5% (matching NanoAmpli-Seq 2D/1D2 accuracy on tested mock communities)
- Fusobacterium OTU consensus accuracy is approximately 98.75%
- Reproduced accuracies clearly exceed the INC-Seq 97-98% median accuracy baseline
- The 10-organism community is partitioned into its 8 theoretical OTUs by nanoClust
- Per-OTU accuracies are computed by direct alignment of each consensus against the expected OTU reference sequence

## Limitations

- Nearly all residual errors in NanoAmpli-Seq sequences originate from deletions in homopolymer regions, which caps achievable accuracy
- Result demonstrated on previously published R7-chemistry data; accuracy depends on homopolymer-aware base calling or error correction to reach short-read-platform levels
- Reference-based benchmarking relies on expected OTU references existing for the mock community; reference-based mapping of noisy long reads is limited by incorrect mapping and discarding of reads from organisms absent from the reference database

## Evidence

- [abstract] INC-Seq baseline: "NanoAmpli-Seq accurately estimates the diversity of tested mock communities with average consensus sequence accuracy of 99.5% for 2D and 1D2 sequencing on the nanopore sequencing platform"
- [abstract] workflow definition: "while nanoClust algorithm was designed for read partitioning-based de novo clustering and within cluster consensus calling"
- [intro] motivation for de novo approach: "incorrect mapping to a reference database is likely due to high error rates of raw nanopore reads"
- [abstract] residual error mode: "Nearly all residual errors in NanoAmpli-Seq sequences originate from deletions in homopolymer regions"
- [readme] provenance: "Scripts for processing short amplicon reads from Oxford Nanopore. For details, please see"
