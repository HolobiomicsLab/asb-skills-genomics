---
name: phylogenetic-substitution-model-selection
description: Use when a cleaned multiple alignment of consensus genomes plus database
  sequences must be converted into phylogenetic trees under both maximum-likelihood
  and Bayesian inference, and a substitution model compatible with both programs is
  required.
when_to_use_negative:
- Input is raw reads or unassembled contigs rather than a cleaned multiple alignment
  — the model-selection step presupposes prior alignment and CIAlign cleaning
- Only one inference framework will be used and strict IQ-TREE2/MrBayes model compatibility
  is not required — the MrBayes-compatible restriction unnecessarily narrows the candidate
  models
- The virus detected by BLAST bit-score is not represented in database/sequences.fasta,
  so no database sequences are available to enrich the alignment
edam_topics:
- http://edamontology.org/topic_0182
tools:
- name: IQ-TREE2
  role: Performs the substitution-model selection on the NEXUS alignment, restricted
    to MrBayes-compatible models, and subsequently infers the maximum-likelihood tree
    with the chosen model and outgroup via params.iqtree_args
- name: MrBayes
  role: Consumes the same NEXUS alignment and the IQ-TREE2-selected model to infer
    the Bayesian tree; outgroup set via params.mrbayes_args
- name: seqmagick
  role: Converts the CIAlign-cleaned alignment to NEXUS format as required for MrBayes
    and outgroup specification
  repo: https://github.com/fhcrc/seqmagick
- name: Nextflow
  role: Orchestrates the model-selection and dual tree-inference steps through nextflow
    run main.nf
  repo: https://github.com/NLKaiser/VirDetector
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bioinformatics/btaf029
    title: 'VirDetector: a bioinformatic pipeline for virus surveillance using nanopore
      sequencing'
  source_content_hash: sha256:68153a8ed9b261a7ba6ba11d44a0a4939b271e489c3da57ff8f80a655121daf5
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/fhcrc/seqmagick
  tools:
  - IQ-TREE2
  - MAFFT
  - MUSCLE
  - MrBayes
  - Nextflow
  - VirDetector
  - seqmagick
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0182
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# phylogenetic-substitution-model-selection

## Summary

Selects the best-fitting substitution model for viral consensus-genome alignments in VirDetector using IQ-TREE2's model-selection step, deliberately restricted to models that MrBayes can implement so the same alignment feeds both maximum-likelihood (IQ-TREE2) and Bayesian (MrBayes) tree inference.

## When to use

Use when a cleaned multiple alignment of consensus genomes plus database sequences must be converted into phylogenetic trees under both maximum-likelihood and Bayesian inference, and a substitution model compatible with both programs is required.

## When NOT to use

- Input is raw reads or unassembled contigs rather than a cleaned multiple alignment — the model-selection step presupposes prior alignment and CIAlign cleaning
- Only one inference framework will be used and strict IQ-TREE2/MrBayes model compatibility is not required — the MrBayes-compatible restriction unnecessarily narrows the candidate models
- The virus detected by BLAST bit-score is not represented in database/sequences.fasta, so no database sequences are available to enrich the alignment

## Inputs

- CIAlign-cleaned multiple sequence alignment (MAFFT or MUSCLE) of sample consensus genomes and matching VirDetector database sequences
- NEXUS-format alignment produced from the cleaned alignment via seqmagick
- Outgroup accession from database/sequences.fasta (e.g. NC_009942.1 for West Nile virus)

## Outputs

- Selected substitution model (restricted to MrBayes-compatible models)
- Maximum-likelihood tree inferred by IQ-TREE2 under the selected model
- Bayesian tree inferred by MrBayes under the same selected model
- Annotated NEXUS tree files for FigTree visualisation

## How to apply

First blast the sample consensus genomes against the VirDetector database (database/sequences.fasta) and select the virus with the highest bit-score hit, then collect all database sequences of that virus (segmented genomes as independent entries) together with the consensus genomes. Align them with MAFFT (faster) or MUSCLE (more precise, more resources) as set via params.alignment_tool, and clean the raw alignment with CIAlign. Convert the cleaned alignment to NEXUS with seqmagick and set the outgroup through params.iqtree_args (e.g. -o "NC_009942.1") and params.mrbayes_args (e.g. 'outgroup NC_009942.1'). Run IQ-TREE2's model-selection on this NEXUS alignment but restrict the candidate set to MrBayes-compatible models; this is the key decision point, ensuring the model chosen is reused consistently for both the IQ-TREE2 maximum-likelihood tree and the MrBayes Bayesian tree orchestrated by Nextflow (nextflow run main.nf).

## Related tools

- **IQ-TREE2** (Performs the substitution-model selection on the NEXUS alignment, restricted to MrBayes-compatible models, and subsequently infers the maximum-likelihood tree with the chosen model and outgroup via params.iqtree_args)
- **MrBayes** (Consumes the same NEXUS alignment and the IQ-TREE2-selected model to infer the Bayesian tree; outgroup set via params.mrbayes_args)
- **seqmagick** (Converts the CIAlign-cleaned alignment to NEXUS format as required for MrBayes and outgroup specification) — https://github.com/fhcrc/seqmagick
- **Nextflow** (Orchestrates the model-selection and dual tree-inference steps through nextflow run main.nf) — https://github.com/NLKaiser/VirDetector

## Examples

```
nextflow run main.nf
```

## Evaluation signals

- The selected model is executable by both IQ-TREE2 and MrBayes (MrBayes-compatible restriction was applied during model selection)
- The alignment supplied to the model-selection step is the CIAlign-cleaned, seqmagick-converted NEXUS file, not the raw MAFFT/MUSCLE output
- The outgroup accession (e.g. NC_009942.1) is consistently set for both tools via params.iqtree_args and params.mrbayes_args
- Both the maximum-likelihood (IQ-TREE2) and Bayesian (MrBayes) trees are produced from the same alignment and the same model
- Final tree files are in NEXUS format and can be annotated with virus name, lineage, country and collection date for FigTree

## Limitations

- Model choice is constrained to the subset of models MrBayes supports, which may exclude the globally best-fitting model available to IQ-TREE2 alone
- Alignment-tool choice trades speed for precision: MAFFT is faster while MUSCLE is more precise but requires more computing resources, and this choice affects the alignment the model is fitted on
- A metadata file, if specified, filters which samples are passed on to subsequent processing steps, changing the sequences entering the alignment and thus the model selection

## Evidence

- [intro] Dual inference rationale: "VirDetector then detects the virus present in the sample and uses a selection of sequences to infer phylogenetic trees, separately using maximum likelihood and bayesian inference."
- [readme] Outgroup recommendation: "If you want to execute a phylogenetic analysis, we recommend setting the outgroup for the specific tool. You select a sequence from *database/sequences.fasta* as outgrop via its respective accession."
- [readme] Alignment tool choice: "For aligning the sequences, you can choose between two different alignment tools, MAFFT or MUSCLE. While MAFFT is generally faster, MUSCLE is more precise but requires more computing resources."
- [methods] Outgroup example: "You have samples of the West Nile Virus and selected the sequence NC_009942.1 as outgroup:"
