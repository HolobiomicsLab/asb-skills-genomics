---
name: de-novo-read-partition-clustering
description: Use when MinION amplicon reads (e.g. INC-Seq consensus 16S rRNA reads
  from a mock community or mixed microbial sample) must be grouped into OTUs and turned
  into consensus sequences without a reference database.
when_to_use_negative:
- Input is raw noisy MinION reads that have not been through INC-Seq (>=3 concatemers)
  and chopSeq fragmentation/orientation correction
- The goal is reference-based OTU assignment against SILVA or another curated 16S
  database rather than de novo clustering
- Short-read amplicon data (e.g. Illumina MiSeq) where read length does not support
  partitioning-based clustering
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_0196
tools:
- name: nanoClust
  role: performs read partitioning-based de novo clustering and within-cluster consensus
    calling on chopSeq-processed INC-Seq consensus reads
  repo: https://github.com/umerijaz/nanopore
- name: chopSeq
  role: upstream fragmentation and read orientation correction of INC-Seq consensus
    reads, producing the input partitions for nanoClust clustering
  repo: https://github.com/umerijaz/nanopore
provenance:
  source_task_ids:
  - task_001
  - task_004
  source_papers:
  - doi: 10.1093/gigascience/giy140
    title: 'NanoAmpli-Seq: a workflow for amplicon sequencing for mixed microbial
      communities on the nanopore sequencing platform'
  source_content_hash: sha256:32714107a750919f94382fae1f5946a7d18d27873b4b71c8eacda89308e303af
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/umerijaz/nanopore
  tools:
  - chopSeq
  - nanoClust
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_0196
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# de-novo-read-partition-clustering

## Summary

nanoClust-style de novo clustering of nanopore amplicon reads: reads are partitioned along the read length, clustered without any reference database, and a within-cluster consensus full-length 16S rRNA gene sequence is called per OTU.

## When to use

Use when MinION amplicon reads (e.g. INC-Seq consensus 16S rRNA reads from a mock community or mixed microbial sample) must be grouped into OTUs and turned into consensus sequences without a reference database.

## When NOT to use

- Input is raw noisy MinION reads that have not been through INC-Seq (>=3 concatemers) and chopSeq fragmentation/orientation correction
- The goal is reference-based OTU assignment against SILVA or another curated 16S database rather than de novo clustering
- Short-read amplicon data (e.g. Illumina MiSeq) where read length does not support partitioning-based clustering

## Inputs

- chopSeq-processed INC-Seq consensus reads from ONT MinION 2D or 1D2 sequencing (full-length 16S rRNA gene amplicons)
- partition count / partition length parameter setting (1-7 partitions; 3 used in the reference workflow)

## Outputs

- OTU clusters (de novo, reference-free) of nanopore amplicon reads
- per-OTU full-length 16S rRNA gene consensus sequences (e.g. nanoampliseq_consensus.fasta)
- OTU counts per partition setting for the partition-count ablation (e.g. partition_sweep_results.csv)

## How to apply

Start from INC-Seq circular consensus reads (reads supporting at least 3 concatemers) that have been fragmented and orientation-corrected with chopSeq. Partition each read into segments (nanoClust partitioning; the reference workflow uses 3 partitions, and the partition count should be swept from 1 to 7 since more partitions means shorter partitions), then perform de novo clustering of the partitions to group reads by organism. For each cluster, call a within-cluster consensus using multiple sequence alignment of ~50 reads per OTU (MAFFT G-INS-i) to obtain a full-length 16S rRNA gene OTU sequence, writing results to a FASTA such as nanoampliseq_consensus.fasta. The rationale is a completely reference-free approach: raw nanopore reads are too noisy for reliable reference mapping, and reference-based clustering discards organisms absent from the database. Fewer, longer partitions give fewer inflated OTUs and higher accuracy.

## Related tools

- **nanoClust** (performs read partitioning-based de novo clustering and within-cluster consensus calling on chopSeq-processed INC-Seq consensus reads) — https://github.com/umerijaz/nanopore
- **chopSeq** (upstream fragmentation and read orientation correction of INC-Seq consensus reads, producing the input partitions for nanoClust clustering) — https://github.com/umerijaz/nanopore

## Evaluation signals

- Consensus sequences for tested mock communities reach average accuracy of ~99.5% for 2D and 1D2 sequencing (per-OTU range 99.2%-100%)
- OTU count stays at or below the theoretical number of organisms in the mock community; more partitions (shorter partition length) inflates OTU counts above this threshold
- Average consensus accuracy decreases as the partition count rises from the optimal low setting toward 7
- Alignment of each called consensus (MAFFT G-INS-i, ~50 reads per OTU) shows the entire read aligned to the mock community reference 16S sequence
- Residual consensus errors are dominated by deletions in homopolymer regions rather than mismatches

## Limitations

- Residual errors in the consensus sequences are almost entirely deletions in homopolymer regions, a nanopore-specific error mode
- Too many partitions (short partition length) inflates the detected OTU count above the true number of organisms and lowers average accuracy
- Accuracy comparable to short-read platforms would require homopolymer-aware base calling or error correction, which the base workflow lacks

## Evidence

- [abstract] nanoClust de novo clustering and consensus: "while nanoClust algorithm was designed for read partitioning-based de novo clustering and within cluster consensus calling to obtain accurate full-length 16S rRNA gene sequences"
- [abstract] chopSeq input preparation: "The developed protocol includes chopSeq software for fragmentation and read orientation correction of INC-Seq consensus reads"
- [intro] de novo rationale: "The more robust alternative towards high accuracy consensus sequence generation would be a completely de novo approach, i.e., generation of a consensus sequence without the use of any reference"
- [abstract] expected accuracy benchmark: "NanoAmpli-Seq accurately estimates the diversity of tested mock communities with average consensus sequence accuracy of 99.5% for 2D and 1D2 sequencing on the nanopore sequencing platform"
