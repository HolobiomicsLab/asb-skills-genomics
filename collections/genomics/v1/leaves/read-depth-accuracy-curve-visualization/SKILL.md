---
name: read-depth-accuracy-curve-visualization
description: Use when planning nanopore amplicon experiments or consensus calling
  where you must decide how many reads per OTU are needed, e.g. to verify >99% accuracy
  above 5 reads and stability at >=50 reads.
when_to_use_negative:
- Your goal is mock-community diversity estimation rather than characterizing read-depth
  requirements for consensus accuracy.
- Your input is a reference-based pipeline (e.g. mapping raw reads to SILVA); the
  skill targets nanoClust's de novo clustering approach, which was motivated because
  reference-based mapping of noisy nanopore reads is unreliable.
- Reads are short reads (Illumina MiSeq, Ion Torrent PGM) rather than long nanopore
  amplicon reads.
edam_operation: http://edamontology.org/operation_0564
edam_topics:
- http://edamontology.org/topic_3293
- http://edamontology.org/topic_0622
tools:
- name: nanoClust
  role: Read-partitioning-based de novo clustering and within-cluster consensus calling
    on each subsample to obtain full-length 16S rRNA gene consensus sequences
  repo: github.com/umerijaz/nanopore
- name: ONT MinION
  role: Instrument generating the 2D and 1D2 amplicon reads subsampled at each depth
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/gigascience/giy140
    title: 'NanoAmpli-Seq: a workflow for amplicon sequencing for mixed microbial
      communities on the nanopore sequencing platform'
  source_content_hash: sha256:d444439868d5f26ebb3bd91f94cd4b14242c223d392cd995f0b96ad37daca431
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - ONT MinION
  - nanoClust
  edam_operation: http://edamontology.org/operation_0564
  edam_topics:
  - http://edamontology.org/topic_3293
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 5
---

# read-depth-accuracy-curve-visualization

## Summary

Characterizes how many nanopore reads must feed consensus construction for accurate full-length 16S rRNA gene sequences by subsampling per-OTU read depth, running nanoClust de novo clustering and consensus calling at each depth, and plotting consensus accuracy (sequence identity vs a SILVA reference) as a function of read count.

## When to use

Use when planning nanopore amplicon experiments or consensus calling where you must decide how many reads per OTU are needed, e.g. to verify >99% accuracy above 5 reads and stability at >=50 reads.

## When NOT to use

- Your goal is mock-community diversity estimation rather than characterizing read-depth requirements for consensus accuracy.
- Your input is a reference-based pipeline (e.g. mapping raw reads to SILVA); the skill targets nanoClust's de novo clustering approach, which was motivated because reference-based mapping of noisy nanopore reads is unreliable.
- Reads are short reads (Illumina MiSeq, Ion Torrent PGM) rather than long nanopore amplicon reads.

## Inputs

- One-organism 2D nanopore amplicon read set (Fastq)
- One-organism 1D2 nanopore amplicon read set (Fastq)
- SILVA 16S rRNA reference sequence for the organism

## Outputs

- Consensus FASTA of full-length 16S rRNA gene sequences per subsample
- accuracy_curve.png plotting consensus accuracy (identity to SILVA reference) versus subsampled read count with annotated plateau and stability regions

## How to apply

From the one-organism 2D and 1D2 ONT MinION amplicon read sets in the nanopore repository, subsample n reads per OTU for depth levels spanning 5–100. For each subsample, run nanoClust read-partitioning-based de novo clustering followed by within-cluster consensus calling to produce full-length 16S rRNA gene consensus sequences. Align each consensus against the organism's SILVA 16S reference and compute sequence identity as the accuracy metric. Repeat across depths to characterize the accuracy curve, marking the regions above 5 reads (>99%), the ~10–15 read plateau, and the >=50 read stability region. Write the consensus FASTA and an accuracy_curve.png plot.

## Related tools

- **nanoClust** (Read-partitioning-based de novo clustering and within-cluster consensus calling on each subsample to obtain full-length 16S rRNA gene consensus sequences) — github.com/umerijaz/nanopore
- **ONT MinION** (Instrument generating the 2D and 1D2 amplicon reads subsampled at each depth)

## Evaluation signals

- Consensus accuracy (identity vs the SILVA 16S reference) exceeds 99% once more than 5 reads are used per consensus.
- The accuracy curve plateaus at approximately 10–15 reads and is reliably high only at >=50 reads.
- Subsampling spans depth levels 5–100 and accuracy is computed identically at each level against the same SILVA reference.
- Deliverables are present: consensus FASTA per OTU and accuracy_curve.png with the 5-read, 10–15-read, and >=50-read regions marked.
- Consensus sequences are full-length 16S rRNA gene sequences, not partial or reference-mapped assemblies.

## Limitations

- Accuracy is reliably high only when at least 50 reads are available for consensus construction; at 5–15 reads accuracy is above 99% but the curve plateaus rather than reaching maximum stability.
- Nearly all residual errors in NanoAmpli-Seq sequences originate from deletions in homopolymer regions, capping achievable accuracy without homopolymer-aware base calling or error correction.
- Curve behavior was characterized on a one-organism experiment; extrapolating the exact plateau to complex mixed communities is not directly supported by this card.

## Evidence

- [results] Consensus accuracy surpasses 99% when more than five reads are used for consensus construction and plateaus at approximately 10-15 reads, with accuracy reliably high only when at least 50 reads are available.: [evidence span withheld — anchor status: not_found]
- [abstract] nanoClust read partitioning and consensus calling produce the full-length sequences evaluated at each depth.: "nanoClust algorithm was designed for read partitioning-based de novo clustering and within cluster consensus calling to obtain accurate full-length 16S rRNA gene sequences"
- [intro] De novo consensus is used because reference-based mapping of noisy reads is unreliable.: "incorrect mapping to a reference database is likely due to high error rates of raw nanopore reads"
- [abstract] Residual errors concentrate in homopolymer deletions, bounding achievable accuracy.: "Nearly all residual errors in NanoAmpli-Seq sequences originate from deletions in homopolymer regions"
- [readme] The repository provides the scripts for processing the nanopore amplicon reads used in this workflow.: "Scripts for processing short amplicon reads from Oxford Nanopore."
