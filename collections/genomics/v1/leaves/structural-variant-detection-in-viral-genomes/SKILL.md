---
name: structural-variant-detection-in-viral-genomes
description: Use when nanopore reads have already been assembled into viral consensus
  genomes (via reference-based or hybrid assembly) and structural variants relative
  to a reference should be identified as part of the VirDetector downstream analyses.
when_to_use_negative:
- When only raw FASTQ/FAST5/POD5 reads are available without reconstructed consensus
  genomes — the step operates on assembled consensus sequences, not reads
- When the goal is small-scale SNV/indel calling from raw reads rather than structural
  variant detection in assembled genomes
- When all VirDetector steps are disabled and only preprocessing and trimming of samples
  is desired
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3168
tools:
- name: VirDetector
  role: Pipeline orchestrating the optional structural-variant detection step on consensus
    genomes within its downstream analyses
  repo: https://github.com/NLKaiser/VirDetector
- name: Sniffles2
  role: Tool applied to viral consensus genome sequences to identify structural variants
    relative to the reference
- name: Nextflow
  role: Workflow engine executing the VirDetector step and organizing output directories
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/bioinformatics/btaf029
    title: 'VirDetector: a bioinformatic pipeline for virus surveillance using nanopore
      sequencing'
  source_content_hash: sha256:825e4b4bf5d9f3f2dd7c3de418f9b5d623f00df6308583ce6dbe2990d78db4ff
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
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# structural-variant-detection-in-viral-genomes

## Summary

Optional VirDetector step that detects structural variants in assembled viral genomes by running Sniffles2 on the sample's reconstructed consensus sequence and writing per-sample variant records. It extends nanopore-based virus surveillance beyond consensus reconstruction to large-scale genomic differences.

## When to use

Use when nanopore reads have already been assembled into viral consensus genomes (via reference-based or hybrid assembly) and structural variants relative to a reference should be identified as part of the VirDetector downstream analyses.

## When NOT to use

- When only raw FASTQ/FAST5/POD5 reads are available without reconstructed consensus genomes — the step operates on assembled consensus sequences, not reads
- When the goal is small-scale SNV/indel calling from raw reads rather than structural variant detection in assembled genomes
- When all VirDetector steps are disabled and only preprocessing and trimming of samples is desired

## Inputs

- Viral consensus genome sequences (FASTA) reconstructed by the reference-based or hybrid assembly step
- Reference sequence (FASTA) placed in the resources directory

## Outputs

- Per-sample VCF-style variant record file of detected structural variants
- Structural variant summary alongside reference-assembly outputs in results.txt

## How to apply

Load the viral consensus genome sequences generated earlier in the VirDetector workflow (Nextflow-managed) for each sample, since this step operates on consensus genomes, not raw reads. Run Sniffles2 on each sample's consensus sequence to call structural variants relative to the reference. Write the detected variants per sample into a VCF-style variant record file placed in the pipeline's structured output directories. Because the step is optional, it can be enabled or disabled through the nextflow.config parameters like other VirDetector steps. Validation consists of confirming a variant record file exists per processed sample and that variants are summarized alongside the other reference-assembly outputs in results.txt.

## Related tools

- **VirDetector** (Pipeline orchestrating the optional structural-variant detection step on consensus genomes within its downstream analyses) — https://github.com/NLKaiser/VirDetector
- **Sniffles2** (Tool applied to viral consensus genome sequences to identify structural variants relative to the reference)
- **Nextflow** (Workflow engine executing the VirDetector step and organizing output directories)

## Examples

```
nextflow run main.nf
```

## Evaluation signals

- A variant record file (VCF-style) exists for each processed sample
- Detected structural variants are summarized in results.txt together with the other reference-assembly outputs
- Variant calls are relative to the same reference sequence used for the reference-based assembly
- The consensus genomes input to Sniffles2 match the outputs of the earlier assembly step per sample

## Limitations

- The step is optional; it must be enabled via the pipeline configuration and depends on prior genome reconstruction
- Variant detection operates on consensus genomes, so information contained only in raw reads may be missed
- Customization of the tool call is limited to parameters exposed in nextflow.config or via the respective '[program]_args' parameter

## Evidence

- [intro] Structural variant detection is part of the VirDetector downstream analyses: "It also detects structural variants in the assembled genomes."
- [other] Sniffles2 is applied to consensus sequences produced by the pipeline: [evidence span withheld — anchor status: not_found]
- [methods] VirDetector is executed via Nextflow: "To test the execution of all steps of the pipeline, except basecalling, execute the following command:
``nnextflow run main.nf
"
- [readme] Structural variant detection confirmed in README: "VirDetector then detects the virus present in the sample and uses a selection of sequences to infer phylogenetic trees, separately using maximum likelihood and bayesian inference. It also detects"
