---
name: otu-count-partition-sensitivity-analysis
description: Use when tuning a de novo clustering partition count for long-read 16S
  amplicon data and you need to quantify the OTU-inflation vs. consensus-accuracy
  trade-off across partition settings (1–7) on mock or known-composition communities.
when_to_use_negative:
- Data are raw noisy nanopore reads that have not been processed by chopSeq/INC-Seq
  consensus generation — the sweep assumes consensus reads as input
- The community composition is unknown (no reference 16S sequences), so consensus
  accuracy cannot be computed and only OTU-count trends are assessable
- The goal is taxonomic classification with a reference database rather than de novo
  partitioning-based clustering, which the paper argues risks incorrect mapping and
  discarding of reads
edam_operation: http://edamontology.org/operation_0291
edam_topics:
- http://edamontology.org/topic_0637
- http://edamontology.org/topic_3174
tools:
- name: nanoClust
  role: Read partitioning-based de novo clustering and within-cluster consensus calling;
    run repeatedly with partition counts 1–7 to produce OTUs and consensus 16S sequences
    per setting
  repo: https://github.com/umerijaz/nanopore
- name: chopSeq
  role: Upstream fragmentation and read orientation correction of INC-Seq consensus
    reads, producing the input sequences for the partition sweep
  repo: https://github.com/umerijaz/nanopore
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/gigascience/giy140
    title: 'NanoAmpli-Seq: a workflow for amplicon sequencing for mixed microbial
      communities on the nanopore sequencing platform'
  source_content_hash: sha256:d3b70fb5018cfd0fef75df59b7759816965550f01e9e77d235430fef63e12cf6
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/umerijaz/nanopore
  tools:
  - chopSeq
  - nanoClust
  edam_operation: http://edamontology.org/operation_0291
  edam_topics:
  - http://edamontology.org/topic_0637
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# otu-count-partition-sensitivity-analysis

## Summary

A sensitivity-analysis skill that sweeps the nanoClust partition count (1–7 partitions) over chopSeq-processed INC-Seq consensus 16S rRNA reads from ONT MinION mock communities, quantifying how partition length trades off detected OTU count against consensus sequence accuracy. It reproduces the NanoAmpli-Seq Fig. 7 / Table 2 ablation showing that more partitions inflate OTU counts and degrade accuracy.

## When to use

Use when tuning a de novo clustering partition count for long-read 16S amplicon data and you need to quantify the OTU-inflation vs. consensus-accuracy trade-off across partition settings (1–7) on mock or known-composition communities.

## When NOT to use

- Data are raw noisy nanopore reads that have not been processed by chopSeq/INC-Seq consensus generation — the sweep assumes consensus reads as input
- The community composition is unknown (no reference 16S sequences), so consensus accuracy cannot be computed and only OTU-count trends are assessable
- The goal is taxonomic classification with a reference database rather than de novo partitioning-based clustering, which the paper argues risks incorrect mapping and discarding of reads

## Inputs

- chopSeq-processed INC-Seq consensus reads (FASTA/FASTQ) for 1-organism and 10-organism mock communities in 2D and 1D2 sequencing modes
- Mock community reference 16S rRNA gene sequences (for accuracy alignment)

## Outputs

- OTU (cluster) counts per partition count setting (1–7)
- Consensus sequence accuracy values per partition count setting
- partition_sweep_results.csv tabulating OTU counts and accuracy for the 1-, 2-, and 3-partition settings

## How to apply

Load the chopSeq-processed INC-Seq consensus reads for the 1-organism and 10-organism mock communities in both 2D and 1D2 nanopore sequencing modes. For each dataset, run the nanoClust read partitioning-based de novo clustering algorithm with the partition count set to each value from 1 through 7, calling within-cluster consensus full-length 16S rRNA gene sequences at each setting. Count the number of OTUs (clusters) per partition setting to measure OTU inflation as partition count rises, and align each consensus sequence to the mock community reference 16S sequences to compute consensus accuracy. Rationale: decreasing partition length (more partitions) splits true organisms into spurious clusters, inflating OTU counts above the theoretical threshold while lowering accuracy, whereas fewer, larger partitions concentrate reads and improve consensus quality. Tabulate OTU counts and accuracy for the 1-, 2-, and 3-partition settings (as in Fig. 7 and Table 2) and write the results to partition_sweep_results.csv.

## Related tools

- **nanoClust** (Read partitioning-based de novo clustering and within-cluster consensus calling; run repeatedly with partition counts 1–7 to produce OTUs and consensus 16S sequences per setting) — https://github.com/umerijaz/nanopore
- **chopSeq** (Upstream fragmentation and read orientation correction of INC-Seq consensus reads, producing the input sequences for the partition sweep) — https://github.com/umerijaz/nanopore

## Evaluation signals

- Consensus accuracy at the optimal (low) partition setting matches the reported average of 99.5% for 2D and 1D2 nanopore sequencing on the mock communities
- Accuracy monotonically decreases as the partition count rises from 1 toward 7
- OTU count monotonically increases with partition count, exceeding the theoretical mock-community organism count at high partition settings
- partition_sweep_results.csv contains OTU counts and accuracy values for the 1-, 2-, and 3-partition settings consistent with Fig. 7 and Table 2
- Residual errors in the consensus sequences are dominated by deletions in homopolymer regions, consistent with the reported NanoAmpli-Seq error profile

## Limitations

- Residual consensus errors originate almost entirely from deletions in homopolymer regions, so accuracy gains from partition tuning are bounded unless homopolymer-aware base calling or error correction is applied
- Sweep requires mock communities with known reference 16S sequences; accuracy cannot be evaluated for uncharacterized samples
- Trend was characterized for 1- and 10-organism mock communities in 2D and 1D2 modes on the ONT MinION platform; extrapolation to complex environmental communities is not demonstrated

## Evidence

- [abstract] nanoClust algorithm was designed for read partitioning-based de novo clustering and within cluster consensus calling: "while nanoClust algorithm was designed for read partitioning-based de novo clustering and within cluster consensus calling"
- [abstract] 99.5% average consensus accuracy: "NanoAmpli-Seq accurately estimates the diversity of tested mock communities with average consensus sequence accuracy of 99.5% for 2D and 1D2 sequencing on the nanopore sequencing platform"
- [abstract] homopolymer deletion limitation: "Nearly all residual errors in NanoAmpli-Seq sequences originate from deletions in homopolymer regions"
- [intro] reference-based clustering limitation motivating de novo partitioning: "incorrect mapping to a reference database is likely due to high error rates of raw nanopore reads"
- [readme] repository pointer for scripts: "Scripts for processing short amplicon reads from Oxford Nanopore."
