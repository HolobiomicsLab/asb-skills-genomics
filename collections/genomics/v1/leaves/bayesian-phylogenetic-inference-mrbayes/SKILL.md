---
name: bayesian-phylogenetic-inference-mrbayes
description: Use when you need a well-supported tree topology from a multi-kilobase
  ribosomal DNA alignment (concatenated 18S/5.8S/28S or full-amplicon rDNA with ITS),
  and a reference topology exists for node-by-node comparison.
when_to_use_negative:
- The goal is species-abundance or metabarcoding recovery from bulk arthropod community
  samples — the source work found long rDNA amplicon sequencing recovers highly skewed
  diversity estimates and is not yet reliable for bulk sample analysis
- The marker is short mitochondrial COI rather than multi-kilobase rDNA — the paper
  treats COI barcoding as a separate taxonomic-resolution comparison
- Input is raw dual-indexed Nanopore read output that first needs demultiplexing with
  MiniBar before consensus sequences exist to align
edam_operation: http://edamontology.org/operation_0323
edam_topics:
- http://edamontology.org/topic_0194
tools:
- name: MAFFT
  role: Aligns the ~4 kb rDNA datasets using accurate iterative-refinement mode appropriate
    for divergent eukaryotes
- name: PartitionFinder
  role: Selects the best-fit partitioning scheme and substitution models for the ribosomal
    markers prior to MrBayes
- name: MrBayes
  role: Performs Bayesian inference under the PartitionFinder-selected models to obtain
    majority-rule consensus trees, run separately per dataset
- name: Python
  role: Used for handling/exporting the consensus trees and comparing topologies node-by-node
    against deposited references
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/gigascience/giz006
    title: Nanopore sequencing of long ribosomal DNA amplicons enables portable and
      simple biodiversity assessments with high phylogenetic resolution across broad
      taxonomic scale
  source_content_hash: sha256:7b1762d49190c265ae1a5864bb433e865d3e80ede800783e0ac7abd61e94fffa
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
  - http://edamontology.org/topic_0194
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# bayesian-phylogenetic-inference-mrbayes

## Summary

Builds Bayesian majority-rule consensus phylogenies from long ribosomal DNA (rDNA) alignments (e.g., 18S/5.8S/28S with ITS regions retained) using MAFFT alignment, PartitionFinder model/partition selection, and MrBayes inference, then validates the resulting trees against deposited Newick references.

## When to use

Use when you need a well-supported tree topology from a multi-kilobase ribosomal DNA alignment (concatenated 18S/5.8S/28S or full-amplicon rDNA with ITS), and a reference topology exists for node-by-node comparison.

## When NOT to use

- The goal is species-abundance or metabarcoding recovery from bulk arthropod community samples — the source work found long rDNA amplicon sequencing recovers highly skewed diversity estimates and is not yet reliable for bulk sample analysis
- The marker is short mitochondrial COI rather than multi-kilobase rDNA — the paper treats COI barcoding as a separate taxonomic-resolution comparison
- Input is raw dual-indexed Nanopore read output that first needs demultiplexing with MiniBar before consensus sequences exist to align

## Inputs

- Concatenated 18S/5.8S/28S rDNA sequence set (~3,656 bp, 117 taxa) in FASTA
- Full-amplicon rDNA set retaining both ITS regions (~4,214 bp, 83 spider taxa) in FASTA
- Deposited Newick/NEXUS reference tree files from the GigaScience repository

## Outputs

- MrBayes majority-rule consensus tree in NEXUS/Newick format per dataset
- Posterior support values on consensus clades
- PartitionFinder-selected partitioning scheme and substitution models per alignment

## How to apply

Start from the deposited rDNA sequence sets (here, a 3,656 bp concatenated 18S/5.8S/28S set for 117 taxa and a 4,214 bp full-amplicon set retaining both ITS regions for 83 spiders) and align each with MAFFT in its accurate iterative-refinement mode, which suits ~4 kb ribosomal sequences across divergent eukaryotes. Run PartitionFinder on each alignment to select the best-fit partitioning scheme and substitution models for the ribosomal markers rather than assuming a single model. Perform Bayesian inference in MrBayes under the PartitionFinder-selected models, running each dataset as a separate analysis, and generate majority-rule consensus trees. Export the consensus trees in NEXUS/Newick format and compare them node-by-node against the deposited Newick reference trees; the acceptance criterion is that the inferred consensus topologies reproduce the clade structure of the deposited references for both datasets.

## Related tools

- **MAFFT** (Aligns the ~4 kb rDNA datasets using accurate iterative-refinement mode appropriate for divergent eukaryotes)
- **PartitionFinder** (Selects the best-fit partitioning scheme and substitution models for the ribosomal markers prior to MrBayes)
- **MrBayes** (Performs Bayesian inference under the PartitionFinder-selected models to obtain majority-rule consensus trees, run separately per dataset)
- **Python** (Used for handling/exporting the consensus trees and comparing topologies node-by-node against deposited references)

## Examples

```
mb spider_rDNA_partitioned.nex  # NEXUS file carries the MAFFT alignment, PartitionFinder partition/model block, and mcmc/sumt settings
```

## Evaluation signals

- The inferred MrBayes consensus topology for the 117-taxon concatenated rDNA dataset reproduces the clade structure of the deposited Newick reference tree
- The inferred consensus topology for the 83-taxon, 4,214 bp full-amplicon (ITS-retained) dataset reproduces the clade structure of its deposited Newick reference
- Overall spider tree topology agrees with the most recent phylogenetic work on spiders
- Major clades carry strong posterior support in the majority-rule consensus
- Trees are exported in valid NEXUS/Newick format and can be parsed for node-by-node comparison

## Limitations

- No quantitative topological metric is reported in the source text; reproducibility is assessed qualitatively against the deposited Newick references
- Long rDNA amplicon data show highly skewed diversity estimates for bulk community samples, so tree-based conclusions should not be extended to abundance inference
- The two datasets differ in taxon sampling (117 taxa concatenated vs. 83 spiders full-amplicon), so topologies are validated per dataset rather than pooled

## Evidence

- [full_text] MAFFT iterative-refinement alignment of ~4 kb rDNA: [evidence span withheld — anchor status: not_found]
- [full_text] PartitionFinder model selection then MrBayes on separate datasets: [evidence span withheld — anchor status: not_found]
- [full_text] Accepted topology result: [evidence span withheld — anchor status: not_found]
- [abstract] Phylogenetic resolution of long rDNA: "We find excellent phylogenetic and taxonomic resolution offered by long rDNA sequences across broad taxonomic scales"
- [discussion] Bias caveat for bulk samples: "Further research into the sources of the observed bias is required before long rDNA amplicon sequencing can be utilized as a reliable resource for the analysis of bulk samples."
