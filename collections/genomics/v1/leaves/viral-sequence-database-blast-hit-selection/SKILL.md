---
name: viral-sequence-database-blast-hit-selection
description: Use when consensus viral genomes have been reconstructed from nanopore
  reads (FASTQ/FAST5/POD5 input processed by VirDetector) and the virus species in
  the sample must be identified and used to seed a phylogenetic tree.
when_to_use_negative:
- Input files are raw reads or FAST5/POD5 that have not yet been preprocessed, classified
  and assembled into consensus genomes — the BLAST step starts from consensus FASTA
- The virus of interest is absent from the VirDetector phylogenetic database (database/sequences.fasta),
  since both detection and the sequence selection for tree inference depend on it
- A single-reference-based placement (e.g. read mapping against a known reference)
  is already sufficient and no phylogenetic tree with database sequences is required
edam_operation: http://edamontology.org/operation_0338
edam_topics:
- http://edamontology.org/topic_0084
- http://edamontology.org/topic_0091
tools:
- name: VirDetector
  role: Pipeline that hosts the phylogenetic database, performs BLAST-based virus
    detection and orchestrates the downstream tree inference
  repo: https://github.com/NLKaiser/VirDetector
- name: Nextflow
  role: Workflow engine executing the downstream pipeline (nextflow run main.nf)
  repo: https://github.com/NLKaiser/VirDetector
- name: seqmagick
  role: Converts the cleaned multiple alignment to NEXUS format for MrBayes
  repo: https://github.com/fhcrc/seqmagick
- name: FigTree
  role: Graphical viewer used to display the annotated NEXUS tree files
  repo: https://github.com/rambaut/figtree
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bioinformatics/btaf029
    title: 'VirDetector: a bioinformatic pipeline for virus surveillance using nanopore
      sequencing'
  source_content_hash: sha256:fd7f93abbf7ac2608ae20f53b78d6ae35ccfd9a8f726a157c205bba63e3ee6ae
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/NLKaiser/VirDetector
  tools:
  - BLAST
  - FigTree
  - IQ-TREE2
  - MAFFT
  - MUSCLE
  - MrBayes
  - Nextflow
  - VirDetector
  - seqmagick
  edam_operation: http://edamontology.org/operation_0338
  edam_topics:
  - http://edamontology.org/topic_0084
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 12
---

# viral-sequence-database-blast-hit-selection

## Summary

Selects the virus present in a nanopore-sequenced sample by blasting the reconstructed consensus genomes against the VirDetector reference database (database/sequences.fasta) and taking the highest bit-score hit, then gathers all database sequences of that virus for downstream maximum-likelihood (IQ-TREE2) and Bayesian (MrBayes) phylogenetic inference.

## When to use

Use when consensus viral genomes have been reconstructed from nanopore reads (FASTQ/FAST5/POD5 input processed by VirDetector) and the virus species in the sample must be identified and used to seed a phylogenetic tree.

## When NOT to use

- Input files are raw reads or FAST5/POD5 that have not yet been preprocessed, classified and assembled into consensus genomes — the BLAST step starts from consensus FASTA
- The virus of interest is absent from the VirDetector phylogenetic database (database/sequences.fasta), since both detection and the sequence selection for tree inference depend on it
- A single-reference-based placement (e.g. read mapping against a known reference) is already sufficient and no phylogenetic tree with database sequences is required

## Inputs

- Reconstructed viral consensus genomes (FASTA) from the VirDetector assembly stage
- VirDetector phylogenetic reference database (database/sequences.fasta)
- Optional metadata file (.txt) with sample names, lineage, country and collection date
- Outgroup accession from database/sequences.fasta

## Outputs

- Identified virus (highest bit-score BLAST hit) for the sample
- Cleaned multiple alignment (CIAlign-ed, MAFFT/MUSCLE) converted to NEXUS
- Maximum-likelihood tree (IQ-TREE2) and Bayesian tree (MrBayes)
- Annotated NEXUS tree files for FigTree visualisation carrying virus name, lineage, country and collection date

