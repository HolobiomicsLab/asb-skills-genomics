---
name: maximum-likelihood-phylogenetic-inference
description: Use when nanopore-sequenced samples have consensus genomes reconstructed
  and you need to place the detected virus phylogenetically, selecting the virus by
  highest BLAST bit-score against database/sequences.fasta and inferring ML and Bayesian
  trees.
when_to_use_negative:
- The detected virus has no additional database entries in database/sequences.fasta
  (only one sequence), so there are too few taxa for a meaningful tree.
- You only need taxonomic classification or structural variant detection, not phylogenetic
  placement of the consensus genome.
- Input is already an annotated, publication-ready tree (NEXUS) and the goal is only
  visualisation in FigTree rather than inference.
edam_operation: http://edamontology.org/operation_0323
edam_topics:
- http://edamontology.org/topic_0084
tools:
- name: VirDetector
  role: pipeline that orchestrates virus selection by BLAST bit-score, alignment,
    tree inference and annotation via nextflow run main.nf
  repo: https://github.com/NLKaiser/VirDetector
- name: BLAST
  role: selects the detected virus as the highest bit-score hit of the sample consensus
    genome against the database
- name: MAFFT
  role: faster option for aligning database and consensus sequences
- name: MUSCLE
  role: more precise alignment option requiring more computing resources, chosen via
    params.alignment_tool
- name: CIAlign
  role: cleans the raw multiple alignment to obtain the edited multiple alignment
- name: seqmagick
  role: converts the cleaned alignment to NEXUS format
- name: IQ-TREE2
  role: substitution-model selection (restricted to MrBayes-compatible models) and
    maximum-likelihood tree inference with outgroup set via params.iqtree_args
- name: MrBayes
  role: Bayesian inference tree with outgroup set via params.mrbayes_args
- name: FigTree
  role: visualisation of the annotated NEXUS tree files
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bioinformatics/btaf029
    title: 'VirDetector: a bioinformatic pipeline for virus surveillance using nanopore
      sequencing'
  source_content_hash: sha256:30300bda832de5ad106897d420581a60595cf97e0c2d59e04e789558c8015592
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/NLKaiser/VirDetector
  tools:
  - BLAST
  - CIAlign
  - FigTree
  - IQ-TREE2
  - MAFFT
  - MUSCLE
  - MrBayes
  - VirDetector
  - seqmagick
  edam_operation: http://edamontology.org/operation_0323
  edam_topics:
  - http://edamontology.org/topic_0084
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5454545454545454
n_claims: 11
---

# maximum-likelihood-phylogenetic-inference

## Summary

Infers phylogenetic trees from reconstructed viral consensus genomes within VirDetector's downstream analysis, running maximum-likelihood (IQ-TREE2) and Bayesian (MrBayes) inference in parallel on a cleaned multiple alignment of database and sample sequences.

## When to use

Use when nanopore-sequenced samples have consensus genomes reconstructed and you need to place the detected virus phylogenetically, selecting the virus by highest BLAST bit-score against database/sequences.fasta and inferring ML and Bayesian trees.

## When NOT to use

- The detected virus has no additional database entries in database/sequences.fasta (only one sequence), so there are too few taxa for a meaningful tree.
- You only need taxonomic classification or structural variant detection, not phylogenetic placement of the consensus genome.
- Input is already an annotated, publication-ready tree (NEXUS) and the goal is only visualisation in FigTree rather than inference.

## Inputs

- viral consensus genomes reconstructed by VirDetector from nanopore reads (FASTQ/FAST5/POD5 input, or FASTA used from the alignment step onwards)
- VirDetector database sequences (database/sequences.fasta) for the detected virus
- params.alignment_tool setting ('mafft' or 'muscle') from nextflow.config
- outgroup accession (e.g. NC_009942.1) via params.iqtree_args and params.mrbayes_args

## Outputs

