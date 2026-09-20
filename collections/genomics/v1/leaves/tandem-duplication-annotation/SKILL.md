---
name: tandem-duplication-annotation
description: Use when long-read sequencing data are suspected to carry a large tandem
  duplication (e.g. an unintended duplication in a knock-out line) and split-read
  or soft-clip evidence in a BAM must be converted into precise breakpoint coordinates.
when_to_use_negative:
- The variant of interest is a small indel or SNV resolvable with short-read sequencing
  data, rather than a large-scale rearrangement requiring long reads.
- No sorted, indexed alignment BAM exists and reads have not been aligned with a long-read-specific
  aligner such as minimap2 or lra.
- The task is bulk automated SV calling across many samples rather than user-guided
  inspection and annotation of a specific breakpoint event.
edam_operation: http://edamontology.org/operation_3228
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3175
tools:
- name: AgileStructure
  role: Desktop application used to visualise split reads spanning the breakpoint,
    annotate the duplication via Variant determination > Use soft clip data, and merge
    per-read annotations into a single generic SV annotation
  repo: https://github.com/msjimc/AgileStructure
- name: minimap2
  role: Long-read-specific aligner used with the '-ax map-ont' preset and '--secondary=no'
    to align ONT reads to the mm10 reference genome
  repo: https://github.com/lh3/minimap2
- name: Samtools
  role: Converts the minimap2 SAM output to BAM, sorts by genomic position, and indexes
    the BAM so it can be loaded in AgileStructure
- name: MakeSVGenome
  role: Creates chromosomes with structural rearrangements and synthetic long-read
    data that can be used to test the accuracy of structural variant detection programs
  repo: https://github.com/msjimc/MakeSVGenome
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioinformatics/btag294
    title: Utilization of long-read sequencing for the detection of structural rearrangements
      with AgileStructure
  source_content_hash: sha256:adab5cc1edc8ead2d2b2dbb65f8103b97e412a87988a66ba9c8d35bebce2f26e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/msjimc/AgileStructure
  tools:
  - AgileStructure
  - MakeSVGenome
  - Samtools
  - minimap2
  edam_operation: http://edamontology.org/operation_3228
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3175
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# tandem-duplication-annotation

## Summary

Annotate large-scale tandem duplications from aligned long-read (PacBio/ONT) data by user-guided inspection of split and soft-clipped reads in AgileStructure, then validate the resulting SV coordinates against published breakpoints. Demonstrated on the Agmo knockout mouse dataset (SRR12783028) where AgileStructure recovered chr12:37,206,133_37,300,424dup.

## When to use

Use when long-read sequencing data are suspected to carry a large tandem duplication (e.g. an unintended duplication in a knock-out line) and split-read or soft-clip evidence in a BAM must be converted into precise breakpoint coordinates.

## When NOT to use

- The variant of interest is a small indel or SNV resolvable with short-read sequencing data, rather than a large-scale rearrangement requiring long reads.
- No sorted, indexed alignment BAM exists and reads have not been aligned with a long-read-specific aligner such as minimap2 or lra.
- The task is bulk automated SV calling across many samples rather than user-guided inspection and annotation of a specific breakpoint event.

## Inputs

- Long-read sequencing data from a PacBio or ONT sequencer (e.g. NCBI SRA sample SRR12783028)
- Reference genome FASTA (mouse mm10 obtained from the UCSC Genome Browser)
- Sorted, indexed BAM file of long reads aligned with minimap2
- Published variant coordinates for comparison (chr12:37,206,133-37,300,425dup)

## Outputs

- Saved AgileStructure structural variant annotation of the tandem duplication (chr12:37,206,133_37,300,424dup, mm10)
- Per-read breakpoint annotations and a per-read-annotation-median-based coordinate
- Validation report comparing tool coordinates to published breakpoints (agreement on 5' side, -1 bp on 3' side)

## How to apply