## How to apply

Blast all sample consensus genomes against the VirDetector database (database/sequences.fasta) and select as the detected virus the hit with the highest bit-score; segmented genomes are treated as independent entries in the database. Then collect every database sequence whose header carries the same virus field as the hit and pool them with the sample consensus genomes for alignment with MAFFT (faster) or MUSCLE (more precise, more compute) as set by params.alignment_tool. The raw multiple alignment is cleaned with CIAlign, converted to NEXUS with seqmagick, and an outgroup is chosen by accession from database/sequences.fasta (e.g. NC_009942.1 for West Nile virus via params.iqtree_args = '-o "NC_009942.1"' and 'outgroup NC_009942.1' for MrBayes). A substitution model is selected with IQ-TREE2 restricted to MrBayes-compatible models, after which the maximum-likelihood (IQ-TREE2) and Bayesian (MrBayes) trees are inferred and orchestrated by Nextflow via 'nextflow run main.nf'.

## Related tools

- **VirDetector** (Pipeline that hosts the phylogenetic database, performs BLAST-based virus detection and orchestrates the downstream tree inference) — https://github.com/NLKaiser/VirDetector
- **Nextflow** (Workflow engine executing the downstream pipeline (nextflow run main.nf)) — https://github.com/NLKaiser/VirDetector
- **seqmagick** (Converts the cleaned multiple alignment to NEXUS format for MrBayes) — https://github.com/fhcrc/seqmagick
- **FigTree** (Graphical viewer used to display the annotated NEXUS tree files) — https://github.com/rambaut/figtree

## Examples

```
nextflow run main.nf
```

## Evaluation signals

- The BLAST report shows a single unambiguous top hit: the selected virus has the strictly largest bit-score among all database matches for the sample consensus genome
- All database entries sharing the selected virus field in their header (segmented genomes included) are present in the input alignment alongside the consensus sequence
- The cleaned alignment is in NEXUS format and contains the specified outgroup accession (e.g. NC_009942.1) so IQ-TREE2 (-o) and MrBayes (outgroup) root correctly
- IQ-TREE2 reports a substitution model that is MrBayes-compatible, and both the ML and Bayesian tree files are produced
- Tree tip labels carry virus name, lineage, country and collection date as encoded in the sequence headers/metadata file

## Limitations

- Detection is limited to viruses contained in the VirDetector phylogenetic database (database/sequences.fasta); segmented genomes are treated as independent entries, so per-segment hits must be aggregated by the virus field
- If a metadata file is specified, only samples defined in that file are passed on to subsequent processing steps, so samples missing from the metadata are silently dropped
- Alignment tool choice is a trade-off: MAFFT is faster while MUSCLE is more precise but requires more computing resources
- Without setting an explicit outgroup (params.iqtree_args / params.mrbayes_args), the inferred trees are not rooted on a chosen reference

## Evidence

- [intro] VirDetector then detects the virus present in the sample and uses a selection of sequences to infer phylogenetic trees, separately using maximum likelihood and bayesian inference.: "VirDetector then detects the virus present in the sample and uses a selection of sequences to infer phylogenetic trees, separately using maximum likelihood and bayesian inference."
- [methods] VirDetector database contains the phylogenetic reference sequences: "The VirDetector database for phylogenetic inference currently contains the viruses:"
- [readme] Outgroup selection example for West Nile virus: "You have samples of the West Nile Virus and selected the sequence NC_009942.1 as outgroup:"
- [readme] Alignment tool trade-off between MAFFT and MUSCLE: "While MAFFT is generally faster, MUSCLE is more precise but requires more computing resources."
- [readme] Metadata file filtering of samples: "If you specify a metadata file, only those samples are passed on to subsequent processing steps that are defined in this file!"