- maximum-likelihood tree (IQ-TREE2)
- Bayesian inference tree (MrBayes)
- cleaned multiple alignment from CIAlign
- NEXUS-format tree files annotated with virus name, lineage, country and collection date for FigTree visualisation

## How to apply

First blast all sample consensus genomes against the VirDetector database (database/sequences.fasta) and take the highest bit-score hit as the detected virus; collect all database sequences of that virus (segmented genomes treated as independent entries) plus the consensus genomes. Align them with MAFFT (faster) or MUSCLE (more precise, more resources) as set via params.alignment_tool, then clean the raw alignment with CIAlign. Convert the cleaned alignment to NEXUS with seqmagick and set the outgroup (e.g. params.iqtree_args = "-o \"NC_009942.1\"" for IQ-TREE2, 'outgroup NC_009942.1' for MrBayes). Select a substitution model with IQ-TREE2 restricted to MrBayes-compatible models, then infer the maximum-likelihood tree with IQ-TREE2 and the Bayesian tree with MrBayes, orchestrated by Nextflow (nextflow run main.nf). Finally annotate the NEXUS tree files with virus name, lineage, country and collection date parsed from sequence headers for FigTree visualisation.

## Related tools

- **VirDetector** (pipeline that orchestrates virus selection by BLAST bit-score, alignment, tree inference and annotation via nextflow run main.nf) — https://github.com/NLKaiser/VirDetector
- **BLAST** (selects the detected virus as the highest bit-score hit of the sample consensus genome against the database)
- **MAFFT** (faster option for aligning database and consensus sequences)
- **MUSCLE** (more precise alignment option requiring more computing resources, chosen via params.alignment_tool)
- **CIAlign** (cleans the raw multiple alignment to obtain the edited multiple alignment)
- **seqmagick** (converts the cleaned alignment to NEXUS format)
- **IQ-TREE2** (substitution-model selection (restricted to MrBayes-compatible models) and maximum-likelihood tree inference with outgroup set via params.iqtree_args)
- **MrBayes** (Bayesian inference tree with outgroup set via params.mrbayes_args)
- **FigTree** (visualisation of the annotated NEXUS tree files)

## Examples

```
nextflow run main.nf
```

## Evaluation signals

- The BLAST hit with the largest bit-score against database/sequences.fasta identifies the virus used for phylogenetic analysis; verify the selected virus matches this criterion.
- The cleaned alignment exists and was produced by CIAlign from the MAFFT/MUSCLE alignment of database-plus-consensus sequences.
- The NEXUS file contains a stated outgroup (e.g. NC_009942.1) consistent with params.iqtree_args / params.mrbayes_args.
- Two trees are produced — one maximum-likelihood (IQ-TREE2) and one Bayesian (MrBayes) — as the workflow infers them separately.
- Tree tip labels carry virus name, lineage, country and collection date parsed from the sequence headers, enabling FigTree visualisation.

## Limitations

- Segmented genomes are treated as independent database entries, so all segments of the detected virus are collected for the alignment.
- MAFFT is generally faster while MUSCLE is more precise but requires more computing resources; the choice via params.alignment_tool affects runtime and accuracy.
- If a metadata file is specified, only samples defined in that file are passed to subsequent processing steps, which can silently drop samples from the tree.
- Setting the outgroup is a recommendation, not enforced; omitting it may compromise tree rooting.

## Evidence

- [intro] Virus selection by bit-score and dual tree inference: "VirDetector then detects the virus present in the sample and uses a selection of sequences to infer phylogenetic trees, separately using maximum likelihood and bayesian inference."
- [readme] Outgroup recommendation: "If you want to execute a phylogenetic analysis, we recommend setting the outgroup for the specific tool."
- [readme] Alignment tool trade-off: "While MAFFT is generally faster, MUSCLE is more precise but requires more computing resources."
- [methods] Concrete outgroup example: "You have samples of the West Nile Virus and selected the sequence NC_009942.1 as outgroup:"
