---
name: 16s-amplicon-consensus-calling
description: Use when raw, noisy MinION 16S rRNA amplicon reads (2D or 1D2) must be
  clustered de novo into OTUs and accurate full-length consensus sequences called
  without reference-based mapping.
when_to_use_negative:
- Reads can be confidently mapped to a complete reference database (e.g., SILVA) and
  reference-based clustering is acceptable — the de novo route here is specifically
  motivated by incorrect mapping and discarded reads for organisms absent from the
  reference
- Only taxonomy classification is needed and closely related sequences need not be
  differentiated — classification-only approaches risk false detections but suffice
  for coarse profiling
- Input is already consensus-quality short-read (e.g., Illumina MiSeq) data with low
  error rates; the homopolymer-deletion error profile addressed here is nanopore-specific
edam_operation: http://edamontology.org/operation_0291
edam_topics:
- http://edamontology.org/topic_0080
- http://edamontology.org/topic_0637
tools:
- name: nanoClust
  role: Read partitioning-based de novo clustering and within-cluster consensus calling
    to obtain accurate full-length 16S rRNA gene sequences
  repo: github.com/umerijaz/nanopore
- name: chopSeq
  role: Fragmentation and read orientation correction of INC-Seq consensus reads prior
    to nanoClust clustering
  repo: github.com/umerijaz/nanopore
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/gigascience/giy140
    title: 'NanoAmpli-Seq: a workflow for amplicon sequencing for mixed microbial
      communities on the nanopore sequencing platform'
  source_content_hash: sha256:f8894d5983dbcb35bf748ae42e5249cbf8cff7a016204130312660087aae4f1b
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - chopSeq
  - nanoClust
  edam_operation: http://edamontology.org/operation_0291
  edam_topics:
  - http://edamontology.org/topic_0080
  - http://edamontology.org/topic_0637
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# 16s-amplicon-consensus-calling

## Summary

De novo generation of full-length 16S rRNA gene consensus sequences from noisy Oxford Nanopore amplicon reads (2D/1D2 MinION), using nanoClust read partitioning-based clustering and within-cluster consensus calling after chopSeq fragmentation and orientation correction, achieving ~99.5% accuracy without reference databases.

## When to use

Use when raw, noisy MinION 16S rRNA amplicon reads (2D or 1D2) must be clustered de novo into OTUs and accurate full-length consensus sequences called without reference-based mapping.

## When NOT to use

- Reads can be confidently mapped to a complete reference database (e.g., SILVA) and reference-based clustering is acceptable — the de novo route here is specifically motivated by incorrect mapping and discarded reads for organisms absent from the reference
- Only taxonomy classification is needed and closely related sequences need not be differentiated — classification-only approaches risk false detections but suffice for coarse profiling
- Input is already consensus-quality short-read (e.g., Illumina MiSeq) data with low error rates; the homopolymer-deletion error profile addressed here is nanopore-specific

## Inputs

- INC-Seq consensus amplicon reads from Oxford Nanopore MinION (2D or 1D2 sequencing mode), from mixed microbial communities / mock communities
- chopSeq-processed fragmented, orientation-corrected short amplicon reads (nanopore fastq)

## Outputs

- Full-length 16S rRNA gene consensus sequences, one per de novo cluster (OTU)
- OTU counts per partition setting (e.g., tabulated for partition counts 1-7, as in partition_sweep_results.csv)
- Per-consensus sequence accuracy values relative to reference 16S sequences

## How to apply

First process INC-Seq consensus reads with chopSeq for fragmentation and read orientation correction. Then run the nanoClust algorithm, which partitions reads de novo (e.g., sweeping partition counts 1-7; low partition counts are optimal) and calls within-cluster consensus full-length 16S rRNA gene sequences. Evaluate each consensus by aligning it to mock community reference 16S sequences to compute accuracy, and count clusters as OTUs. Rationale: de novo clustering avoids the incorrect mapping and read discarding that plague reference-based clustering of noisy nanopore reads. Expect ~99.5% average consensus accuracy for 2D and 1D2 sequencing on mock communities; residual errors are almost entirely homopolymer-region deletions.

## Related tools

- **nanoClust** (Read partitioning-based de novo clustering and within-cluster consensus calling to obtain accurate full-length 16S rRNA gene sequences) — github.com/umerijaz/nanopore
- **chopSeq** (Fragmentation and read orientation correction of INC-Seq consensus reads prior to nanoClust clustering) — github.com/umerijaz/nanopore

## Evaluation signals

- Average consensus sequence accuracy of ~99.5% for 2D and 1D2 sequencing on mock communities
- OTU counts at the optimal (low) partition setting match the theoretical number of organisms in the mock community
- Sweep over partition counts 1-7 reproduces the reported trend: accuracy decreases and OTU count increases as partition count rises
- Residual errors are dominated by deletions in homopolymer regions rather than substitutions or other indels
- Results are reproducible with scripts from github.com/umerijaz/nanopore

## Limitations

- Residual errors in consensus sequences originate almost entirely from deletions in homopolymer regions; homopolymer-aware base calling or error correction would be needed for short-read-comparable accuracy
- Reference-based clustering of noisy long reads is limited by incorrect mapping and discarding of reads from organisms absent from the reference database — do not mix reference-based filtering with this de novo workflow
- Increasing the number of partitions (decreasing partition length) inflates OTU counts above the theoretical threshold and decreases average sequence accuracy, so partition count must be chosen carefully

## Evidence

- [abstract] nanoClust algorithm was designed for read partitioning-based de novo clustering and within cluster consensus calling: "while nanoClust algorithm was designed for read partitioning-based de novo clustering and within cluster consensus calling"
- [abstract] chopSeq preprocessing role: "The developed protocol includes chopSeq software for fragmentation and read orientation correction of INC-Seq consensus reads"
- [abstract] 99.5% accuracy benchmark: "NanoAmpli-Seq accurately estimates the diversity of tested mock communities with average consensus sequence accuracy of 99.5% for 2D and 1D2 sequencing"
- [abstract] error profile: "Nearly all residual errors in NanoAmpli-Seq sequences originate from deletions in homopolymer regions"
- [readme] repository scope: "Scripts for processing short amplicon reads from Oxford Nanopore."
