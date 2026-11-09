---
name: nanopore-read-de-novo-assembly
description: Use when assembling untargeted (metagenomic) Oxford Nanopore FASTQ reads
  from clinical samples where uneven viral genome abundance (e.g., co-infections)
  may prevent a single Canu run from recovering low-abundance virus genomes.
when_to_use_negative:
- Large DNA viruses with extensive repetitive regions (e.g., mpox), where assemblies
  may contain inaccuracies and should only be regarded as draft genomes
- Reads already reduced to a single high-abundance target where a single Canu run
  suffices and iterative re-assembly would only add runtime
- Reads that have not passed adapter/primer trimming, Centrifuge non-viral filtering,
  and host/contaminant removal — assembling unfiltered reads will waste Canu resources
  on non-viral content
edam_operation: http://edamontology.org/operation_0524
edam_topics:
- http://edamontology.org/topic_0196
- http://edamontology.org/topic_3174
tools:
- name: Canu
  role: Long-read de novo assembler; run iteratively on the residual unmapped reads
    at each cycle
- name: cd-hit-est
  role: Clusters the longest pre-selected reads when a target-filtered assembly yields
    no contigs, providing contig stand-ins
- name: Nextflow
  role: Workflow engine that orchestrates the assembly, mapping-removal, and clustering
    steps in ViMOP
  repo: https://github.com/opr-group-bnitm/vimop
- name: ViMOP
  role: Pipeline implementing the iterative Canu re-assembly and CD-HIT fallback logic
  repo: https://github.com/opr-group-bnitm/vimop
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioinformatics/btaf687
    title: 'ViMOP: a user-friendly and field-applicable pipeline for untargeted viral
      genome nanopore sequencing'
  source_content_hash: sha256:d282d37a698b46ebc4d349b7f7c092d1f1a95bed40c03b6201c1ee5b653d8d11
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/opr-group-bnitm/vimop
  tools:
  - Canu
  - Nextflow
  - ViMOP
  - cd-hit-est
  edam_operation: http://edamontology.org/operation_0524
  edam_topics:
  - http://edamontology.org/topic_0196
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# nanopore-read-de-novo-assembly

## Summary

Iterative de novo assembly of untargeted viral nanopore reads with Canu in ViMOP: reads mapping to newly generated contigs are removed and Canu is re-run on the remainder, with a cd-hit-est pre-clustering fallback when a target-filtered read set yields no contigs.

## When to use

Use when assembling untargeted (metagenomic) Oxford Nanopore FASTQ reads from clinical samples where uneven viral genome abundance (e.g., co-infections) may prevent a single Canu run from recovering low-abundance virus genomes.

## When NOT to use

- Large DNA viruses with extensive repetitive regions (e.g., mpox), where assemblies may contain inaccuracies and should only be regarded as draft genomes
- Reads already reduced to a single high-abundance target where a single Canu run suffices and iterative re-assembly would only add runtime
- Reads that have not passed adapter/primer trimming, Centrifuge non-viral filtering, and host/contaminant removal — assembling unfiltered reads will waste Canu resources on non-viral content

## Inputs

- Filtered FASTQ read sets (target-filtered and no-target) produced by ViMOP's trimming, Centrifuge classification, and host/contaminant removal steps
- ViMOP Nextflow parameters: --reassemble_max_iter, --assemble_notarget, --nocontigs_max_reads_precluster, --nocontigs_nreads

## Outputs

- Contig FASTA files (e.g. no-target.contigs.fasta, re-assembly.contigs.fasta) in the assembly subdirectory of the sample output folder
- Contigs used downstream for BLAST-based reference identification and reference-guided consensus assembly

## How to apply

Run Canu on the filtered read set (and on the no-target read set unless --assemble_notarget false) to produce an initial contig set. Map all reads against the new contigs, discard mapped reads, and re-run Canu on the unmapped remainder, repeating the mapping-removal-assembly cycle up to --reassemble_max_iter iterations (0 disables it) and stopping early when no reads remain or Canu yields no contigs — this recovers genomes of lower-abundance viruses that would otherwise be swamped in a single assembly. For a target-filtered read set that produces no contigs, take the longest --nocontigs_max_reads_precluster reads, cluster them with cd-hit-est, and select the longest --nocontigs_nreads reads (Canu-corrected if available, otherwise raw) as contig stand-ins for the subsequent target search. Canu is resource-hungry, so provision adequate RAM/CPUs (ViMOP recommends at least 30 GB RAM and 16 CPUs).

## Related tools

- **Canu** (Long-read de novo assembler; run iteratively on the residual unmapped reads at each cycle)
- **cd-hit-est** (Clusters the longest pre-selected reads when a target-filtered assembly yields no contigs, providing contig stand-ins)
- **Nextflow** (Workflow engine that orchestrates the assembly, mapping-removal, and clustering steps in ViMOP) — https://github.com/opr-group-bnitm/vimop
- **ViMOP** (Pipeline implementing the iterative Canu re-assembly and CD-HIT fallback logic) — https://github.com/opr-group-bnitm/vimop

## Examples

```
nextflow run opr-group-bnitm/vimop --fastq fastq_dir --reassemble_max_iter 3 --nocontigs_max_reads_precluster 50000 --nocontigs_nreads 100
```

## Evaluation signals

- The assembly directory contains the expected contig FASTA files (no-target.contigs.fasta, re-assembly.contigs.fasta)
- The iterative procedure terminated on one of the valid conditions: --reassemble_max_iter cycles reached, reads exhausted after mapping-removal, or a Canu run producing no contigs
- Each assembly iteration consumes the reads unmapped against the previous contig set, so the read input strictly shrinks across iterations
- When the fallback fires, the cd-hit-est input is exactly the longest --nocontigs_max_reads_precluster reads and the output stand-ins are the longest --nocontigs_nreads reads (Canu-corrected where available)
- Downstream BLAST reference identification succeeds in matching contigs to known viral references in ViMOP_DB

## Limitations

- Iterative re-assembly is deactivated entirely with --reassemble_max_iter 0
- Some tools like Canu need considerable resources; ViMOP recommends at least 30 GB RAM and 16 CPUs and may run slowly with less
- For large DNA viruses with extensive repetitive regions (e.g., mpox), assemblies may contain inaccuracies and should be regarded as draft genomes
- Centrifuge-based filtering is limited in accuracy and may remove some false positives before assembly

## Evidence

- [methods] After the initial assembly, an iterative re-assembly procedure is run (unless deactivated with --reassemble_max_iter 0).: "After the initial assembly, an iterative re-assembly procedure is run (unless deactivated with `--reassemble_max_iter 0`)."
- [methods] An assembly is run for each of the filtered read sets and for the read set with no target filter.: "An assembly is run for each of the filtered read sets (explained in the previous section) and for the read set with no target filter."
- [methods] the longest X reads (set with --nocontigs_max_reads_precluster) are passed to a clustering by cd-hit-est.: "the longest X reads (set with `--nocontigs_max_reads_precluster`) are passed to a clustering by cd-hit-est."
- [readme] some tools like Canu need quite some resources.: "However, this may or may not work, and it may take much longer, as some tools like Canu need quite some resources."
- [abstract] The software assembles de novo contigs, matches contigs to known viral references and uses them to assemble consensus genomes.: "The software assembles de novo contigs, matches contigs to known viral references and uses them to assemble consensus genomes."
