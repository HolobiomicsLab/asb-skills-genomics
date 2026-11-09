---
name: minimap2-junc-bed-configuration
description: Use when aligning Nanopore long reads to a genome in minimap2 spliced
  mode and annotated exon-exon junctions must be extracted from a Gencode GTF into
  a BED file for the --junc-bed option.
when_to_use_negative:
- The GTF annotation build does not match the reference genome build used for minimap2
  alignment (e.g. Gencode v38 hg38 annotation against a mouse reference)
- Input reads are short Illumina reads that do not require minimap2 spliced alignment
  with known-junction guidance
- A validated junction BED file already exists for the same annotation release — re-extracting
  from the GTF is unnecessary
edam_operation: http://edamontology.org/operation_0362
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0622
tools:
- name: minimap2
  role: Provides paftools.js (companion script) used to convert the Gencode GTF exon
    records into a BED junction file, and performs '-ax splice' alignment accepting
    the BED via --junc-bed
  repo: https://github.com/lh3/minimap2
- name: samtools
  role: Post-processes the splice-aligned BAM (sort/index) so junction-guided alignments
    are usable in later SiCeLoRe steps
  repo: https://github.com/samtools/samtools
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-020-17800-6
    title: High throughput error corrected Nanopore single cell transcriptome sequencing
  source_content_hash: sha256:fca9734366e8417aaa4683ee63d78405417a531ad2edf43dfd4b8412993866d0
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/lh3/minimap2
  tools:
  - minimap2
  - samtools
  edam_operation: http://edamontology.org/operation_0362
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 5
---

# minimap2-junc-bed-configuration

## Summary

Prepares known splice junctions for spliced long-read alignment by converting the Gencode GTF annotation (e.g. Gencode vM18 for mouse, v38 for hg38) into a BED file with paftools.js, then supplying it to minimap2 via --junc-bed during '-ax splice' mapping of Nanopore reads.

## When to use

Use when aligning Nanopore long reads to a genome in minimap2 spliced mode and annotated exon-exon junctions must be extracted from a Gencode GTF into a BED file for the --junc-bed option.

## When NOT to use

- The GTF annotation build does not match the reference genome build used for minimap2 alignment (e.g. Gencode v38 hg38 annotation against a mouse reference)
- Input reads are short Illumina reads that do not require minimap2 spliced alignment with known-junction guidance
- A validated junction BED file already exists for the same annotation release — re-extracting from the GTF is unnecessary

## Inputs

- Gencode annotation GTF (.gtf.gz), matched to the reference genome build (e.g. gencode.v38.primary_assembly.annotation.gtf.gz for hg38; Gencode vM18 for mouse)
- Nanopore (or PacBio) long-read FASTQ from the scRNA-seq library
- Reference genome FASTA (hg38 or mouse)

## Outputs

- junctions.bed: BED-format splice-junction file (chromosome, junction start/end coordinates, strand), one junction per line
- Splice-aligned SAM/BAM from minimap2 '-ax splice' with --junc-bed, for downstream SiCeLoRe UMI assignment steps

## How to apply

Download the Gencode annotation GTF for the matching reference build (e.g. gencode.v38.primary_assembly.annotation.gtf.gz from the Gencode FTP release 38, or Gencode vM18 for the mouse genome) and decompress it so exon records can be parsed. Extract all annotated exon-exon junctions from the GTF exon coordinates using paftools.js, a companion script shipped with the minimap2 package, writing one junction per line. Emit the result as junctions.bed with chromosome, junction start/end coordinates, and strand fields. Pass the BED file via minimap2's --junc-bed option when running '-ax splice' so Nanopore reads are guided to annotated junctions, with samtools used to sort and index the resulting BAM. The rationale is that supplying known junctions improves splice alignment of error-prone long reads against annotated transcript structures.

## Related tools

- **minimap2** (Provides paftools.js (companion script) used to convert the Gencode GTF exon records into a BED junction file, and performs '-ax splice' alignment accepting the BED via --junc-bed) — https://github.com/lh3/minimap2
- **samtools** (Post-processes the splice-aligned BAM (sort/index) so junction-guided alignments are usable in later SiCeLoRe steps) — https://github.com/samtools/samtools

## Examples

```
wget https://ftp.ebi.ac.uk/pub/databases/gencode/Gencode_human/release_38/gencode.v38.primary_assembly.annotation.gtf.gz
```

## Evaluation signals

- junctions.bed exists and parses as a valid BED file with chromosome, start/end, and strand fields
- One junction record per line, and junction coordinates are consistent with the exon structure of the source Gencode GTF
- The BED file is accepted by minimap2's --junc-bed option and alignments are produced with '-ax splice'
- Junction coordinates agree with the genome build of the downloaded Gencode release (e.g. release_38 GTF for hg38)

## Limitations

- Junction guidance only covers junctions present in the supplied Gencode annotation; novel junctions are not in the BED file
- The annotation must match the reference genome and build, or junction coordinates will be wrong (e.g. Gencode vM18 mouse vs Gencode v38 human)
- The GTF must be decompressed before exon records can be parsed by paftools.js

## Evidence

- [methods] The splice junction input for minimap2 is prepared by converting the Gencode vM18 GTF annotation into a BED file using paftools.js, a companion script of minimap2.: [evidence span withheld — anchor status: not_found]
- [methods] Gencode v38 annotation download URL from the methods: "wget https://ftp.ebi.ac.uk/pub/databases/gencode/Gencode_human/release_38/gencode.v38.primary_assembly.annotation.gtf.gz"
- [methods] Junction BED passed via minimap2's --junc-bed during spliced mapping: [evidence span withheld — anchor status: not_found]
- [readme] minimap2 listed as a required dependency in the SiCeLoRe 2.1 README: "* [minimap2](https://github.com/lh3/minimap2)"
