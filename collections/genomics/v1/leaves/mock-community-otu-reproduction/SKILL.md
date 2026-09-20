---
name: mock-community-otu-reproduction
description: Use when reprocessing previously published nanopore amplicon mock-community
  reads (e.g., INC-Seq consensus reads from older R7 chemistry) to reproduce de novo
  OTU-level consensus sequences and validate accuracy against published baselines.
when_to_use_negative:
- Input is raw nanopore read data not previously processed with INC-Seq; the workflow
  expects INC-Seq consensus reads as chopSeq input
- The community composition is unknown and no reference sequences exist for computing
  per-OTU accuracy against expected OTUs
- The goal is reference-database-based classification rather than de novo clustering
  and consensus calling
edam_operation: http://edamontology.org/operation_0289
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
    to obtain full-length 16S rRNA gene consensus sequences
  repo: github.com/umerijaz/nanopore
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/gigascience/giy140
    title: 'NanoAmpli-Seq: a workflow for amplicon sequencing for mixed microbial
      communities on the nanopore sequencing platform'
  source_content_hash: sha256:864804c971dfe18af7f94f25c5fb867742eb37d81a11a89f1b763078adfd67f6
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - chopSeq
  - nanoClust
  edam_operation: http://edamontology.org/operation_0289
  edam_topics:
  - http://edamontology.org/topic_0637
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# mock-community-otU-reproduction

## Summary

Reproduces the NanoAmpli-Seq reprocessing of previously published INC-Seq mock-community data (e.g., Li et al. R7 Ladder replicates) by running chopSeq fragmentation/orientation correction followed by nanoClust de novo clustering and consensus calling, then benchmarking per-OTU consensus accuracy (~99.5% average).

## When to use

Use when reprocessing previously published nanopore amplicon mock-community reads (e.g., INC-Seq consensus reads from older R7 chemistry) to reproduce de novo OTU-level consensus sequences and validate accuracy against published baselines.

## When NOT to use

- Input is raw nanopore read data not previously processed with INC-Seq; the workflow expects INC-Seq consensus reads as chopSeq input
- The community composition is unknown and no reference sequences exist for computing per-OTU accuracy against expected OTUs
- The goal is reference-database-based classification rather than de novo clustering and consensus calling

## Inputs

- INC-Seq consensus reads from a previously published mock-community amplicon sequencing study (e.g., Li et al. R7 Ladder replicate data)
- Expected OTU reference sequences for the known mock community composition (e.g., 10-organism community, 8 OTUs)

## Outputs

- Full-length 16S rRNA gene consensus sequences per OTU
- Per-OTU consensus sequence accuracy values from alignment against reference sequences
- Average consensus accuracy metric (~99.5% expected) for benchmark comparison

## How to apply

Obtain the original INC-Seq reads (e.g., the Ladder replicate data from Li et al. via github.com/umerijaz/nanopore) and run chopSeq on the INC-Seq consensus reads for fragmentation and read orientation correction. Next, run nanoClust for read partitioning-based de novo clustering and within-cluster consensus calling to obtain full-length 16S rRNA gene sequences, partitioning a 10-organism community into its 8 theoretical OTUs. Align each called consensus against the expected OTU reference sequences to compute per-OTU accuracy, including challenging OTUs such as Fusobacterium. Compare reproduced accuracies against the INC-Seq 97-98% median baseline; the run is validated when the average accuracy reaches approximately 99.5% with Fusobacterium at approximately 98.75%, matching NanoAmpli-Seq results on 2D/1D2 mock communities. The approach is fully de novo, requiring no reference-based mapping of noisy raw reads, which the article argues is compromised by high nanopore error rates.

## Related tools

- **chopSeq** (Performs fragmentation and read orientation correction of INC-Seq consensus reads prior to clustering) — github.com/umerijaz/nanopore
- **nanoClust** (Read partitioning-based de novo clustering and within-cluster consensus calling to obtain full-length 16S rRNA gene consensus sequences) — github.com/umerijaz/nanopore

## Evaluation signals

- Average consensus sequence accuracy across OTUs reaches approximately 99.5%
- Fusobacterium OTU consensus accuracy is approximately 98.75%
- The 10-organism community is partitioned into its 8 theoretical OTUs
- Reproduced accuracies exceed the INC-Seq 97-98% median accuracy baseline
- Residual errors are predominantly deletions in homopolymer regions, consistent with reported error profiles

## Limitations

- Nearly all residual errors in NanoAmpli-Seq sequences originate from deletions in homopolymer regions, limiting accuracy on homopolymer-rich sequences
- Reproducing results on older nanopore chemistry (R7) may yield accuracy matching 2D/1D2 mock-community results but does not establish performance on current chemistries

## Evidence

- [abstract] abstract: "The developed protocol includes chopSeq software for fragmentation and read orientation correction of INC-Seq consensus reads"
- [readme] readme: "Scripts for processing short amplicon reads from Oxford Nanopore."
- [intro] intro: "The more robust alternative towards high accuracy consensus sequence generation would be a completely de novo approach, i.e., generation of a consensus sequence without the use of any reference"
