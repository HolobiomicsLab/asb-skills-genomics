---
name: breakpoint-coordinate-validation-against-published-positions
description: Use when a long-read structural variant call (e.g. a translocation from
  ONT/PacBio data aligned with minimap2) must be verified against published breakpoint
  coordinates within small base-pair offsets.
when_to_use_negative:
- The input is short-read sequencing data, which the article notes cannot resolve
  the large-scale rearrangements targeted by AgileStructure
- No published or reference breakpoint coordinates exist to compare against, so offset-based
  validation has no ground truth
- The goal is fully automated SV discovery rather than user-guided annotation and
  validation of a specific rearrangement
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3175
tools:
- name: AgileStructure
  role: Desktop application used to visually inspect split-read alignments at candidate
    breakpoints and annotate the translocation via the Variant determination menu
  repo: https://github.com/msjimc/AgileStructure
- name: minimap2
  role: Long-read aligner producing the primary-only SAM/BAM alignments (with -ax
    map-ont and --secondary=no) that AgileStructure examines
  repo: https://github.com/lh3/minimap2
- name: Samtools
  role: Converts minimap2 SAM output to position-sorted, indexed BAM suitable for
    loading into AgileStructure
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioinformatics/btag294
    title: Utilization of long-read sequencing for the detection of structural rearrangements
      with AgileStructure
  source_content_hash: sha256:f9084bdbc67712318c91c43b2ad15005658738cc9775c1cad3c7656981f84b61
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/msjimc/AgileStructure
  tools:
  - AgileStructure
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
n_claims: 7
---

# Breakpoint coordinate validation against published positions

## Summary

Validates structural-variant breakpoint coordinates called from aligned long-read data by comparing them against published reference positions within small reported offsets, as demonstrated for the t(chr6;chr8) translocation in Hu et al. (2019) sample SRR9982132 with AgileStructure.

## When to use

Use when a long-read structural variant call (e.g. a translocation from ONT/PacBio data aligned with minimap2) must be verified against published breakpoint coordinates within small base-pair offsets.

## When NOT to use

- The input is short-read sequencing data, which the article notes cannot resolve the large-scale rearrangements targeted by AgileStructure
- No published or reference breakpoint coordinates exist to compare against, so offset-based validation has no ground truth
- The goal is fully automated SV discovery rather than user-guided annotation and validation of a specific rearrangement

## Inputs

- Long-read sequencing dataset (e.g. SRA accession SRR9982132, ONT/PacBio reads)
- Human reference genome sequence (hg19, from UCSC Genome Browser)
- Published breakpoint coordinates for the rearrangement of interest

## Outputs

- Translocation annotation with breakpoint coordinates (e.g. t(chr6;chr8) with g.-annotated positions)
- Supporting split-read/junction read information per breakpoint
- Annotation file, e.g. translocation_annotation.txt

## How to apply

Align the long-read dataset (e.g. SRA SRR9982132) to the reference (hg19 from UCSC) with minimap2 using -ax map-ont and --secondary=no, then convert SAM to BAM, sort and index with Samtools. Load the indexed BAM into AgileStructure and inspect split-read alignments at the published breakpoint loci (e.g. chr6:167,281,717 and chr8:113,696,089), where a putative breakpoint is called when 2 or more split reads have junction positions within 250 nucleotides of each other. Annotate the rearrangement with the Variant determination menu using the consensus of per-read annotations, and record the resulting t(chr6;chr8) coordinates (published exemplar: g.167 281 716; g.113 696 100 on chr6 and g.167 281 719; g.113 696 098 on chr8). Judge success by whether the called chr6 breakpoint falls within -1/+2 bp of g.167,281,717 and the chr8 breakpoint within +9/+11 bp of g.113,696,089, matching the offsets observed in the paper's exemplar.

## Related tools

- **AgileStructure** (Desktop application used to visually inspect split-read alignments at candidate breakpoints and annotate the translocation via the Variant determination menu) — https://github.com/msjimc/AgileStructure
- **minimap2** (Long-read aligner producing the primary-only SAM/BAM alignments (with -ax map-ont and --secondary=no) that AgileStructure examines) — https://github.com/lh3/minimap2
- **Samtools** (Converts minimap2 SAM output to position-sorted, indexed BAM suitable for loading into AgileStructure)

## Examples

```
minimap2 -ax map-ont --secondary=no hg19.fa SRR9982132.fastq > aln.sam && samtools sort -o aln.bam aln.sam && samtools index aln.bam
```

## Evaluation signals

- Called chr6 breakpoint falls within -1/+2 bp of the published position g.167,281,717
- Called chr8 breakpoint falls within +9/+11 bp of the published position g.113,696,089
- At the breakpoint loci, at least 2 split reads show junction positions within 250 nucleotides of each other
- Annotations derived from primary alignments on chromosome 6 and chromosome 8 agree within a few base pairs (exemplar: g.167 281 716 vs g.167 281 719; g.113 696 100 vs g.113 696 098)
- Output annotation file records both breakpoint coordinates and supporting read information

## Limitations

- Breakpoint calls depend on primary alignments; secondary alignments are excluded (--secondary=no), so alignment artifacts may shift called positions by several base pairs relative to published coordinates
- AgileStructure requires user-guided exploration of breakpoint-spanning reads rather than fully automated calling, and runs on POSIX systems via WINE
- Validation depends on availability of published ground-truth coordinates; offsets up to ~11 bp (as in the chr8 exemplar) should be tolerated

## Evidence

- [other] AgileStructure identified the chr6;chr8 translocation as t(chr6; chr8) (g.167 281 716; g.113 696 100) using primary alignments on chromosome 6: [evidence span withheld — anchor status: not_found]
- [other] a putative breakpoint is called when 2 or more split reads have junction positions within 250 nucleotides of each other: [evidence span withheld — anchor status: not_found]
- [abstract] We developed AgileStructure, a desktop application for locating and annotating large-scale genomic rearrangements using aligned long-read data: "We developed AgileStructure, a desktop application for locating and annotating large‑scale genomic rearrangements using aligned long‑read data."
- [readme] the data is typically aligned to the appropriate genome using a long-read-specific aligner such as minimap2: "the data is typically aligned to the appropriate genome using a long-read-specific aligner such as minimap2"
- [abstract] The software enables user-guided exploration of breakpoint-spanning reads, supporting accurate interpretation of complex events: "The software enables user‑guided exploration of breakpoint‑spanning reads, supporting accurate interpretation of complex events"
