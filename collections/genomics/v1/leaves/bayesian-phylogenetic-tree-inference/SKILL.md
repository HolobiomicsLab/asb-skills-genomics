---
name: bayesian-phylogenetic-tree-inference
description: Use when nanopore-derived viral consensus genomes (e.g. FASTA samples
  of West Nile virus) must be placed phylogenetically with posterior support alongside
  reference database sequences from database/sequences.fasta.
when_to_use_negative:
- The input has not been taxonomically classified or no consensus genome has been
  reconstructed — the tree inference step consumes cleaned multiple alignments, not
  raw FASTQ/POD5 reads
- You only need a quick clustering or taxonomic assignment; Bayesian MCMC inference
  requires MrBayes-compatible model selection and substantial compute compared to
  the faster MAFFT-only alignment stage
- No suitable outgroup sequence exists in database/sequences.fasta for the detected
  virus, since the README recommends setting the outgroup via its accession
edam_operation: http://edamontology.org/operation_0323
edam_topics:
- http://edamontology.org/topic_0684
tools:
- name: VirDetector
  role: Pipeline orchestrating virus detection, alignment, model selection and tree
    inference; installed with Conda or Docker
  repo: https://github.com/NLKaiser/VirDetector.git
- name: MrBayes
  role: Infers the Bayesian phylogenetic tree; outgroup set via params.mrbayes_args
    (e.g. 'outgroup NC_009942.1')
- name: IQ-TREE2
  role: Selects a MrBayes-compatible substitution model and infers the parallel maximum-likelihood
    tree; outgroup set via params.iqtree_args = "-o \"NC_009942.1\""
- name: BLAST
  role: Selects the detected virus as the highest bit-score hit when consensus genomes
    are searched against database/sequences.fasta
- name: MAFFT
  role: Faster multiple sequence alignment option for consensus + database sequences
    (params.alignment_tool = 'mafft')
- name: MUSCLE
  role: More precise but resource-hungry alignment option (params.alignment_tool =
    'muscle')
- name: seqmagick
  role: Converts the cleaned alignment to NEXUS format required by MrBayes
  repo: https://github.com/fhcrc/seqmagick
- name: FigTree
  role: Graphical viewer for the annotated NEXUS tree files
  repo: https://github.com/rambaut/figtree
- name: Nextflow
  role: Orchestrates the tree inference steps via nextflow run main.nf
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bioinformatics/btaf029
    title: 'VirDetector: a bioinformatic pipeline for virus surveillance using nanopore
      sequencing'
  source_content_hash: sha256:0c28ccefbfe0be09e9da32b07034bf7b6ea2ec6ea33d0ec5764c98362de61910
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/NLKaiser/VirDetector.git
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
  edam_operation: http://edamontology.org/operation_0323
  edam_topics:
  - http://edamontology.org/topic_0684
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# bayesian-phylogenetic-tree-inference

## Summary

Infers Bayesian phylogenetic trees from reconstructed viral consensus genomes within the VirDetector nanopore-surveillance pipeline, running MrBayes in parallel with maximum-likelihood (IQ-TREE2) inference on the same cleaned alignment.

## When to use

Use when nanopore-derived viral consensus genomes (e.g. FASTA samples of West Nile virus) must be placed phylogenetically with posterior support alongside reference database sequences from database/sequences.fasta.

## When NOT to use

- The input has not been taxonomically classified or no consensus genome has been reconstructed — the tree inference step consumes cleaned multiple alignments, not raw FASTQ/POD5 reads
- You only need a quick clustering or taxonomic assignment; Bayesian MCMC inference requires MrBayes-compatible model selection and substantial compute compared to the faster MAFFT-only alignment stage
- No suitable outgroup sequence exists in database/sequences.fasta for the detected virus, since the README recommends setting the outgroup via its accession

## Inputs

- Viral consensus genomes reconstructed by VirDetector (FASTA)
- Reference sequences for the detected virus from database/sequences.fasta (segmented genomes as independent entries)
- Cleaned multiple sequence alignment converted to NEXUS format via seqmagick
- Outgroup accession selected from database/sequences.fasta (e.g. NC_009942.1)
- params.mrbayes_args and params.alignment_tool settings from nextflow.config

## Outputs

- Bayesian phylogenetic tree (MrBayes) in NEXUS format
- Maximum-likelihood tree (IQ-TREE2) inferred in parallel on the same alignment
- Annotated NEXUS tree files carrying virus name, lineage, country and collection date for FigTree visualisation

## How to apply

