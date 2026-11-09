---
name: newick-tree-topology-comparison
description: Use when a reproduced Bayesian phylogeny (e.g. MrBayes consensus from
  MAFFT-aligned rDNA alignments) must be checked for agreement with a deposited reference
  Newick tree to confirm reproducibility of the topology.
when_to_use_negative:
- When a quantitative topological distance is required (the source reports no metric,
  so only qualitative clade-structure agreement is supported)
- When the reference tree is unavailable in Newick format — without a deposited tree
  file there is nothing to compare node-by-node
- When the goal is taxonomic (barcode/species-ID) assessment of the MinION rDNA reads
  rather than reproduction of the published phylogenies
edam_topics: []
tools:
- name: MrBayes
  role: Performs Bayesian inference under PartitionFinder-selected models and emits
    the majority-rule consensus trees that are compared to the deposited Newick references
- name: MAFFT
  role: Aligns the deposited rDNA datasets using its accurate iterative-refinement
    mode before tree inference
- name: PartitionFinder
  role: Selects the best-fit partitioning scheme and substitution models for the ribosomal
    markers used in the MrBayes runs
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/gigascience/giz006
    title: Nanopore sequencing of long ribosomal DNA amplicons enables portable and
      simple biodiversity assessments with high phylogenetic resolution across broad
      taxonomic scale
  source_content_hash: sha256:00b1e7f5a93a5ceb295af3212fee79915aadc89df00f07d2230bb1283fba9fd0
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - MAFFT
  - MrBayes
  - PartitionFinder
  - Python
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# Newick tree topology comparison

## Summary

Validates a phylogenetic inference rerun by comparing the inferred MrBayes majority-rule consensus trees (exported as NEXUS/Newick) node-by-node against reference Newick trees deposited with the original publication, in this case the eukaryote and spider rDNA topologies from Krehenwinkel et al. 2019 (GigaScience).

## When to use

Use when a reproduced Bayesian phylogeny (e.g. MrBayes consensus from MAFFT-aligned rDNA alignments) must be checked for agreement with a deposited reference Newick tree to confirm reproducibility of the topology.

## When NOT to use

- When a quantitative topological distance is required (the source reports no metric, so only qualitative clade-structure agreement is supported)
- When the reference tree is unavailable in Newick format — without a deposited tree file there is nothing to compare node-by-node
- When the goal is taxonomic (barcode/species-ID) assessment of the MinION rDNA reads rather than reproduction of the published phylogenies

## Inputs

- Inferred MrBayes majority-rule consensus tree in NEXUS/Newick format
- Reference Newick tree files deposited in the GigaScience repository for the 117-taxon concatenated rDNA and 83-spider full-amplicon datasets
- PartitionFinder-selected partitioning scheme and substitution models for each rDNA alignment

## Outputs

- Per-dataset node-by-node comparison of inferred vs. deposited consensus tree clade structure
- Qualitative reproducibility verdict (agrees / disagrees) for each of the two rDNA topologies

## How to apply

Run the inference pipeline first: align the deposited rDNA datasets (the 3,656 bp concatenated 18S/5.8S/28S set for 117 taxa and the 4,214 bp full-amplicon set with ITS retained for 83 spiders) with MAFFT iterative refinement, select partitions and models with PartitionFinder, and run MrBayes as separate analyses per dataset. Export the resulting majority-rule consensus trees in NEXUS/Newick format, then parse both the inferred and the deposited reference Newick files and compare their clade structure node-by-node. Agreement is assessed qualitatively — the acceptance criterion is that the inferred consensus topologies reproduce the clade structure of the deposited reference trees, since no quantitative topological metric (e.g. Robinson–Foulds distance) is reported in the source. Perform the comparison separately for each dataset rather than pooling, so that a mismatch can be attributed to a specific alignment.

## Related tools

- **MrBayes** (Performs Bayesian inference under PartitionFinder-selected models and emits the majority-rule consensus trees that are compared to the deposited Newick references)
- **MAFFT** (Aligns the deposited rDNA datasets using its accurate iterative-refinement mode before tree inference)
- **PartitionFinder** (Selects the best-fit partitioning scheme and substitution models for the ribosomal markers used in the MrBayes runs)

## Evaluation signals

- Inferred MrBayes consensus topology reproduces the clade structure of the deposited Newick reference tree for the 117-taxon concatenated rDNA dataset
- Inferred MrBayes consensus topology reproduces the clade structure of the deposited Newick reference tree for the 83-spider, 4,214 bp full-amplicon dataset
- Spider topology shows strong posterior support and overall agreement with the most recent phylogenetic work on spiders
- Both datasets were run as separate analyses and each consensus tree was exported in NEXUS/Newick before comparison
- Any node that differs from the reference tree can be localized to a specific alignment (concatenated vs. full-amplicon) rather than to the comparison procedure

## Limitations

- No quantitative topological metric is reported in the source text, so agreement is assessed only qualitatively against the deposited reference trees
- The paper's own finding is that long rDNA metabarcoding recovers highly skewed diversity estimates in bulk samples, so tree-based conclusions should be restricted to the phylogenetic (not abundance) layer
- Source of taxonomic bias in bulk samples is undetermined; further research is required before long rDNA amplicon sequencing is reliable for bulk sample analysis

## Evidence

- [methods] node-by-node comparison against deposited references: [evidence span withheld — anchor status: not_found]
- [methods] qualitative acceptance criterion: [evidence span withheld — anchor status: not_found]
- [results] spider topology agreement with recent work: [evidence span withheld — anchor status: not_found]
- [methods] separate analyses per dataset: [evidence span withheld — anchor status: not_found]
- [discussion] bias caveat: "Further research into the sources of the observed bias is required before long rDNA amplicon sequencing can be utilized as a reliable resource for the analysis of bulk samples."
