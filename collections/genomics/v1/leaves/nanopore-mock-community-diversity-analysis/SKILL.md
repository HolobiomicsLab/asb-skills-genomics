---
name: nanopore-mock-community-diversity-analysis
description: Use when assessing whether direct 97% VSEARCH clustering of full-length
  post-chopSeq nanopore amplicon reads gives accurate OTU richness for mock communities,
  or when justifying a partition-based clustering step by quantifying singleton-driven
  OTU inflation.
when_to_use_negative:
- When the goal is accurate de novo OTU estimation itself — the article shows direct
  full-length clustering fails here; use the partition-based nanoClust approach instead
- When reads have not been fragmented and orientation-corrected with chopSeq and size-selected,
  since the skill is defined on post-chopSeq, size-selected full-length reads
edam_operation: http://edamontology.org/operation_0291
edam_topics:
- http://edamontology.org/topic_0637
- http://edamontology.org/topic_3174
tools:
- name: VSEARCH
  role: De novo clustering (cluster_fast) of full-length post-chopSeq reads at 97%
    identity to count OTUs and singletons
- name: chopSeq
  role: Upstream fragmentation and read orientation correction of INC-Seq consensus
    reads that produce the input read set
  repo: github.com/umerijaz/nanopore
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/gigascience/giy140
    title: 'NanoAmpli-Seq: a workflow for amplicon sequencing for mixed microbial
      communities on the nanopore sequencing platform'
  source_content_hash: sha256:63ef5f0fe4dd8344b03b6bfaaaafa1796b67bd10fbea11fa7159bf3e813177d5
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - VSEARCH
  - chopSeq
  edam_operation: http://edamontology.org/operation_0291
  edam_topics:
  - http://edamontology.org/topic_0637
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 4
---

# nanopore-mock-community-diversity-analysis

## Summary

Benchmarks de novo OTU richness estimation of nanopore amplicon mock communities by directly clustering post-chopSeq, size-selected full-length reads with VSEARCH at 97% identity, quantifying how grossly OTU counts (817/1301/2122/2725 OTUs) inflate relative to theoretical mock richness and thereby motivating partition-based nanoClust.

## When to use

Use when assessing whether direct 97% VSEARCH clustering of full-length post-chopSeq nanopore amplicon reads gives accurate OTU richness for mock communities, or when justifying a partition-based clustering step by quantifying singleton-driven OTU inflation.

## When NOT to use

- When the goal is accurate de novo OTU estimation itself — the article shows direct full-length clustering fails here; use the partition-based nanoClust approach instead
- When reads have not been fragmented and orientation-corrected with chopSeq and size-selected, since the skill is defined on post-chopSeq, size-selected full-length reads

## Inputs

- Post-chopSeq, size-selected full-length amplicon reads in FASTA format for each mock community dataset (2D single-organism, 1D2 single-organism, 2D 10-organism, 1D2 10-organism)
- Theoretical mock community richness values (expected organism counts) for comparison

## Outputs

- Summary table otu_counts.csv with columns for dataset, total OTUs, and singleton OTU counts
- Per-dataset OTU and singleton counts from VSEARCH uc/clstr clustering output

## How to apply

Load the post-chopSeq, size-selected full-length amplicon reads (FASTA) for each mock community dataset (2D and 1D2 single-organism, 2D and 1D2 10-organism) and run de novo clustering with VSEARCH cluster_fast directly at the 97% identity threshold, with no reference mapping or read partitioning beforehand. Parse the VSEARCH .uc/.clstr output to count total OTUs and singleton OTUs (clusters containing exactly one read) per dataset, and write these to a summary table (otu_counts.csv) with columns for dataset, total OTUs, and singleton counts. The rationale, grounded in the NanoAmpli-Seq paper, is that raw nanopore read error causes direct full-length clustering to fragment single taxa into many spurious clusters — 817 OTUs (777 singletons) for 2D and 1,301 (1,238) for 1D2 single-organism experiments, and 2,122 (1,742) and 2,725 (2,447) for the 10-organism experiments — far above true mock richness, demonstrating the failure mode that nanoClust's read partitioning-based de novo clustering and within-cluster consensus calling is designed to fix. Validate by checking that the table is well-formed and the four total OTU counts match the reported figures, allowing minor divergence from read-set versioning.

## Related tools

- **VSEARCH** (De novo clustering (cluster_fast) of full-length post-chopSeq reads at 97% identity to count OTUs and singletons)
- **chopSeq** (Upstream fragmentation and read orientation correction of INC-Seq consensus reads that produce the input read set) — github.com/umerijaz/nanopore

## Examples

```
vsearch --cluster_fast reads_post_chopSeq.fasta --id 0.97 --uc otus.uc --sizeout; python count_singletons.py otus.uc --out otu_counts.csv
```

## Evaluation signals

- otu_counts.csv exists and is well-formed with dataset, total OTUs, and singleton columns
- 2D single-organism total OTU count reproduces 817 with 777 singletons
- 1D2 single-organism total OTU count reproduces 1,301 with 1,238 singletons
- 10-organism totals reproduce 2,122 (1,742 singletons) for 2D and 2,725 (2,447 singletons) for 1D2, allowing minor divergence from read-set versioning
- Singleton fractions remain extremely high (>90% of OTUs), confirming the expected direct-clustering inflation

## Limitations

- Direct full-length clustering grossly inflates OTU counts far above theoretical mock community richness, so its outputs are diagnostic, not usable richness estimates
- Counts may diverge slightly depending on read-set versioning
- Residual consensus errors are dominated by deletions in homopolymer regions, which drive read divergence and spurious singleton clusters

## Evidence

- [results] OTU inflation counts: [evidence span withheld — anchor status: not_found]
- [abstract] nanoClust rationale: "while nanoClust algorithm was designed for read partitioning-based de novo clustering and within cluster consensus calling"
- [abstract] chopSeq upstream step: "The developed protocol includes chopSeq software for fragmentation and read orientation correction of INC-Seq consensus reads"
- [readme] repository README: "Scripts for processing short amplicon reads from Oxford Nanopore."
