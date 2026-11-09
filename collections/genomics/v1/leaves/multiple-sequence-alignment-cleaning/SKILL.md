---
name: multiple-sequence-alignment-cleaning
description: Use when raw MAFFT/MUSCLE multiple alignments of consensus genomes plus
  reference sequences must be cleaned of poorly aligned regions before phylogenetic
  tree inference.
when_to_use_negative:
- Input is still raw reads or unaligned FASTQ/FASTA/POD5 samples — alignment and cleaning
  happen only after consensus genome reconstruction (FASTA input enters the pipeline
  at the alignment step).
- Only two sequences (e.g. one consensus plus one reference) are available — a multiple
  alignment with CIAlign cleaning and tree inference is not meaningful.
- The analysis goal is basecalling, taxonomic classification, or structural variant
  detection rather than phylogenetic preparation.
edam_operation: http://edamontology.org/operation_0231
edam_topics:
- http://edamontology.org/topic_0092
tools:
- name: MAFFT
  role: faster multiple-sequence alignment of consensus genomes and database sequences,
    selected via params.alignment_tool
- name: MUSCLE
  role: more precise but more resource-intensive alternative aligner, selected via
    params.alignment_tool
- name: CIAlign
  role: cleans the raw multiple alignment to obtain the edited multiple alignment
- name: seqmagick
  role: converts the cleaned alignment to NEXUS format for IQ-TREE2 and MrBayes
  repo: https://github.com/fhcrc/seqmagick
- name: Nextflow
  role: orchestrates the alignment, CIAlign cleaning, and conversion steps via nextflow
    run main.nf
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bioinformatics/btaf029
    title: 'VirDetector: a bioinformatic pipeline for virus surveillance using nanopore
      sequencing'
  source_content_hash: sha256:1c34b2dc049307f3723b3481f9640dbfe161118c407dad1f5fd4a30f64d76f79
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/fhcrc/seqmagick
  tools:
  - CIAlign
  - IQ-TREE2
  - MAFFT
  - MUSCLE
  - MrBayes
  - Nextflow
  - VirDetector
  - seqmagick
  edam_operation: http://edamontology.org/operation_0231
  edam_topics:
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# multiple-sequence-alignment-cleaning

## Summary

Clean raw multiple-sequence alignments built from reconstructed viral consensus genomes and database reference sequences (aligned with MAFFT or MUSCLE) using CIAlign, then convert the edited alignment to NEXUS with seqmagick for downstream maximum-likelihood (IQ-TREE2) and Bayesian (MrBayes) tree inference in VirDetector.

## When to use

Use when raw MAFFT/MUSCLE multiple alignments of consensus genomes plus reference sequences must be cleaned of poorly aligned regions before phylogenetic tree inference.

## When NOT to use

- Input is still raw reads or unaligned FASTQ/FASTA/POD5 samples — alignment and cleaning happen only after consensus genome reconstruction (FASTA input enters the pipeline at the alignment step).
- Only two sequences (e.g. one consensus plus one reference) are available — a multiple alignment with CIAlign cleaning and tree inference is not meaningful.
- The analysis goal is basecalling, taxonomic classification, or structural variant detection rather than phylogenetic preparation.

## Inputs

- raw multiple-sequence alignment (FASTA) of sample consensus genomes plus all database/sequences.fasta sequences of the detected virus (segmented genomes as independent entries)
- VirDetector database sequences (database/sequences.fasta)
- alignment tool choice via params.alignment_tool ("mafft" or "muscle")

## Outputs

- edited (cleaned) multiple alignment produced by CIAlign
- NEXUS-format cleaned alignment ready for outgroup specification, IQ-TREE2 model selection, and MrBayes input

## How to apply

First collect all database sequences of the detected virus (selected as the highest bit-score BLAST hit of the sample consensus genomes against database/sequences.fasta; segmented genomes are independent entries) together with the reconstructed consensus genomes. Align them with MAFFT (faster) or MUSCLE (more precise, more resources), chosen via params.alignment_tool in nextflow.config. Pass the raw multiple alignment to CIAlign to remove poorly aligned or divergent regions, yielding the edited multiple alignment. Finally convert the cleaned alignment to NEXUS format with seqmagick so an outgroup (e.g. NC_009942.1) can be declared and substitution-model selection (IQ-TREE2, restricted to MrBayes-compatible models) and tree inference can proceed.

## Related tools

- **MAFFT** (faster multiple-sequence alignment of consensus genomes and database sequences, selected via params.alignment_tool)
- **MUSCLE** (more precise but more resource-intensive alternative aligner, selected via params.alignment_tool)
- **CIAlign** (cleans the raw multiple alignment to obtain the edited multiple alignment)
- **seqmagick** (converts the cleaned alignment to NEXUS format for IQ-TREE2 and MrBayes) — https://github.com/fhcrc/seqmagick
- **Nextflow** (orchestrates the alignment, CIAlign cleaning, and conversion steps via nextflow run main.nf)

## Examples

```
nextflow run main.nf  # with params.alignment_tool = "mafft" in nextflow.config
```

## Evaluation signals

- The edited alignment is shorter than the raw MAFFT/MUSCLE alignment because CIAlign removed poorly aligned regions, while all input sequences (consensus + database entries of the detected virus) are retained.
- The cleaned alignment is present in NEXUS format (seqmagick conversion succeeded) and can be parsed by IQ-TREE2 and MrBayes.
- The outgroup specified via params (e.g. NC_009942.1 for West Nile virus) appears in the alignment and is accepted by both tools' argument formats.
- Segmented genomes of the detected virus are present as independent entries in the alignment, matching the database header structure.
- Downstream IQ-TREE2 substitution-model search succeeds restricted to MrBayes-compatible models, confirming the alignment is valid input for both inference engines.

## Limitations

- Alignment tool choice is a trade-off: MAFFT is faster while MUSCLE is more precise but requires more computing resources, so cleaning quality depends on this params.alignment_tool setting.
- If a metadata file is specified, only samples defined in that file are passed to subsequent steps, so sequences may be silently dropped from the alignment.
- Cleaning quality depends on the BLAST-based virus selection: the wrong highest bit-score hit means the wrong reference set enters the alignment.

## Evidence

- [methods] Clean the raw multiple alignment with CIAlign to obtain the edited multiple alignment.: [evidence span withheld — anchor status: not_found]
- [readme] MAFFT vs MUSCLE trade-off: "For aligning the sequences, you can choose between two different alignment tools, MAFFT or MUSCLE. While MAFFT is generally faster, MUSCLE is more precise but requires more computing resources."
- [methods] virus selection by bit-score feeds the alignment: [evidence span withheld — anchor status: not_found]
- [methods] outgroup configuration example: "You have samples of the West Nile Virus and selected the sequence NC_009942.1 as outgroup:"
- [intro] tree inference after alignment: "VirDetector then detects the virus present in the sample and uses a selection of sequences to infer phylogenetic trees, separately using maximum likelihood and bayesian inference."