BLAST all sample consensus genomes against the VirDetector database (database/sequences.fasta) and select the detected virus as the highest bit-score hit, collecting all database entries sharing that virus field (segmented genomes are independent entries). Align consensus plus database sequences with MAFFT (faster) or MUSCLE (more precise, via params.alignment_tool), clean the raw multiple alignment with CIAlign, then convert it to NEXUS with seqmagick and set the outgroup (e.g. 'outgroup NC_009942.1' for MrBayes). Run IQ-TREE2 first to select a substitution model restricted to MrBayes-compatible models, then infer the Bayesian tree with MrBayes via the Nextflow orchestration (nextflow run main.nf). Finally annotate the NEXUS tree with virus name, lineage, country and collection date parsed from the sequence headers for FigTree visualisation.

## Related tools

- **VirDetector** (Pipeline orchestrating virus detection, alignment, model selection and tree inference; installed with Conda or Docker) — https://github.com/NLKaiser/VirDetector.git
- **MrBayes** (Infers the Bayesian phylogenetic tree; outgroup set via params.mrbayes_args (e.g. 'outgroup NC_009942.1'))
- **IQ-TREE2** (Selects a MrBayes-compatible substitution model and infers the parallel maximum-likelihood tree; outgroup set via params.iqtree_args = "-o \"NC_009942.1\"")
- **BLAST** (Selects the detected virus as the highest bit-score hit when consensus genomes are searched against database/sequences.fasta)
- **MAFFT** (Faster multiple sequence alignment option for consensus + database sequences (params.alignment_tool = 'mafft'))
- **MUSCLE** (More precise but resource-hungry alignment option (params.alignment_tool = 'muscle'))
- **seqmagick** (Converts the cleaned alignment to NEXUS format required by MrBayes) — https://github.com/fhcrc/seqmagick
- **FigTree** (Graphical viewer for the annotated NEXUS tree files) — https://github.com/rambaut/figtree
- **Nextflow** (Orchestrates the tree inference steps via nextflow run main.nf)

## Examples

```
nextflow run main.nf  # with params.mrbayes_args = ";\n\toutgroup NC_009942.1" and params.alignment_tool = "mafft" in nextflow.config
```

## Evaluation signals

- The detected virus was chosen as the highest bit-score BLAST hit of the consensus genome against database/sequences.fasta, and all same-virus database entries (including each segment) are present in the alignment input
- The alignment was converted to NEXUS via seqmagick and the chosen outgroup accession (e.g. NC_009942.1) appears as the rooted outgroup in the MrBayes tree
- The substitution model supplied to MrBayes is MrBayes-compatible, having been selected by IQ-TREE2 under that restriction
- Both the Bayesian and maximum-likelihood trees were produced from the same CIAlign-cleaned alignment and can be cross-checked for topological agreement
- Tree tips carry the virus name, lineage, country and collection date parsed from the sequence headers, enabling FigTree visualisation

## Limitations

- MUSCLE gives more precise alignments but requires substantially more computing resources than MAFFT (configured via params.alignment_tool)
- The phylogenetic database is limited to the viruses present in database/sequences.fasta; viruses outside it cannot be placed
- Outgroup selection must be configured manually per tool (params.iqtree_args / params.mrbayes_args); if metadata filtering is used, only samples defined in the metadata file pass through to downstream steps
- Tree rendering depends on database sequence headers carrying lineage, country and collection date; missing metadata degrades FigTree annotation

## Evidence

- [intro] VirDetector then detects the virus present in the sample and uses a selection of sequences to infer phylogenetic trees, separately using maximum likelihood and bayesian inference.: "VirDetector then detects the virus present in the sample and uses a selection of sequences to infer phylogenetic trees, separately using maximum likelihood and bayesian inference."
- [methods] If you want to execute a phylogenetic analysis, we recommend setting the outgroup for the specific tool.: "If you want to execute a phylogenetic analysis, we recommend setting the outgroup for the specific tool. You select a sequence from *database/sequences.fasta* as outgrop via its respective accession."
- [readme] For aligning the sequences, you can choose between two different alignment tools, MAFFT or MUSCLE.: "For aligning the sequences, you can choose between two different alignment tools, MAFFT or MUSCLE. While MAFFT is generally faster, MUSCLE is more precise but requires more computing resources."
- [methods] nextflow run main.nf is the pipeline test/execution command orchestrating all steps except basecalling.: "To test the execution of all steps of the pipeline, except basecalling, execute the following command:
``nnextflow run main.nf
"
- [methods] The VirDetector database for phylogenetic inference currently contains the viruses:: "The VirDetector database for phylogenetic inference currently contains the viruses:"
