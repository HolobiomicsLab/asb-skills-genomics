---
name: singleton-counting-in-otu-tables
description: Use when full-length post-chopSeq, size-selected amplicon reads are clustered
  directly at 97% identity and you need to quantify how far resulting OTU richness
  and singleton counts exceed the known mock-community richness.
when_to_use_negative:
- Reads will be processed with the partition-based nanoClust pipeline — singleton
  counts from direct clustering are a negative control, not the final OTU estimate.
- Input reads have not been fragmented and orientation-corrected by chopSeq, so cluster
  membership is not comparable to the published benchmark.
- The community is environmental with unknown true richness, where 'inflation' relative
  to theoretical richness cannot be defined.
edam_operation: http://edamontology.org/operation_0282
tools:
- name: VSEARCH
  role: De novo clustering (cluster_fast) of full-length amplicon reads at 97% identity;
    its uc/clstr output is parsed to count OTUs and singletons
- name: chopSeq
  role: Upstream fragmentation and read orientation correction of INC-Seq consensus
    reads that produce the full-length reads clustered here
  repo: github.com/umerijaz/nanopore
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/gigascience/giy140
    title: 'NanoAmpli-Seq: a workflow for amplicon sequencing for mixed microbial
      communities on the nanopore sequencing platform'
  source_content_hash: sha256:b745129201a305f20609e45a7b0049d2d2a9aee22c1a6f93d5395bfd4b9421d3
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - VSEARCH
  - chopSeq
  edam_operation: http://edamontology.org/operation_0282
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 6
---

# singleton-counting-in-otu-tables

## Summary

A diagnostic benchmarking skill that counts total OTUs and singleton OTUs produced by direct de novo VSEARCH clustering of full-length nanopore amplicon reads, exposing gross OTU inflation caused by high raw-read error rates.

## When to use

Use when full-length post-chopSeq, size-selected amplicon reads are clustered directly at 97% identity and you need to quantify how far resulting OTU richness and singleton counts exceed the known mock-community richness.

## When NOT to use

- Reads will be processed with the partition-based nanoClust pipeline — singleton counts from direct clustering are a negative control, not the final OTU estimate.
- Input reads have not been fragmented and orientation-corrected by chopSeq, so cluster membership is not comparable to the published benchmark.
- The community is environmental with unknown true richness, where 'inflation' relative to theoretical richness cannot be defined.

## Inputs

- Post-chopSeq, size-selected full-length 16S rRNA amplicon reads in FASTA format (2D and 1D2 nanopore mock-community datasets)
- VSEARCH 97% identity clustering output (.uc or .clstr files)

## Outputs

- otu_counts.csv with columns: dataset, total OTUs, singleton OTU count
- Per-dataset singleton fraction used to diagnose OTU inflation

## How to apply

Load the post-chopSeq, size-selected full-length amplicon reads (FASTA) for each mock community dataset (2D and 1D2 nanopore sequencing, single-organism and 10-organism communities). Run de novo clustering with VSEARCH cluster_fast at a 97% identity threshold directly on each dataset, without reference mapping or the partition-based nanoClust step. Parse the VSEARCH uc/clstr output to count total OTUs and singleton clusters (clusters containing exactly one read) per dataset, then write a summary table (otu_counts.csv) with dataset, total OTUs, and singleton counts. Rationale: incorrect mapping and direct clustering of noisy nanopore reads is unreliable, so singleton-dominated OTU counts here demonstrate that direct full-length clustering fails and motivate the partition-based de novo clustering of nanoClust.

## Related tools

- **VSEARCH** (De novo clustering (cluster_fast) of full-length amplicon reads at 97% identity; its uc/clstr output is parsed to count OTUs and singletons)
- **chopSeq** (Upstream fragmentation and read orientation correction of INC-Seq consensus reads that produce the full-length reads clustered here) — github.com/umerijaz/nanopore

## Examples

```
vsearch --cluster_fast post_chopseq_size_selected.fasta --id 0.97 --uc otus.uc && python count_singletons.py otus.uc --out otu_counts.csv
```

## Evaluation signals

- 2D single-organism dataset yields 817 total OTUs with 777 singletons
- 1D2 single-organism dataset yields 1,301 total OTUs with 1,238 singletons
- 10-organism datasets yield 2,122 OTUs (1,742 singletons, 2D) and 2,725 OTUs (2,447 singletons, 1D2)
- Singletons dominate every cluster set (>90% of OTUs), confirming gross inflation above theoretical mock richness
- otu_counts.csv exists, is well-formed, and the four total OTU counts match reported figures (allowing minor divergence from read-set versioning)

## Limitations

- Direct 97% clustering of full-length post-chopSeq reads grossly inflates OTU counts far above true mock richness, so results are valid only as a failure benchmark, not as diversity estimates.
- Counts may diverge slightly with read-set versioning, so exact-match validation should tolerate minor differences.
- Residual nanopore errors (deletions in homopolymer regions) drive the inflation; clustering accuracy is bounded by raw-read error rates.

## Evidence

- [intro] direct clustering inflates OTUs: "incorrect mapping to a reference database is likely due to high error rates of raw nanopore reads"
- [intro] de novo alternative to reference mapping: "mapping raw, noisy reads to custom or publicly available reference databases (i.e., SILVA)"
- [abstract] partition-based clustering is the correct alternative: "nanoClust algorithm was designed for read partitioning-based de novo clustering and within cluster consensus calling"
- [abstract] expected accuracy of the full workflow being benchmarked: "NanoAmpli-Seq accurately estimates the diversity of tested mock communities with average consensus sequence accuracy of 99.5% for 2D and 1D2 sequencing"
- [abstract] residual error source driving inflation: "Nearly all residual errors in NanoAmpli-Seq sequences originate from deletions in homopolymer regions"
- [readme] upstream data provenance of clustered reads: "Scripts for processing short amplicon reads from Oxford Nanopore."
