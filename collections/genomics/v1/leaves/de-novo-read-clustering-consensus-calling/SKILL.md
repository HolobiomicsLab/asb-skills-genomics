---
name: de-novo-read-clustering-consensus-calling
description: Use when raw nanopore (ONT MinION 2D/1D2 or R7 INC-Seq) 16S rRNA amplicon
  reads must be partitioned into organism-level OTUs and high-accuracy full-length
  consensus sequences built entirely de novo, without reference-based mapping.
when_to_use_negative:
- When a reference-based clustering approach is acceptable and the target organisms
  are all present in the reference database — but note the article warns that incorrect
  reference mapping due to high raw nanopore error rates and absent organisms motivates
  the de novo route
- When only short-read data (Illumina MiSeq, Ion Torrent PGM) is available — the workflow
  targets long nanopore amplicon reads
- When the goal is classification-only assignment rather than consensus sequence generation
  — the article notes classification-only analysis is limited in differentiating closely
  related sequences
edam_operation: http://edamontology.org/operation_0291
edam_topics:
- http://edamontology.org/topic_3168
- http://edamontology.org/topic_3697
tools:
- name: nanoClust
  role: Read partitioning-based de novo clustering and within-cluster consensus calling
    to obtain full-length 16S rRNA gene consensus sequences
  repo: github.com/umerijaz/nanopore
- name: chopSeq
  role: Fragmentation and read orientation correction of INC-Seq consensus reads prior
    to nanoClust clustering
  repo: github.com/umerijaz/nanopore
- name: ONT MinION
  role: Sequencing platform producing the 2D and 1D2 long amplicon reads that the
    workflow processes
provenance:
  source_task_ids:
  - task_005
  - task_006
  source_papers:
  - doi: 10.1093/gigascience/giy140
    title: 'NanoAmpli-Seq: a workflow for amplicon sequencing for mixed microbial
      communities on the nanopore sequencing platform'
  source_content_hash: sha256:44affcaa44282f8f6d1f9c7c0b2b86788ce182b44c7b0a91807b5dbd4759af31
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - ONT MinION
  - chopSeq
  - nanoClust
  edam_operation: http://edamontology.org/operation_0291
  edam_topics:
  - http://edamontology.org/topic_3168
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 4
---

# de-novo-read-clustering-consensus-calling

## Summary

A de novo strategy for noisy nanopore amplicon reads that partitions reads into OTU clusters (nanoClust) and calls within-cluster consensus full-length 16S rRNA gene sequences without any reference database, achieving ~99.5% average accuracy on 2D and 1D2 ONT MinION mock community data.

## When to use

Use when raw nanopore (ONT MinION 2D/1D2 or R7 INC-Seq) 16S rRNA amplicon reads must be partitioned into organism-level OTUs and high-accuracy full-length consensus sequences built entirely de novo, without reference-based mapping.

## When NOT to use

- When a reference-based clustering approach is acceptable and the target organisms are all present in the reference database — but note the article warns that incorrect reference mapping due to high raw nanopore error rates and absent organisms motivates the de novo route
- When only short-read data (Illumina MiSeq, Ion Torrent PGM) is available — the workflow targets long nanopore amplicon reads
- When the goal is classification-only assignment rather than consensus sequence generation — the article notes classification-only analysis is limited in differentiating closely related sequences

## Inputs

- Raw or INC-Seq consensus nanopore amplicon reads (FASTQ) from ONT MinION 2D/1D2 chemistry
- Previously published INC-Seq reads (e.g., Li et al. R7 Ladder replicate data from github.com/umerijaz/nanopore)
- SILVA 16S rRNA reference sequences for accuracy evaluation

## Outputs

- Per-OTU full-length 16S rRNA gene consensus sequences (FASTA)
- Per-OTU consensus accuracy values (sequence identity vs. reference OTUs)
- Accuracy-versus-read-depth curves characterizing the >5-read / ~10-15-read plateau / >=50-read stability regions

## How to apply

First apply chopSeq to fragment and orientation-correct INC-Seq consensus reads, then run nanoClust for read partitioning-based de novo clustering followed by within-cluster consensus calling to obtain full-length 16S rRNA gene sequences, partitioning communities into their theoretical OTU count (e.g., 8 OTUs for a 10-organism community). Validate by aligning each consensus against expected OTU reference sequences (e.g., SILVA 16S references) and computing per-OTU sequence identity as the accuracy metric. Accept the result when average consensus accuracy reaches ~99.5% (as for NanoAmpli-Seq on 2D/1D2 data and reprocessed Li et al. R7 Ladder replicate data, with the Fusobacterium OTU at ~98.75%), exceeding the INC-Seq 97-98% median accuracy baseline. Consensus accuracy surpasses 99% with more than five reads per OTU, plateaus at ~10-15 reads, and is reliably high only with at least 50 reads.

## Related tools

- **nanoClust** (Read partitioning-based de novo clustering and within-cluster consensus calling to obtain full-length 16S rRNA gene consensus sequences) — github.com/umerijaz/nanopore
- **chopSeq** (Fragmentation and read orientation correction of INC-Seq consensus reads prior to nanoClust clustering) — github.com/umerijaz/nanopore
- **ONT MinION** (Sequencing platform producing the 2D and 1D2 long amplicon reads that the workflow processes)

## Evaluation signals

- Average consensus sequence accuracy reaches ~99.5% against expected OTU references for 2D and 1D2 mock community data
- Reprocessed Li et al. R7 Ladder replicate data yields average accuracy of ~99.5% with the Fusobacterium OTU at ~98.75%, exceeding the INC-Seq 97-98% median baseline
- Called consensus sequences partition the community into its theoretical OTU count (e.g., 8 OTUs for a 10-organism community)
- Read-depth subsampling shows consensus accuracy surpasses 99% at >5 reads, plateaus at ~10-15 reads, and is stable only at >=50 reads
- Residual errors are dominated by deletions in homopolymer regions, consistent with the paper's error-profile finding

## Limitations

- Nearly all residual errors in NanoAmpli-Seq sequences originate from deletions in homopolymer regions, so homopolymer-aware base calling or error correction would be needed to reach short-read platform accuracy
- Consensus accuracy is reliably high only when at least 50 reads are available per OTU; low-depth OTUs yield unstable consensus sequences
- Reference-based comparison of consensus sequences still requires a reference database (e.g., SILVA) for the accuracy evaluation step, even though consensus generation itself is de novo

## Evidence

- [abstract] The developed protocol includes chopSeq software for fragmentation and read orientation correction of INC-Seq consensus reads: "The developed protocol includes chopSeq software for fragmentation and read orientation correction of INC-Seq consensus reads"
- [abstract] nanoClust read partitioning and consensus calling: "while nanoClust algorithm was designed for read partitioning-based de novo clustering and within cluster consensus calling"
- [abstract] Accuracy claim on mock communities: "NanoAmpli-Seq accurately estimates the diversity of tested mock communities with average consensus sequence accuracy of 99.5% for 2D and 1D2 sequencing"
- [intro] Motivation for de novo approach: "incorrect mapping to a reference database is likely due to high error rates of raw nanopore reads"
- [readme] Scripts for processing short amplicon reads from Oxford Nanopore: "Scripts for processing short amplicon reads from Oxford Nanopore"
