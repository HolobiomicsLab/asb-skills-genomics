---
name: assembly-read-mapping-filtering
description: Use when assembling untargeted nanopore FASTQ reads from samples with
  mixed viral abundances (e.g., co-infections), where an initial Canu assembly only
  recovers the most abundant genomes and residual reads must be re-assembled iteratively.
when_to_use_negative:
- The sample contains a single dominant virus at even coverage, so a single Canu run
  suffices and iterative re-assembly adds cost without benefit
- Reads are already assembled contigs or a reference-guided consensus is being built
  directly — this skill only operates on unassembled read sets
- --reassemble_max_iter is set to 0, which deactivates the iterative procedure entirely
edam_operation: http://edamontology.org/operation_0524
edam_topics:
- http://edamontology.org/topic_3174
- http://edamontology.org/topic_0196
tools:
- name: Canu
  role: Long-read de novo assembler run once per iteration; also provides corrected
    reads used preferentially in the cd-hit-est fallback
- name: cd-hit-est
  role: Clusters the longest pre-selected reads when a filtered read set produces
    no contigs, to pick representative reads as contig stand-ins
- name: Nextflow
  role: Workflow engine that orchestrates the iterative assembly, read mapping/removal,
    and fallback clustering within ViMOP
  repo: https://github.com/opr-group-bnitm/vimop
- name: ViMOP
  role: Pipeline implementing the iterative Canu re-assembly and cd-hit-est fallback
    logic
  repo: https://github.com/opr-group-bnitm/vimop
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioinformatics/btaf687
    title: 'ViMOP: a user-friendly and field-applicable pipeline for untargeted viral
      genome nanopore sequencing'
  source_content_hash: sha256:70138056d5759d861fc5d04d2dd71ffd730bf43db4ee65274b3dd1ced2841183
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
  - http://edamontology.org/topic_3174
  - http://edamontology.org/topic_0196
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# assembly-read-mapping-filtering

## Summary

ViMOP's iterative de novo assembly procedure for uneven viral genome abundance: Canu is run repeatedly on read sets, with reads mapping against newly generated contigs removed between iterations so low-abundance genomes can still be assembled from the remaining unmapped reads.

## When to use

Use when assembling untargeted nanopore FASTQ reads from samples with mixed viral abundances (e.g., co-infections), where an initial Canu assembly only recovers the most abundant genomes and residual reads must be re-assembled iteratively.

## When NOT to use

- The sample contains a single dominant virus at even coverage, so a single Canu run suffices and iterative re-assembly adds cost without benefit
- Reads are already assembled contigs or a reference-guided consensus is being built directly — this skill only operates on unassembled read sets
- --reassemble_max_iter is set to 0, which deactivates the iterative procedure entirely

## Inputs

- Filtered (Centrifuge-classified, host/contaminant-removed, optionally target-filtered) nanopore FASTQ read sets
- No-target read set (unless --assemble_notarget false)
- Parameters: --reassemble_max_iter, --assemble_notarget, --nocontigs_max_reads_precluster, --nocontigs_nreads

## Outputs

- Contig FASTA files per sample, e.g. no-target.contigs.fasta and re-assembly.contigs.fasta, written to the assembly subdirectory of the sample output folder
- Residual unmapped read sets carried between iterations

## How to apply

Run the Canu assembler on the filtered read set (and on the no-target read set unless --assemble_notarget false) to produce an initial contig set. Map all reads against the newly generated contigs and remove mapped reads, keeping only unmapped reads. Re-run Canu on the remaining reads and repeat the mapping-removal-assembly cycle up to --reassemble_max_iter iterations (0 disables the procedure), stopping early when no reads remain or Canu produces no contigs. If a target-filtered read set yields no contigs, take the longest --nocontigs_max_reads_precluster reads, cluster them with cd-hit-est, and select the longest --nocontigs_nreads reads (Canu-corrected if available, otherwise raw) as stand-ins for contigs in the subsequent target search. Write contig FASTA files (e.g., no-target.contigs.fasta and re-assembly.contigs.fasta) to the assembly subdirectory of the sample output folder. The rationale is that each Canu iteration consumes reads from abundant genomes, so subsequent iterations can assemble rarer viral genomes present in co-infections.

## Related tools

- **Canu** (Long-read de novo assembler run once per iteration; also provides corrected reads used preferentially in the cd-hit-est fallback)
- **cd-hit-est** (Clusters the longest pre-selected reads when a filtered read set produces no contigs, to pick representative reads as contig stand-ins)
- **Nextflow** (Workflow engine that orchestrates the iterative assembly, read mapping/removal, and fallback clustering within ViMOP) — https://github.com/opr-group-bnitm/vimop
- **ViMOP** (Pipeline implementing the iterative Canu re-assembly and cd-hit-est fallback logic) — https://github.com/opr-group-bnitm/vimop

## Examples

```
nextflow run OPR-group-BNITM/vimop --fastq <dir> --reassemble_max_iter 5 --nocontigs_max_reads_precluster 1000 --nocontigs_nreads 50
```

## Evaluation signals

- The assembly directory contains the expected contig FASTA files (e.g., no-target.contigs.fasta and re-assembly.contigs.fasta)
- The iterative procedure terminated only upon reaching --reassemble_max_iter cycles, exhaustion of reads, or a Canu run producing no contigs
- Each iteration's read filter removed reads mapping to the newly generated contigs, and the next Canu run used only the unmapped remainder
- When no contigs were produced for a filtered read set, exactly --nocontigs_nreads longest clustered reads (Canu-corrected preferred) were emitted as contig stand-ins
- Contigs from each iteration can be classified by Centrifuge and BLAST-searched against the virus reference database in the subsequent reference-identification step

## Limitations

- Canu needs substantial resources; the ViMOP README recommends at least 30 GB RAM and 16 CPUs, and lower-resource runs may fail or take much longer
- For large DNA viruses with extensive repetitive regions (e.g., mpox), assemblies may contain inaccuracies and should be regarded as draft genomes
- The mapping-based read removal only helps genomes sufficiently distinct that their reads do not map to contigs of other genomes; divergent but related co-infecting strains may not be separated
- The cd-hit-est fallback is a read stand-in, not a true assembly, so downstream target searches operate on individual long reads rather than consensus contigs

## Evidence

- [full_text] ViMOP runs Canu iteratively: at each iteration, reads mapping against any generated contigs are removed, and the remaining reads are used for another Canu run.: [evidence span withheld — anchor status: not_found]
- [methods] Iterative re-assembly is deactivated with --reassemble_max_iter 0: "After the initial assembly, an iterative re-assembly procedure is run (unless deactivated with `--reassemble_max_iter 0`)."
- [methods] cd-hit-est fallback clusters longest reads when no contigs are produced: "the longest X reads (set with `--nocontigs_max_reads_precluster`) are passed to a clustering by cd-hit-est."
- [methods] An assembly is run for each of the filtered read sets (explained in the previous section) and for the read set with no target filter.: "An assembly is run for each of the filtered read sets (explained in the previous section) and for the read set with no target filter."
- [readme] some tools like Canu need quite some resources: "However, this may or may not work, and it may take much longer, as some tools like Canu need quite some resources."
