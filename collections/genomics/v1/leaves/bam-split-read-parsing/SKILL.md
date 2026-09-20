---
name: bam-split-read-parsing
description: Use when an indexed BAM of long reads (e.g. ONT reads aligned with minimap2
  -ax map-ont --secondary=no) must be scanned for split-read junctions to locate large-scale
  genomic rearrangement breakpoints.
when_to_use_negative:
- Input reads are short Illumina reads rather than PacBio/ONT long reads — short reads
  rarely span breakpoints and cannot supply the linked primary/secondary split alignments
  this method relies on.
- The BAM was produced with secondary alignments suppressed inconsistently or by an
  aligner that does not emit linked split-alignment records, since junction positions
  cannot be reconstructed from primary alignments alone.
- The goal is de novo variant calling with base-level sensitivity for SNVs or small
  indels rather than large-scale structural rearrangement breakpoint detection.
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3175
tools:
- name: AgileStructure
  role: 'Desktop application whose breakpoint detection this skill reproduces: retains
    genomic positions of split-read junctions and calls putative breakpoints from
    clustered junction positions'
  repo: https://github.com/msjimc/AgileStructure
- name: minimap2
  role: Long-read aligner used with '-ax map-ont' and '--secondary=no' to produce
    the BAM split-read alignment structure that the parser reads
  repo: https://github.com/lh3/minimap2
- name: Samtools
  role: Sorts and indexes the minimap2 BAM so primary and linked secondary alignments
    can be read efficiently
- name: MakeSVGenome
  role: Generates synthetic chromosomes with known structural rearrangements and faux
    fixed-length, alternating-orientation long reads to benchmark the parsing and
    classification rules
  repo: https://github.com/msjimc/MakeSVGenome
provenance:
  source_task_ids:
  - task_001
  - task_006
  source_papers:
  - doi: 10.1093/bioinformatics/btag294
    title: Utilization of long-read sequencing for the detection of structural rearrangements
      with AgileStructure
  source_content_hash: sha256:66719805a3d38a41a41abd941fb8d08c775f9512a1ebe9e7e68a2d3baf4fdf0a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/msjimc/AgileStructure
  tools:
  - AgileStructure
  - MakeSVGenome
  - Samtools
  - minimap2
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3175
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# bam-split-read-parsing

## Summary

Parses an indexed long-read BAM file to recover split-read alignment junctions and call putative structural-rearrangement breakpoints, the core detection mechanism behind AgileStructure's breakpoint identification from aligned PacBio/ONT data.

## When to use

Use when an indexed BAM of long reads (e.g. ONT reads aligned with minimap2 -ax map-ont --secondary=no) must be scanned for split-read junctions to locate large-scale genomic rearrangement breakpoints.

## When NOT to use

- Input reads are short Illumina reads rather than PacBio/ONT long reads — short reads rarely span breakpoints and cannot supply the linked primary/secondary split alignments this method relies on.
- The BAM was produced with secondary alignments suppressed inconsistently or by an aligner that does not emit linked split-alignment records, since junction positions cannot be reconstructed from primary alignments alone.
- The goal is de novo variant calling with base-level sensitivity for SNVs or small indels rather than large-scale structural rearrangement breakpoint detection.

## Inputs

- Indexed BAM file of long reads aligned with minimap2 ('-ax map-ont', '--secondary=no') and sorted/indexed with Samtools
- Alignment records containing primary alignments and linked secondary (split) alignments plus tag data
- Optionally, synthetic test datasets generated with MakeSVGenome (e.g. 68 examples of simple/complex insertion-deletion rearrangements on hg19/mm10 chromosomes 7 and 8 with 10 kb faux reads at 1 kb spacing)

## Outputs

- breakpoints.tsv with one row per called breakpoint: chromosome, position, supporting read count
- classification_results.csv with per-rearrangement predicted labels, split-read column counts, and mismatches vs expected classes
- Per-locus lists of clustered junction positions supporting each breakpoint

## How to apply

