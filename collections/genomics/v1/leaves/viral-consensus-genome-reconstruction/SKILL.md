---
name: viral-consensus-genome-reconstruction
description: Use when nanopore FASTQ/FAST5/POD5 samples in VirDetector's input directory
  must yield a viral consensus genome, and the user chooses a reference sequence or
  opts into the automated hybrid assembly in nextflow.config.
when_to_use_negative:
- Input is already assembled FASTA sequences — VirDetector accepts FASTA input but
  only uses it from the alignment/phylogeny step onwards, so no consensus reconstruction
  occurs
- Samples are filtered out by a metadata file (params.metadata), since only samples
  defined in that file are passed on to subsequent processing steps
- All reconstruction steps are disabled in nextflow.config — samples will only be
  preprocessed and trimmed
edam_operation: http://edamontology.org/operation_0362
edam_topics:
- http://edamontology.org/topic_0196
tools:
- name: VirDetector
  role: Nextflow pipeline orchestrating the reference-based and hybrid genome reconstruction
    branches
  repo: https://github.com/NLKaiser/VirDetector.git
- name: minimap2
  role: Aligns trimmed sample reads against the provided reference sequence in the
    reference-based branch
- name: samtools
  role: Sorts/indexes the BAM and calls the viral consensus genome (consensus_genome.fasta)
- name: Qualimap
  role: Produces mapping and coverage statistics visualisations from the sorted BAM
- name: Canu
  role: De novo assembly of trimmed reads in the hybrid branch, parameterised with
    the expected genome size
- name: BLAST
  role: Blasts hybrid-branch contigs against the VirDetector database; the highest
    bit-score hit becomes the reference
provenance:
  source_task_ids:
  - task_001
  - task_004
  - task_003
  source_papers:
  - doi: 10.1093/bioinformatics/btaf029
    title: 'VirDetector: a bioinformatic pipeline for virus surveillance using nanopore
      sequencing'
  source_content_hash: sha256:91477ef84362c1e8687afa5008b0b90f189a506a558c0cb3c1c9ea2f6017e3d1
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/NLKaiser/VirDetector.git
  tools:
  - BLAST
  - Canu
  - Conda
  - Docker
  - Dorado
  - Guppy
  - Qualimap
  - VirDetector
  - minimap2
  - samtools
  edam_operation: http://edamontology.org/operation_0362
  edam_topics:
  - http://edamontology.org/topic_0196
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 12
---

# viral-consensus-genome-reconstruction

## Summary

Reconstruct viral consensus genomes from nanopore reads within VirDetector, either by mapping trimmed reads against a user-supplied reference (minimap2 + samtools) or via a hybrid branch that de novo-assembles with Canu and picks a database reference by highest BLAST bit-score, with structural variant detection as part of the reference-based branch.

## When to use

Use when nanopore FASTQ/FAST5/POD5 samples in VirDetector's input directory must yield a viral consensus genome, and the user chooses a reference sequence or opts into the automated hybrid assembly in nextflow.config.

## When NOT to use

- Input is already assembled FASTA sequences — VirDetector accepts FASTA input but only uses it from the alignment/phylogeny step onwards, so no consensus reconstruction occurs
- Samples are filtered out by a metadata file (params.metadata), since only samples defined in that file are passed on to subsequent processing steps
- All reconstruction steps are disabled in nextflow.config — samples will only be preprocessed and trimmed

## Inputs

- Trimmed nanopore FASTQ reads per sample (.fastq, .fq, .fastq.gz, .fq.gz)
- Reference sequence in FASTA format placed in the resources folder (params.ref)
- VirDetector reference database (database/sequences.fasta) for hybrid-branch BLAST selection
- nextflow.config parameters: params.ref, params.assembly_hybrid, params.canu_genome_size

## Outputs

- consensus_genome.fasta — consensus FASTA of reference-length genome per sample
- Sorted/indexed BAM mapping files per sample
- Qualimap mapping and coverage statistics visualisations
- Structural variant calls from the assembled genomes
- Assembled contigs (hybrid branch, from Canu) and BLAST hit selection

## How to apply

For the reference-based branch, set the required params.ref in nextflow.config to a reference FASTA placed in the resources folder, align trimmed FASTQ reads to it with minimap2, sort/index the BAM with samtools, run Qualimap for coverage and mapping-statistics visualisations, and call the consensus FASTA (consensus_genome.fasta) with samtools; structural variant calls are produced alongside the consensus. For the hybrid branch, enable params.assembly_hybrid and set params.canu_genome_size (e.g. "11k" for the ~11,000-base West Nile virus genome), assemble reads de novo with Canu, BLAST the contigs against database/sequences.fasta, select the sequence with the highest bit-score as reference, then proceed identically to the reference-based branch. Both branches feed the consensus genomes into MAFFT (default) or MUSCLE alignment and downstream phylogenetic inference. Rationale: VirDetector executes all of this automatically with a single 'nextflow run main.nf' command, so configuration of nextflow.config is the only manual editing required.

## Related tools

- **VirDetector** (Nextflow pipeline orchestrating the reference-based and hybrid genome reconstruction branches) — https://github.com/NLKaiser/VirDetector.git
- **minimap2** (Aligns trimmed sample reads against the provided reference sequence in the reference-based branch)
- **samtools** (Sorts/indexes the BAM and calls the viral consensus genome (consensus_genome.fasta))
- **Qualimap** (Produces mapping and coverage statistics visualisations from the sorted BAM)
- **Canu** (De novo assembly of trimmed reads in the hybrid branch, parameterised with the expected genome size)
- **BLAST** (Blasts hybrid-branch contigs against the VirDetector database; the highest bit-score hit becomes the reference)

## Examples

```
conda activate VirDetector; nextflow run main.nf
```

## Evaluation signals

- consensus_genome.fasta exists for the sample and is a valid FASTA record of the reference-length genome
- For the West Nile virus hybrid test, consensus length is close to the expected ~11,000 bases (params.canu_genome_size = "11k") and the top BLAST hit is assigned to West Nile virus
- Mapping files (sorted/indexed BAM) and Qualimap coverage reports were generated for the sample
- Hybrid-branch BLAST selection succeeded against database/sequences.fasta with a single highest bit-score reference chosen
- Consensus genomes are available downstream for MAFFT/MUSCLE alignment and maximum likelihood and Bayesian tree inference

## Limitations

- Reference-based assembly requires params.ref to point at a reference FASTA; without it the step cannot run
- Hybrid assembly requires the expected genome size (params.canu_genome_size); a wrong size estimate degrades Canu assembly quality
- Development and testing were performed under Ubuntu 22.04 LTS; on other operating systems the Docker installation should be used
- Alignment tool choice involves a trade-off: MAFFT is faster while MUSCLE is more precise but requires more computing resources

## Evidence

- [abstract] After preprocessing the samples, including the possibility for basecalling, the pipeline classifies each sample taxonomically and reconstructs the viral consensus genomes, which are then used in: "After preprocessing the samples, including the possibility for basecalling, the pipeline classifies each sample taxonomically and reconstructs the viral consensus genomes"
- [intro] Reference vs hybrid reconstruction options: "reconstructs viral genomes, either by using a reference sequence or with an automated hybrid assembly approach"
- [readme] Required reference parameter: "For the reference based assembly, you must specifiy the path to your reference sequence."
- [readme] Hybrid genome size parameter: "For the hybrid assembly, you must specify the expected genome size of your organism."
- [readme] WNV genome size example: "You can specify a genome size of 11.000 bases like this: **params.canu_genome_size = "11k"**"
