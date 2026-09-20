---
name: gencode-gtf-annotation-parsing
description: Use when long Nanopore/PacBio reads must be spliced-aligned to a reference
  genome and annotated splice junctions from a Gencode GTF (e.g. v38, vM18) need to
  be converted into a BED file for minimap2's --junc-bed.
when_to_use_negative:
- The reads will be aligned in an unspliced (genome/short-read) mode where --junc-bed
  is not used, so annotated junctions are irrelevant.
- A BED file of splice junctions already exists for the reference annotation, making
  GTF re-parsing redundant.
- Novel junction discovery is the goal and the annotation should only be used downstream
  (e.g. as SiCeLoRe junction validation evidence with juncCo) rather than as an alignment
  guide.
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_0085
- http://edamontology.org/topic_0102
tools:
- name: minimap2
  role: provides paftools.js used to convert the Gencode GTF into a splice-junction
    BED file; its --junc-bed option consumes junctions.bed during '-ax splice' alignment
  repo: https://github.com/lh3/minimap2
- name: samtools
  role: part of the SiCeLoRe-2.1 toolchain accompanying the spliced-alignment step
    that consumes the junction BED
  repo: https://github.com/samtools/samtools
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-020-17800-6
    title: High throughput error corrected Nanopore single cell transcriptome sequencing
  source_content_hash: sha256:6a1bc9b023bf2671ceda47714da5cdd8b3ddaead2699fadef6537430123b4541
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/lh3/minimap2
  tools:
  - minimap2
  - samtools
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_0085
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# gencode-gtf-annotation-parsing

## Summary

Convert a Gencode GTF annotation into a BED file of exon-exon splice junctions (e.g. using minimap2's companion paftools.js) so annotated junctions can be supplied to minimap2's --junc-bed for spliced alignment of Nanopore reads.

## When to use

Use when long Nanopore/PacBio reads must be spliced-aligned to a reference genome and annotated splice junctions from a Gencode GTF (e.g. v38, vM18) need to be converted into a BED file for minimap2's --junc-bed.

## When NOT to use

- The reads will be aligned in an unspliced (genome/short-read) mode where --junc-bed is not used, so annotated junctions are irrelevant.
- A BED file of splice junctions already exists for the reference annotation, making GTF re-parsing redundant.
- Novel junction discovery is the goal and the annotation should only be used downstream (e.g. as SiCeLoRe junction validation evidence with juncCo) rather than as an alignment guide.

## Inputs

- Gencode GTF annotation file (e.g. gencode.v38.primary_assembly.annotation.gtf.gz from release 38, or Gencode vM18 for mouse)
- paftools.js script from the minimap2 package

## Outputs

- junctions.bed — BED-format file with one splice junction per line (chromosome, junction start/end coordinates, strand)

## How to apply

Download the Gencode annotation GTF (e.g. gencode.v38.primary_assembly.annotation.gtf.gz from release 38, or the mouse vM18 release for mouse genome alignments) and decompress it so exon records can be parsed. Use paftools.js, a companion script shipped with minimap2, to extract all annotated exon-exon junctions from the GTF exon coordinates, writing one junction per line. Emit the junctions as a BED-format file named junctions.bed with chromosome, junction start/end coordinates, and strand fields. Supply this BED via minimap2's --junc-bed option during '-ax splice' mapping so Nanopore reads are aligned against known junctions. Validate that junctions.bed exists, parses as valid BED, and that its records are consistent with the exon structure of the source Gencode GTF.

## Related tools

- **minimap2** (provides paftools.js used to convert the Gencode GTF into a splice-junction BED file; its --junc-bed option consumes junctions.bed during '-ax splice' alignment) — https://github.com/lh3/minimap2
- **samtools** (part of the SiCeLoRe-2.1 toolchain accompanying the spliced-alignment step that consumes the junction BED) — https://github.com/samtools/samtools

## Examples

```
wget https://ftp.ebi.ac.uk/pub/databases/gencode/Gencode_human/release_38/gencode.v38.primary_assembly.annotation.gtf.gz && gunzip gencode.v38.primary_assembly.annotation.gtf.gz && paftools.js gff2bed gencode.v38.primary_assembly.annotation.gtf > junctions.bed
```

## Evaluation signals

- junctions.bed exists and parses as a valid BED file (tab-delimited chromosome, start, end, and strand fields)
- Junction records in junctions.bed are consistent with exon coordinates in the source Gencode GTF
- Every junction line is one per line as emitted by paftools.js, with no malformed or empty records
- The junction BED is accepted by minimap2 --junc-bed and spliced alignments ('-ax splice') show reads crossing annotated junctions

## Limitations

- The junction BED only reflects annotated junctions from the chosen Gencode release (v38 human or vM18 mouse); novel junctions absent from the annotation will not guide spliced alignment
- The GTF must be decompressed before exon records can be parsed by paftools.js
- Strand and coordinate conventions of the BED must match minimap2's --junc-bed expectations; incorrect fields would silently mis-guide spliced alignment

## Evidence

- [methods] Extract all annotated exon-exon junctions from the GTF exon coordinates using paftools.js from the minimap2 package, writing one junction per line.: [evidence span withheld — anchor status: not_found]
- [methods] verify junctions.bed exists, parses as a valid BED file, and contains junction records consistent with the exon structure: [evidence span withheld — anchor status: not_found]
- [methods] wget https://ftp.ebi.ac.uk/pub/databases/gencode/Gencode_human/release_38/gencode.v38.primary_assembly.annotation.gtf.gz: "wget https://ftp.ebi.ac.uk/pub/databases/gencode/Gencode_human/release_38/gencode.v38.primary_assembly.annotation.gtf.gz"
- [readme] * [minimap2](https://github.com/lh3/minimap2): "* [samtools](https://github.com/samtools/samtools)
* [minimap2](https://github.com/lh3/minimap2)"
