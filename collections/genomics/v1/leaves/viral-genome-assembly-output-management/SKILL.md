---
name: viral-genome-assembly-output-management
description: Use when a nanopore-sequencing virus-surveillance pipeline has produced
  per-sample viral consensus genomes (from reference-based or hybrid assembly) and
  you must detect structural variants on them and organize the resulting variant record
  files per sample.
when_to_use_negative:
- The input is raw nanopore reads (FASTQ/FAST5/POD5) rather than reconstructed consensus
  genomes — consensus reconstruction must happen first.
- No reference sequence is available and only de novo assembly outputs without variant
  calling are needed, since structural-variant detection is relative to a reference.
- The analysis targets taxonomic classification or phylogenetic tree inference rather
  than structural-variant detection in assembled genomes.
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_0196
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3673
tools:
- name: VirDetector
  role: Pipeline that reconstructs viral consensus genomes and applies the optional
    structural-variant detection step on them
  repo: https://github.com/NLKaiser/VirDetector
- name: Nextflow
  role: Workflow engine orchestrating the per-sample consensus loading, Sniffles2
    execution and output-directory structuring
- name: Sniffles2
  role: Structural-variant detection tool run on each sample's viral consensus genome
    sequence to identify variants relative to the reference
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/bioinformatics/btaf029
    title: 'VirDetector: a bioinformatic pipeline for virus surveillance using nanopore
      sequencing'
  source_content_hash: sha256:3231ce79fbc890e1e126c758065bad5097d9427f1070fabd657d3823d22d76e7
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/NLKaiser/VirDetector
  tools:
  - Nextflow
  - Sniffles2
  - VirDetector
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_0196
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3673
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# viral-genome-assembly-output-management

## Summary

Manages the downstream outputs of VirDetector's viral genome assembly step, including per-sample variant record files produced by applying Sniffles2 to reconstructed viral consensus genomes, and their summarized reporting alongside reference-assembly results in results.txt.

## When to use

Use when a nanopore-sequencing virus-surveillance pipeline has produced per-sample viral consensus genomes (from reference-based or hybrid assembly) and you must detect structural variants on them and organize the resulting variant record files per sample.

## When NOT to use

- The input is raw nanopore reads (FASTQ/FAST5/POD5) rather than reconstructed consensus genomes — consensus reconstruction must happen first.
- No reference sequence is available and only de novo assembly outputs without variant calling are needed, since structural-variant detection is relative to a reference.
- The analysis targets taxonomic classification or phylogenetic tree inference rather than structural-variant detection in assembled genomes.

## Inputs

- per-sample viral consensus genome sequences (FASTA) from the reference-based or automated hybrid assembly step
- reference genome sequence in FASTA format (e.g. placed in the resources directory and set via params.ref)
- Nextflow configuration (nextflow.config) defining enabled steps such as params.assembly_hybrid

## Outputs

- one VCF-style structural-variant record file per processed sample
- results.txt summarizing detected structural variants alongside the other reference-assembly outputs

## How to apply

For each processed sample, load the viral consensus genome sequence produced by the reference-based or automated hybrid assembly step (assembly inputs in VirDetector are FASTQ, FAST5 or POD5, with FASTA accepted from the alignment step onwards). Run Sniffles2 on each sample's consensus genome to identify structural variants relative to the reference sequence specified via params.ref in nextflow.config. Write the detected structural variants for each sample into a VCF-style variant record file, placed in the pipeline's structured output directories. Finally, verify that every processed sample has a variant record file and that the detected variants are summarized together with the other reference-assembly outputs in results.txt.

## Related tools

- **VirDetector** (Pipeline that reconstructs viral consensus genomes and applies the optional structural-variant detection step on them) — https://github.com/NLKaiser/VirDetector
- **Nextflow** (Workflow engine orchestrating the per-sample consensus loading, Sniffles2 execution and output-directory structuring)
- **Sniffles2** (Structural-variant detection tool run on each sample's viral consensus genome sequence to identify variants relative to the reference)

## Examples

```
nextflow run main.nf
```

## Evaluation signals

- A variant record file exists for every processed sample in the pipeline's structured output directories.
- Detected structural variants are summarized alongside the other reference-assembly outputs in results.txt.
- Each variant record file is in VCF style and reflects variants called against the reference set via params.ref.
- Consensus genomes fed to Sniffles2 originate from the reference-based or hybrid assembly step (hybrid assembly controlled by params.assembly_hybrid).

## Limitations

- Structural-variant detection is an optional step in VirDetector; if all steps are disabled, samples are only preprocessed and trimmed, so no variant records are produced.
- Variant detection is relative to a supplied reference sequence; without params.ref set for reference-based assembly, the comparison basis is absent.
- VirDetector was developed and tested under Ubuntu 22.04 LTS; other operating systems require the Docker installation.

## Evidence

- [full_text] VirDetector provides an optional structural-variant analysis step in which Sniffles2 is applied to the consensus sequences produced by the pipeline, operating on the viral consensus genomes reconstructed earlier in the workflow.: [evidence span withheld — anchor status: not_found]
- [intro] The pipeline detects structural variants in assembled genomes as part of its workflow.: "It also detects structural variants in the assembled genomes."
- [readme] The pipeline reconstructs viral genomes either by reference or hybrid assembly, the inputs to the structural-variant step.: "It can taxonomically classify samples and reconstructs viral genomes, either by using a reference sequence or with an automated hybrid assembly approach."
- [readme] Reference-based assembly requires specifying the reference sequence path, the basis for relative variant calling.: "For the reference based assembly, you must specifiy the path to your reference sequence."