Open the indexed BAM (header plus alignment records) and read primary alignments together with their linked secondary alignments; for each split read, extract the genomic positions at which the primary alignment starts or stops and where its linked secondary alignment starts or stops, including junction positions retrieved from tag data. Cluster these junction positions at the locus of interest and call a putative breakpoint when 2 or more junction positions fall within 250 nucleotides of each other, writing chromosome, position, and supporting read count per breakpoint. The linked primary/secondary alignment structure is produced by aligning ONT long reads with minimap2 using the '-ax map-ont' preset and '--secondary=no', so alignment flags directly determine which junctions are recoverable. For simple-versus-complex classification, group start/stop positions into distinct columns of split-read alignments: 2 columns (3 for insertions) indicates a simple rearrangement, while 3 or 4 columns indicate a complex one, matching AgileStructure's expectation that complex events present as 3 or 4 columns depending on whether one or both sides are complex.

## Related tools

- **AgileStructure** (Desktop application whose breakpoint detection this skill reproduces: retains genomic positions of split-read junctions and calls putative breakpoints from clustered junction positions) — https://github.com/msjimc/AgileStructure
- **minimap2** (Long-read aligner used with '-ax map-ont' and '--secondary=no' to produce the BAM split-read alignment structure that the parser reads) — https://github.com/lh3/minimap2
- **Samtools** (Sorts and indexes the minimap2 BAM so primary and linked secondary alignments can be read efficiently)
- **MakeSVGenome** (Generates synthetic chromosomes with known structural rearrangements and faux fixed-length, alternating-orientation long reads to benchmark the parsing and classification rules) — https://github.com/msjimc/MakeSVGenome

## Examples

```
minimap2 -ax map-ont --secondary=no ref.fa ont_reads.fq.gz | samtools sort -o aligned.bam && samtools index aligned.bam
```

## Evaluation signals

- breakpoints.tsv is a well-formed TSV whose coordinate and support-count fields parse, and every reported breakpoint is backed by at least 2 junction positions within the 250-nucleotide window
- Predicted simple/complex labels match the guide-annotated expected classes across the 68 MakeSVGenome examples (32 simple, 36 complex), allowing disagreement only for the two inherently ambiguous scenarios (ring chromosome vs tandem duplication; offset inversion vs inverted insertion-deletion with deletion at the insertion site)
- Column-count rule holds: split-read groups with 2 columns (3 for insertions) label as simple; 3 or 4 columns label as complex
- Junction positions are only recovered for reads whose BAM records contain linked secondary alignments consistent with minimap2 '--secondary=no' minimap2 -ax map-ont output

## Limitations

- Two scenarios are inherently ambiguous under the column-count rule: ring chromosome vs tandem duplication, and offset inversion vs inverted insertion-deletion with deletion at the insertion site
- Existing alignment/visualisation programs (minimap2/lra plus IGV) are limited for rapid detection and annotation of structural rearrangements, which motivates this automated parsing approach
- Approximately 50% of rare-disease pathogenic variants are unresolved by short-read data, so this method presupposes long-read (PacBio or ONT) input; it does not recover variants from short-read BAMs

## Evidence

- [other] AgileStructure retains the genomic positions of junctions between the primary and secondary alignments of split reads, and identifies a putative breakpoint when 2 or more of these positions occur within 250 nucleotides of each other.: [evidence span withheld — anchor status: not_found]
- [intro] the data is typically aligned to the appropriate genome using a long-read-specific aligner such as minimap2: "the data is typically aligned to the appropriate genome using a long-read-specific aligner such as minimap2"
- [readme] The read data produced, consists of reads of fixed length and spacing, that alternate between forward and reverse orientations.: [evidence span withheld — anchor status: external_doc]
- [intro] these programs are limited in their ability to allow the rapid detection and annotation of structural rearrangements; consequently, we have developed AgileStructure specifically for this purpose.: "these programs are limited in their ability to allow the rapid detection and annotation of structural rearrangements; consequently, we have developed AgileStructure specifically for this purpose."
- [abstract] The software enables user‑guided exploration of breakpoint‑spanning reads, supporting accurate interpretation of complex events: "The software enables user‑guided exploration of breakpoint‑spanning reads, supporting accurate interpretation of complex events"
