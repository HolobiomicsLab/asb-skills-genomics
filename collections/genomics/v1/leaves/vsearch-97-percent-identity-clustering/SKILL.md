---
name: vsearch-97-percent-identity-clustering
description: 'Use when benchmarking OTU recovery on post-chopSeq full-length 16S reads:
  cluster each mock-community FASTA directly with VSEARCH at 97% similarity to count
  OTUs/singletons and demonstrate whether direct clustering yields accurate richness
  estimates.'
when_to_use_negative:
- The goal is accurate OTU/richness estimation on noisy full-length nanopore reads
  — use the partition-based nanoClust approach instead, since direct 97% clustering
  is exactly the failure mode this skill demonstrates
- Input reads are raw un-corrected nanopore reads that have not been through chopSeq
  fragmentation/orientation correction and size selection
- Input is already an OTU table or feature table rather than FASTA reads
edam_operation: http://edamontology.org/operation_0291
edam_topics:
- http://edamontology.org/topic_0637
- http://edamontology.org/topic_3941
tools:
- name: VSEARCH
  role: Performs de novo OTU clustering (cluster_fast) at the 97% identity threshold
    on the full-length read sets; its .uc/.clstr output is parsed for OTU and singleton
    counts
- name: chopSeq
  role: Upstream tool that fragments and orientation-corrects INC-Seq consensus reads
    to produce the full-length FASTA inputs clustered here
  repo: github.com/umerijaz/nanopore
- name: nanoClust
  role: The paper's partitioning-based alternative to direct clustering; contrasting
    direct VSEARCH OTU counts against nanoClust output is the point of this skill
  repo: github.com/umerijaz/nanopore
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/gigascience/giy140
    title: 'NanoAmpli-Seq: a workflow for amplicon sequencing for mixed microbial
      communities on the nanopore sequencing platform'
  source_content_hash: sha256:1dbaf2d7e26153341460793446be34ace00b18727c46ed5101cadb5f75286e7c
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - VSEARCH
  - chopSeq
  - nanoClust
  edam_operation: http://edamontology.org/operation_0291
  edam_topics:
  - http://edamontology.org/topic_0637
  - http://edamontology.org/topic_3941
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 5
---

# vsearch-97-percent-identity-clustering

## Summary

De novo clustering of full-length nanopore amplicon reads with VSEARCH (cluster_fast) at a 97% identity threshold to enumerate OTUs and singletons. In the NanoAmpli-Seq context it serves as a negative-control baseline showing that direct clustering of noisy full-length reads grossly inflates OTU richness.

## When to use

Use when benchmarking OTU recovery on post-chopSeq full-length 16S reads: cluster each mock-community FASTA directly with VSEARCH at 97% similarity to count OTUs/singletons and demonstrate whether direct clustering yields accurate richness estimates.

## When NOT to use

- The goal is accurate OTU/richness estimation on noisy full-length nanopore reads — use the partition-based nanoClust approach instead, since direct 97% clustering is exactly the failure mode this skill demonstrates
- Input reads are raw un-corrected nanopore reads that have not been through chopSeq fragmentation/orientation correction and size selection
- Input is already an OTU table or feature table rather than FASTA reads

## Inputs

- Post-chopSeq, size-selected full-length 16S rRNA amplicon reads in FASTA format (2D and 1D2 nanopore mock-community datasets)

## Outputs

- VSEARCH OTU clustering output (.uc/.clstr files)
- otu_counts.csv summary table with columns for dataset, total OTUs, and singleton counts

## How to apply

Load the post-chopSeq, size-selected full-length amplicon reads (FASTA) for each dataset (2D and 1D2 single-organism and 10-organism mock communities) and run de novo clustering with VSEARCH's cluster_fast at a 97% identity threshold, without prior reference mapping or nanoClust partitioning. Parse the VSEARCH .uc/.clstr output to count total OTUs and singleton clusters (exactly one read per cluster). Write the per-dataset counts to a summary table such as otu_counts.csv. The expected result is dramatic OTU inflation far above true mock richness (e.g., 817 OTUs with 777 singletons for 2D single-organism; 2,725 with 2,447 singletons for the 1D2 10-organism set), which is the rationale for the paper's partition-based nanoClust approach.

## Related tools

- **VSEARCH** (Performs de novo OTU clustering (cluster_fast) at the 97% identity threshold on the full-length read sets; its .uc/.clstr output is parsed for OTU and singleton counts)
- **chopSeq** (Upstream tool that fragments and orientation-corrects INC-Seq consensus reads to produce the full-length FASTA inputs clustered here) — github.com/umerijaz/nanopore
- **nanoClust** (The paper's partitioning-based alternative to direct clustering; contrasting direct VSEARCH OTU counts against nanoClust output is the point of this skill) — github.com/umerijaz/nanopore

## Examples

```
vsearch --cluster_fast reads.fa --id 0.97 --uc otus.uc --centroids otus.fa
```

## Evaluation signals

- otu_counts.csv exists, is well-formed, and contains rows for all four datasets (2D and 1D2 single-organism and 10-organism mock communities)
- Total OTU counts match the reported figures: 817 (777 singletons) for 2D single-organism, 1,301 (1,238) for 1D2 single-organism, 2,122 (1,742) for 2D 10-organism, 2,725 (2,447) for 1D2 10-organism, allowing minor divergence from read-set versioning
- Singletons account for ~90% or more of total OTUs in each dataset (e.g., 777/817, 1,238/1,301, 2,447/2,725), confirming the expected inflation pattern
- Counts far exceed theoretical mock community richness (1 or 10 organisms), which is the defining outcome of this baseline

## Limitations

- Direct 97% clustering of full-length post-chopSeq nanopore reads grossly inflates OTU counts (hundreds to thousands of mostly single-copy OTUs vs. 1–10 true species), so results must not be interpreted as biological richness
- Reported counts may diverge slightly with read-set versioning
- The paper attributes residual nanopore errors to deletions in homopolymer regions, which is what drives over-splitting at the 97% threshold

## Evidence

- [results] Direct VSEARCH clustering of full-length post-chopSeq, size-selected reads at 97% similarity grossly inflated OTU counts far above the theoretical mock community richness: [evidence span withheld — anchor status: not_found]
- [results] demonstrating that direct full-length clustering fails and motivating the partition-based nanoClust approach: [evidence span withheld — anchor status: not_found]
- [abstract] while nanoClust algorithm was designed for read partitioning-based de novo clustering and within cluster consensus calling: "while nanoClust algorithm was designed for read partitioning-based de novo clustering and within cluster consensus calling"
- [readme] Scripts for processing short amplicon reads from Oxford Nanopore.: "Scripts for processing short amplicon reads from Oxford Nanopore. For details, please see:"