Align the long reads to the reference (e.g. mouse mm10 from UCSC) with minimap2 using the '-ax map-ont' preset and '--secondary=no', then convert, sort and index the SAM with samtools to produce a sorted indexed BAM. Load the BAM in AgileStructure and screen the chromosome region of interest (chr12 around the Agmo locus) in blocks of up to 10 Mb, identifying the breakpoint by locating two or more split reads whose primary and secondary alignment junction positions fall within 250 nucleotides of each other. Select the split reads linked to the breakpoint in the upper and lower panels and run Variant determination > Use soft clip data to annotate the duplication, letting AgileStructure merge the per-read annotations into a single generic annotation and also record a per-read-annotation-median-based coordinate. Validate by checking the saved annotation against published coordinates: the Agmo duplication should be reported as chr12:37,206,133_37,300,424dup (mm10), matching the published chr12:37,206,133–37,300,425dup on the 5' side and within 1 bp (-1 bp) on the 3' side.

## Related tools

- **AgileStructure** (Desktop application used to visualise split reads spanning the breakpoint, annotate the duplication via Variant determination > Use soft clip data, and merge per-read annotations into a single generic SV annotation) — https://github.com/msjimc/AgileStructure
- **minimap2** (Long-read-specific aligner used with the '-ax map-ont' preset and '--secondary=no' to align ONT reads to the mm10 reference genome) — https://github.com/lh3/minimap2
- **Samtools** (Converts the minimap2 SAM output to BAM, sorts by genomic position, and indexes the BAM so it can be loaded in AgileStructure)
- **MakeSVGenome** (Creates chromosomes with structural rearrangements and synthetic long-read data that can be used to test the accuracy of structural variant detection programs) — https://github.com/msjimc/MakeSVGenome

## Examples

```
minimap2 -ax map-ont --secondary=no mm10.fa SRR12783028.fq.gz > aln.sam && samtools sort -o sorted.bam aln.sam && samtools index sorted.bam
```

## Evaluation signals

- The saved annotation reports the duplication as chr12:37,206,133_37,300,424dup (mm10).
- The 5' breakpoint exactly matches the published chr12:37,206,133 coordinate and the 3' breakpoint differs by only -1 bp from the published 37,300,425.
- At least two split reads with primary and secondary alignment junction positions within 250 nucleotides of each other support the called breakpoint.
- A per-read-annotation-median-based coordinate is recorded alongside the generic annotation.
- The BAM was produced with minimap2 '-ax map-ont' and '--secondary=no', then sorted and indexed, before loading in AgileStructure.

## Limitations

- AgileStructure relies on user-guided exploration of breakpoint-spanning reads, so annotation quality depends on manual inspection of the correct chr12 blocks (screened in up to 10 Mb chunks) rather than fully automated calling.
- Coordinates can differ by around 1 bp from published values (here -1 bp on the 3' side), so exact-coordinate agreement should not be expected.
- The program is a Windows application; on POSIX systems it must be run via WINE on Intel or AMD CPUs.
- Split-read/soft-clip-based breakpoint detection presumes the aligner preserved secondary alignments appropriately; secondary alignments were disabled with '--secondary=no' in this workflow.

## Evidence

- [abstract] AgileStructure is a desktop application for locating and annotating large-scale genomic rearrangements using aligned long-read data.: "We developed AgileStructure, a desktop application for locating and annotating large‑scale genomic rearrangements using aligned long‑read data."
- [intro] Long-read data are aligned with a long-read-specific aligner such as minimap2 before inspection.: "the data is typically aligned to the appropriate genome using a long-read-specific aligner such as minimap2"
- [abstract] The software supports user-guided exploration of breakpoint-spanning reads for complex events.: "The software enables user‑guided exploration of breakpoint‑spanning reads, supporting accurate interpretation of complex events"
- [readme] MakeSVGenome generates synthetic rearranged chromosomes and reads for testing SV detection accuracy.: [evidence span withheld — anchor status: external_doc]
- [readme] minimap2 provides the map-ont preset used for Oxford Nanopore genomic reads.: [evidence span withheld — anchor status: not_found]
