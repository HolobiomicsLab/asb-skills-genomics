---
name: 2d-vs-1d2-basecalling-readset-handling
description: Use when MinION amplicon data from NanoAmpli-Seq (Calus et al. 2018)
  exists as both 2D and 1D2 basecalled read sets (single-organism and multi-organism
  mock communities) and each set must be processed and compared separately, e.g. to
  reproduce OTU-inflation benchmarks.
when_to_use_negative:
- Reads are not INC-Seq consensus reads corrected by chopSeq (e.g., raw noisy nanopore
  reads intended for reference mapping) — the per-mode read-set handling is defined
  for the NanoAmpli-Seq post-chopSeq full-length amplicons
- The analysis goal is diversity estimation via the partition-based nanoClust approach
  rather than benchmarking direct clustering behaviour — pooling or reference-based
  workflows would be the intended path
- Data comes from a single basecalling mode only and no 2D-vs-1D2 comparison is required
  — then a single read-set pipeline suffices
edam_operation: http://edamontology.org/operation_0501
edam_topics:
- http://edamontology.org/topic_3174
tools:
- name: chopSeq
  role: Fragmentation and read orientation correction of INC-Seq consensus reads,
    producing the full-length reads that are then handled per basecalling read set
  repo: https://github.com/umerijaz/nanopore
- name: VSEARCH
  role: De novo cluster_fast clustering of each post-chopSeq read set at 97% identity
    to derive the per-mode OTU/singleton counts being benchmarked
- name: nanoClust
  role: The partitioning-based de novo clustering alternative that the direct-clustering
    benchmark on 2D/1D2 read sets is intended to motivate
  repo: https://github.com/umerijaz/nanopore
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/gigascience/giy140
    title: 'NanoAmpli-Seq: a workflow for amplicon sequencing for mixed microbial
      communities on the nanopore sequencing platform'
  source_content_hash: sha256:3a29b42194f4bba451bc7dac50e79c269e2d2bdfa7e561b200fad95bdbd47a2f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/umerijaz/nanopore
  tools:
  - VSEARCH
  - chopSeq
  - nanoClust
  edam_operation: http://edamontology.org/operation_0501
  edam_topics:
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# 2D-vs-1D2-basecalling-readset-handling

## Summary

Handles the 2D and 1D2 MinION basecalled read sets of the NanoAmpli-Seq workflow as parallel, separately processed datasets — running each (e.g., post-chopSeq full-length amplicon reads) through clustering/OTU-counting steps independently so 2D vs 1D2 behaviour (e.g., OTU inflation and singleton fractions) can be compared and validated against reported counts.

## When to use

Use when MinION amplicon data from NanoAmpli-Seq (Calus et al. 2018) exists as both 2D and 1D2 basecalled read sets (single-organism and multi-organism mock communities) and each set must be processed and compared separately, e.g. to reproduce OTU-inflation benchmarks.

## When NOT to use

- Reads are not INC-Seq consensus reads corrected by chopSeq (e.g., raw noisy nanopore reads intended for reference mapping) — the per-mode read-set handling is defined for the NanoAmpli-Seq post-chopSeq full-length amplicons
- The analysis goal is diversity estimation via the partition-based nanoClust approach rather than benchmarking direct clustering behaviour — pooling or reference-based workflows would be the intended path
- Data comes from a single basecalling mode only and no 2D-vs-1D2 comparison is required — then a single read-set pipeline suffices

## Inputs

- Post-chopSeq, size-selected full-length amplicon reads in FASTA for each 2D read set (single-organism and 10-organism mock communities)
- Post-chopSeq, size-selected full-length amplicon reads in FASTA for each 1D2 read set (single-organism and 10-organism mock communities)

## Outputs

- Per-read-set OTU and singleton counts (VSEARCH uc/clstr-derived)
- Summary table otu_counts.csv with columns: dataset, total OTUs, singleton counts

## How to apply

Treat each basecalling mode (2D, 1D2) and each mock community (single-organism, 10-organism) as a distinct read set; for NanoAmpli-Seq these are INC-Seq consensus reads corrected by chopSeq before downstream steps. Run the comparison operation (e.g., de novo VSEARCH cluster_fast at 97% identity on post-chopSeq full-length reads) on every read set independently — do not pool 2D and 1D2 reads. Parse the VSEARCH .uc/.clstr output per set to count total OTUs and singletons (clusters with exactly one read), then assemble the per-dataset results into one summary table (otu_counts.csv). The rationale is that the article reports 2D and 1D2 metrics side by side (99.5% average consensus accuracy 'for 2D and 1D2 sequencing'), so 1:1 per-set processing is required to validate against the published per-mode figures.

## Related tools

- **chopSeq** (Fragmentation and read orientation correction of INC-Seq consensus reads, producing the full-length reads that are then handled per basecalling read set) — https://github.com/umerijaz/nanopore
- **VSEARCH** (De novo cluster_fast clustering of each post-chopSeq read set at 97% identity to derive the per-mode OTU/singleton counts being benchmarked)
- **nanoClust** (The partitioning-based de novo clustering alternative that the direct-clustering benchmark on 2D/1D2 read sets is intended to motivate) — https://github.com/umerijaz/nanopore

## Examples

```
vsearch --cluster_fast post_chopseq_2D_single.fa --id 0.97 --uc otus_2D_single.uc
```

## Evaluation signals

- 2D single-organism set yields 817 total OTUs with 777 singletons after direct VSEARCH clustering at 97% identity
- 1D2 single-organism set yields 1,301 total OTUs with 1,238 singletons
- 10-organism sets yield 2,122 OTUs (1,742 singletons) for 2D and 2,725 OTUs (2,447 singletons) for 1D2, within tolerance for read-set versioning
- otu_counts.csv exists, is well-formed, and contains one row per dataset with dataset, total OTU, and singleton columns
- Counts for 2D and 1D2 of the same mock community are clearly divergent, confirming the two basecalled sets were processed independently

## Limitations

- Direct full-length VSEARCH clustering at 97% similarity grossly inflates OTU counts far above theoretical mock community richness (hundreds to thousands of OTUs, dominated by singletons), so these counts are a negative-control benchmark, not a usable richness estimate
- Counts are allowed only minor divergence from reported figures because of read-set versioning differences
- Reported counts are specific to the tested mock communities and the MinION 2D/1D2 basecalling of the 2018 NanoAmpli-Seq protocol

## Evidence

- [abstract] Per-mode consensus accuracy claim: "average consensus sequence accuracy of 99.5% for 2D and 1D2 sequencing on the nanopore sequencing platform"
- [abstract] chopSeq role in producing per-mode reads: "The developed protocol includes chopSeq software for fragmentation and read orientation correction of INC-Seq consensus reads"
- [abstract] nanoClust as the partition-based alternative: "while nanoClust algorithm was designed for read partitioning-based de novo clustering and within cluster consensus calling"
- [intro] MinION platform and read-set provenance: "single-molecule sensing technologies on the Oxford Nanopore Technologies (ONT) MinION platform, has opened the possibility of obtaining ultra-long reads"
- [readme] Repository for the read-set processing scripts: "Scripts for processing short amplicon reads from Oxford Nanopore."
