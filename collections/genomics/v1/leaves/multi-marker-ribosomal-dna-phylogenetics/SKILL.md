---
name: multi-marker-ribosomal-dna-phylogenetics
description: Use when building or reproducing a phylogeny from long (~3.5-4.2 kb)
  multi-marker rDNA alignments spanning divergent eukaryotes, and deposited reference
  Newick trees are available for topology validation.
when_to_use_negative:
- Input is not ribosomal DNA (e.g. mitochondrial COI barcodes) — the paper compared
  rDNA resolution against COI, and this skill is specific to the rRNA cluster
- You need quantitative (as opposed to qualitative) topology comparison — the source
  validation only assessed agreement against deposited reference trees without a topological
  metric
- The goal is bulk-sample metabarcoding abundance estimation — the article found long
  rDNA amplicon metabarcoding recovers highly skewed diversity estimates and is not
  yet reliable for bulk sample analysis
edam_operation: http://edamontology.org/operation_0323
edam_topics:
- http://edamontology.org/topic_3293
tools:
- name: MAFFT
  role: Aligns each ~4 kb multi-marker rDNA dataset using accurate iterative-refinement
    mode
- name: PartitionFinder
  role: Selects the best-fit partitioning scheme and substitution models for the ribosomal
    markers prior to Bayesian inference
- name: MrBayes
  role: Performs Bayesian inference under PartitionFinder-selected models to produce
    majority-rule consensus trees
- name: Python
  role: Scripting glue for dataset handling and node-by-node topology comparison against
    deposited Newick files
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/gigascience/giz006
    title: Nanopore sequencing of long ribosomal DNA amplicons enables portable and
      simple biodiversity assessments with high phylogenetic resolution across broad
      taxonomic scale
  source_content_hash: sha256:43dff9a1736f9051f4a34c6f201f8f3143e0130dbdba504a8f8f6eca6e3e51ef
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - MAFFT
  - MrBayes
  - PartitionFinder
  - Python
  edam_operation: http://edamontology.org/operation_0323
  edam_topics:
  - http://edamontology.org/topic_3293
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# multi-marker-ribosomal-dna-phylogenetics

## Summary

Reconstructs Bayesian phylogenies from multi-marker ribosomal DNA alignments (concatenated 18S/5.8S/28S, optionally retaining ITS1/ITS2) by combining MAFFT alignment, PartitionFinder partition/model selection, and MrBayes inference, as demonstrated on the 117-taxon 3,656 bp and 83-spider 4,214 bp rDNA datasets from Krehenwinkel et al. (2019, GigaScience).

## When to use

Use when building or reproducing a phylogeny from long (~3.5-4.2 kb) multi-marker rDNA alignments spanning divergent eukaryotes, and deposited reference Newick trees are available for topology validation.

## When NOT to use

- Input is not ribosomal DNA (e.g. mitochondrial COI barcodes) — the paper compared rDNA resolution against COI, and this skill is specific to the rRNA cluster
- You need quantitative (as opposed to qualitative) topology comparison — the source validation only assessed agreement against deposited reference trees without a topological metric
- The goal is bulk-sample metabarcoding abundance estimation — the article found long rDNA amplicon metabarcoding recovers highly skewed diversity estimates and is not yet reliable for bulk sample analysis

## Inputs

- GigaScience-deposited consensus rDNA dataset: 3,656 bp concatenated 18S/5.8S/28S sequences for 117 taxa
- GigaScience-deposited consensus rDNA dataset: 4,214 bp full-amplicon sequences retaining ITS regions for 83 spiders
- Deposited Newick reference tree files for both datasets
- FASTA/aligned sequence files of ribosomal markers from divergent eukaryote taxa

## Outputs

- MAFFT multiple sequence alignments of each rDNA dataset
- PartitionFinder-selected partitioning schemes and substitution models
- MrBayes majority-rule consensus trees in NEXUS/Newick format (one per dataset)

## How to apply

Download the deposited consensus rDNA datasets — here the 3,656 bp concatenated 18S/5.8S/28S set for 117 taxa and the 4,214 bp full-amplicon set with ITS regions retained for 83 spiders — from the GigaScience repository. Align each dataset independently with MAFFT in its accurate iterative-refinement mode, which suits roughly 4 kb ribosomal sequences across divergent eukaryotes. Run PartitionFinder on each alignment to select the best-fit partitioning scheme and substitution models for the ribosomal markers. Perform Bayesian inference in MrBayes under the PartitionFinder-selected models, running the two datasets as separate analyses, and export the majority-rule consensus trees in NEXUS/Newick format. Validate by comparing the inferred topologies node-by-node against the deposited Newick reference trees; agreement is assessed qualitatively because no quantitative topological metric was reported in the source text.

## Related tools

- **MAFFT** (Aligns each ~4 kb multi-marker rDNA dataset using accurate iterative-refinement mode)
- **PartitionFinder** (Selects the best-fit partitioning scheme and substitution models for the ribosomal markers prior to Bayesian inference)
- **MrBayes** (Performs Bayesian inference under PartitionFinder-selected models to produce majority-rule consensus trees)
- **Python** (Scripting glue for dataset handling and node-by-node topology comparison against deposited Newick files)

## Evaluation signals

- Inferred MrBayes consensus topology for the 117-taxon concatenated rDNA dataset reproduces the clade structure of the deposited Newick reference tree
- Inferred consensus topology for the 83-spider 4,214 bp full-amplicon (ITS-retained) dataset matches the deposited reference tree node-by-node
- Tree topology is well supported and consistent with the most recent phylogenetic work on spiders
- Each of the two datasets was run as a separate MrBayes analysis (not pooled)
- Output trees are exported in valid NEXUS/Newick format

## Limitations

- No quantitative topological metric was reported in the source text, so reproducibility is judged qualitatively against deposited reference trees
- The paper found long rDNA sequences recover highly skewed diversity estimates in bulk samples, so this phylogenetic skill does not extend to quantitative metabarcoding
- Sources of the observed taxonomic bias in bulk samples are undetermined and require further research before reliable bulk-sample use

## Evidence

- [methods] Align each dataset with MAFFT using its accurate iterative-refinement mode, which is appropriate for roughly 4 kb ribosomal sequences across divergent eukaryotes.: [evidence span withheld — anchor status: not_found]
- [methods] MrBayes under PartitionFinder-selected models, separate analyses per dataset: [evidence span withheld — anchor status: not_found]
- [results] Spider phylogeny reproducibility finding: [evidence span withheld — anchor status: not_found]
- [abstract] Long rDNA phylogenetic resolution finding: "We find excellent phylogenetic and taxonomic resolution offered by long rDNA sequences across broad taxonomic scales"
- [methods] Deposited reference trees used for comparison: [evidence span withheld — anchor status: not_found]
