---
name: iterative-contig-reassembly-read-recruitment
description: Use when assembling untargeted nanopore reads from samples with mixed
  viral abundances (co-infections), where a single Canu run would recruit most reads
  to the dominant virus and miss lower-abundance genomes.
when_to_use_negative:
- reassemble_max_iter is set to 0, explicitly deactivating the iterative procedure
- The sample is expected to contain a single virus at roughly uniform abundance —
  the extra mapping-removal-assembly cycles add runtime without benefit
- Large DNA viruses with extensive repetitive regions (e.g. mpox), where ViMOP assemblies
  may already be inaccurate drafts
edam_operation: http://edamontology.org/operation_0524
edam_topics:
- http://edamontology.org/topic_0196
- http://edamontology.org/topic_0622
tools:
- name: Canu
  role: Long-read assembler run once per re-assembly iteration on the remaining unmapped
    reads
- name: cd-hit-est
  role: Clusters the longest --nocontigs_max_reads_precluster reads when a target-filtered
    assembly yields no contigs
- name: Nextflow
  role: Workflow engine executing the ViMOP assembly module and its iterations
  repo: https://github.com/opr-group-bnitm/vimop
- name: ViMOP
  role: Pipeline implementing the iterative assembly and no-contig fallback logic
  repo: https://github.com/opr-group-bnitm/vimop
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioinformatics/btaf687
    title: 'ViMOP: a user-friendly and field-applicable pipeline for untargeted viral
      genome nanopore sequencing'
  source_content_hash: sha256:4c0233d459133c853b03d2ef5dbe2d6c0b2c21da1bd0c32206a2adf172a399b7
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
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# iterative-contig-reassembly-read-recruitment

## Summary

ViMOP's de novo assembly strategy for uneven viral genome abundance: run Canu repeatedly, removing reads that map to newly generated contigs after each cycle so low-abundance co-infecting viruses are not dominated by high-abundance ones, with a cd-hit-est read-clustering fallback when Canu yields no contigs.

## When to use

Use when assembling untargeted nanopore reads from samples with mixed viral abundances (co-infections), where a single Canu run would recruit most reads to the dominant virus and miss lower-abundance genomes.

## When NOT to use

- reassemble_max_iter is set to 0, explicitly deactivating the iterative procedure
- The sample is expected to contain a single virus at roughly uniform abundance — the extra mapping-removal-assembly cycles add runtime without benefit
- Large DNA viruses with extensive repetitive regions (e.g. mpox), where ViMOP assemblies may already be inaccurate drafts

## Inputs

- Filtered FASTQ read sets (target-filtered and no-target) from untargeted Oxford Nanopore sequencing
- reassemble_max_iter, nocontigs_max_reads_precluster, nocontigs_nreads, assemble_notarget parameter values

## Outputs

- no-target.contigs.fasta
- re-assembly.contigs.fasta (contig FASTA files in the assembly subdirectory of the sample output folder)
- cd-hit-est clusters / selected longest reads as contig stand-ins when Canu produced no contigs

## How to apply

Run Canu on the filtered read set (and on the no-target read set unless --assemble_notarget false) to produce an initial contig set. Map all reads against the newly generated contigs, discard mapped reads, and re-run Canu on the remaining unmapped reads; repeat this cycle up to --reassemble_max_iter iterations (0 disables it), stopping early when no reads remain or Canu produces no contigs. If a target-filtered read set yields no contigs, take the longest --nocontigs_max_reads_precluster reads, cluster them with cd-hit-est, and select the longest --nocontigs_nreads reads (Canu-corrected reads if available, otherwise raw reads) as contig stand-ins for the subsequent target search. Rationale: each cycle strips reads already represented by contigs, so reads from rarer viruses accumulate for the next assembly round. Note that Canu is resource-hungry; the README recommends at least 30 GB RAM and 16 CPUs.

## Related tools

- **Canu** (Long-read assembler run once per re-assembly iteration on the remaining unmapped reads)
- **cd-hit-est** (Clusters the longest --nocontigs_max_reads_precluster reads when a target-filtered assembly yields no contigs)
- **Nextflow** (Workflow engine executing the ViMOP assembly module and its iterations) — https://github.com/opr-group-bnitm/vimop
- **ViMOP** (Pipeline implementing the iterative assembly and no-contig fallback logic) — https://github.com/opr-group-bnitm/vimop

## Examples

```
nextflow run opr-group-bnitm/vimop --fastq <fastq_dir> --reassemble_max_iter 3 --nocontigs_max_reads_precluster 5000 --nocontigs_nreads 100
```

## Evaluation signals

- The assembly subdirectory contains the expected contig FASTA files (no-target.contigs.fasta and re-assembly.contigs.fasta)
- The iterative procedure terminated upon reaching --reassemble_max_iter cycles, exhaustion of reads, or a Canu run producing no contigs
- After each cycle, reads mapping against generated contigs are absent from the read set fed into the next Canu run
- For a no-contig target-filtered set, exactly --nocontigs_nreads stand-in reads were selected from cd-hit-est clusters of the longest --nocontigs_max_reads_precluster reads

## Limitations

- Canu needs substantial resources; the README recommends at least 30 GB RAM and 16 CPUs, and lower-resource runs may fail or take much longer
- Centrifuge-based filtering that precedes assembly is limited in accuracy and may remove some false-positive viral reads
- For large DNA viruses with repetitive regions (e.g. mpox), assemblies may contain inaccuracies and should be regarded as draft genomes

## Evidence

- [methods] After the initial assembly, an iterative re-assembly procedure is run (unless deactivated with `--reassemble_max_iter 0`).: "After the initial assembly, an iterative re-assembly procedure is run (unless deactivated with `--reassemble_max_iter 0`)."
- [methods] the longest X reads (set with `--nocontigs_max_reads_precluster`) are passed to a clustering by cd-hit-est.: "the longest X reads (set with `--nocontigs_max_reads_precluster`) are passed to a clustering by cd-hit-est."
- [abstract] The software assembles de novo contigs, matches contigs to known viral references and uses them to assemble consensus genomes.: "The software assembles de novo contigs, matches contigs to known viral references and uses them to assemble consensus genomes."
- [methods] An assembly is run for each of the filtered read sets (explained in the previous section) and for the read set with no target filter.: "An assembly is run for each of the filtered read sets (explained in the previous section) and for the read set with no target filter."
- [readme] it may take much longer, as some tools like Canu need quite some resources.: "However, this may or may not work, and it may take much longer, as some tools like Canu need quite some resources."
