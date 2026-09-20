---
name: partition-model-selection-phylogenetics
description: Use when a concatenated multi-locus nucleotide alignment (e.g. ~4 kb
  ribosomal sequences across divergent eukaryotes) is about to be used for Bayesian
  phylogenetic inference and a per-partition substitution model must be chosen before
  running MrBayes.
when_to_use_negative:
- The dataset is a single short locus (e.g. COI barcodes) where partitioning across
  ribosomal markers is not meaningful
- The input is already demultiplexed raw MinION reads rather than an aligned sequence
  matrix — partition-model selection presupposes a finished alignment
- The goal is abundance/species-richness metabarcoding comparison (e.g. mock-community
  Illumina vs Nanopore recovery) rather than topology reconstruction
edam_operation: http://edamontology.org/operation_0544
edam_topics:
- http://edamontology.org/topic_3293
tools:
- name: PartitionFinder
  role: Selects the best-fit partitioning scheme and substitution models for the ribosomal
    markers prior to Bayesian inference
- name: MrBayes
  role: Performs Bayesian inference under the PartitionFinder-selected models to produce
    majority-rule consensus trees
- name: MAFFT
  role: Aligns the ~4 kb rDNA datasets using its accurate iterative-refinement mode
    before partition-model selection
- name: Python
  role: Used to export and compare the MrBayes consensus trees in NEXUS/Newick format
    against deposited reference trees
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/gigascience/giz006
    title: Nanopore sequencing of long ribosomal DNA amplicons enables portable and
      simple biodiversity assessments with high phylogenetic resolution across broad
      taxonomic scale
  source_content_hash: sha256:1a8c37d4d96cd8bccc900fe1b3ac75997dbeef81e91c1291f0510d18b5c11cbc
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - MAFFT
  - MrBayes
  - PartitionFinder
  - Python
  edam_operation: http://edamontology.org/operation_0544
  edam_topics:
  - http://edamontology.org/topic_3293
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# partition-model-selection-phylogenetics

## Summary

Selects a best-fit partitioning scheme and substitution models for multi-marker nucleotide alignments (e.g. concatenated 18S/5.8S/28S rDNA or full-amplicon rDNA with ITS regions) using PartitionFinder, so that downstream Bayesian inference in MrBayes runs under an appropriate model per partition rather than a single global model.

## When to use

Use when a concatenated multi-locus nucleotide alignment (e.g. ~4 kb ribosomal sequences across divergent eukaryotes) is about to be used for Bayesian phylogenetic inference and a per-partition substitution model must be chosen before running MrBayes.

## When NOT to use

- The dataset is a single short locus (e.g. COI barcodes) where partitioning across ribosomal markers is not meaningful
- The input is already demultiplexed raw MinION reads rather than an aligned sequence matrix — partition-model selection presupposes a finished alignment
- The goal is abundance/species-richness metabarcoding comparison (e.g. mock-community Illumina vs Nanopore recovery) rather than topology reconstruction

## Inputs

- Concatenated nucleotide alignment in FASTA/NEXUS form (e.g. 3,656 bp 18S/5.8S/28S set for 117 taxa, or 4,214 bp full-amplicon rDNA with ITS for 83 spiders)
- Partition definition file specifying which alignment columns correspond to each ribosomal marker / ITS region
- Candidate-model list or scheme file for PartitionFinder

## Outputs

- Best-fit partitioning scheme and per-partition substitution models selected by PartitionFinder
- MrBayes NEXUS block configured with the selected models
- MrBayes majority-rule consensus tree in NEXUS/Newick format with posterior-probability node support

## How to apply

Build a concatenated nucleotide alignment first (e.g. with MAFFT's accurate iterative-refinement mode, appropriate for roughly 4 kb ribosomal sequences across divergent eukaryotes), keeping ITS regions if the full-amplicon dataset requires them (e.g. the 4,214 bp 83-spider full-amplicon alignment vs the 3,656 bp 117-taxon concatenated 18S/5.8S/28S set). Define partition boundaries corresponding to the individual ribosomal markers (and ITS regions) and supply the alignment plus a scheme file to PartitionFinder, which evaluates schemes and selects the best-fit partitioning scheme and substitution models. Feed the PartitionFinder-selected models into MrBayes (e.g. via a NEXUS block) and run Bayesian inference separately for each dataset to obtain majority-rule consensus trees. Decide whether to analyze datasets jointly or as separate analyses based on marker composition: here the 117-taxon concatenated rDNA dataset and the 83-spider ITS-retaining dataset were run as two separate MrBayes analyses. Correctness is judged by exporting the MrBayes consensus trees in NEXUS/Newick format and comparing the inferred topologies node-by-node against deposited reference trees.

## Related tools

- **PartitionFinder** (Selects the best-fit partitioning scheme and substitution models for the ribosomal markers prior to Bayesian inference)
- **MrBayes** (Performs Bayesian inference under the PartitionFinder-selected models to produce majority-rule consensus trees)
- **MAFFT** (Aligns the ~4 kb rDNA datasets using its accurate iterative-refinement mode before partition-model selection)
- **Python** (Used to export and compare the MrBayes consensus trees in NEXUS/Newick format against deposited reference trees)

## Examples

```
mafft --auto rDNA_concat_117taxa.fasta > aligned.fasta  # then PartitionFinder on aligned.fasta + partition scheme, then MrBayes with the selected models
```

## Evaluation signals

- The inferred MrBayes consensus topology reproduces the clade structure of the deposited Newick reference tree for the 117-taxon concatenated rDNA dataset
- The inferred MrBayes consensus topology reproduces the clade structure of the deposited Newick reference tree for the 83-spider 4,214 bp full-amplicon dataset (ITS retained)
- Node supports (posterior probabilities) on the spider phylogeny are strongly supported and the overall topology agrees with the most recent phylogenetic work on spiders
- Each dataset was run as a separate MrBayes analysis under the PartitionFinder-selected models, and consensus trees were exported in NEXUS/Newick format for comparison

## Limitations

- No quantitative topological metric is reported in the source; reproducibility is assessed qualitatively against deposited Newick reference trees rather than by a numeric distance
- Long rDNA sequences recover skewed diversity estimates in bulk-sample contexts, so this phylogenetic skill should not be conflated with reliable metabarcoding abundance analysis
- Alignment quality of divergent eukaryote ribosomal sequences governs the result; MAFFT's iterative-refinement mode is specifically recommended for the ~4 kb rDNA scale

## Evidence

- [other] Align each dataset with MAFFT using its accurate iterative-refinement mode, which is appropriate for roughly 4 kb ribosomal sequences across divergent eukaryotes.: [evidence span withheld — anchor status: not_found]
- [other] Run PartitionFinder on each alignment to select the best-fit partitioning scheme and substitution models for the ribosomal markers.: [evidence span withheld — anchor status: not_found]
- [results] The spider phylogeny from the 83-taxon, 4,214 bp rDNA alignment (including both ITS regions) was strongly supported, and its overall tree topology agreed with the most recent phylogenetic work on spiders.: [evidence span withheld — anchor status: not_found]
- [other] Export the MrBayes consensus trees in NEXUS/Newick format.: [evidence span withheld — anchor status: not_found]
