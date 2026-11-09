---
name: ambiguous-sv-scenario-discrimination
description: Use when annotating long-read (PacBio/ONT) SV breakpoints with AgileStructure
  and the observed split-read/junction pattern could represent either member of a
  documented ambiguous pair — ring chromosome vs tandem duplication, or offset inversion
  vs inverted insertion-deletion.
when_to_use_negative:
- The input is short-read sequencing data — AgileStructure and its breakpoint-spanning
  read exploration are designed for aligned PacBio or ONT long-read data
- A clinically definitive discrimination between ring chromosome and tandem duplication
  is required without orthogonal validation — this pair is documented as not readily
  distinguishable by the tool
- The goal is SNV or small indel calling rather than large-scale structural rearrangement
  detection and annotation
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_2885
tools:
- name: AgileStructure
  role: Desktop application whose two-panel interface and simple/complex rearrangement
    pathways produce the annotations; its resolution limits define the ambiguous pairs
  repo: https://github.com/msjimc/AgileStructure
- name: MakeSVGenome
  role: Generates chromosomes with known structural rearrangements and synthetic fixed-length,
    alternating-orientation long reads so ground truth for each ambiguous pair is
    known
  repo: msjimc/MakeSVGenome
- name: minimap2
  role: Long-read-specific aligner producing the split-read alignments that feed AgileStructure
    breakpoint calling
  repo: https://github.com/lh3/minimap2
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioinformatics/btag294
    title: Utilization of long-read sequencing for the detection of structural rearrangements
      with AgileStructure
  source_content_hash: sha256:6c24dbfb2568c2ad797511f4a932686a26838e71c60f212d2c9ae64ed6430af4
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
  - http://edamontology.org/topic_2885
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# ambiguous-sv-scenario-discrimination

## Summary

Recognizes and flags the two documented scenarios that AgileStructure cannot unambiguously resolve when annotating long-read structural rearrangements: ring chromosome vs tandem duplication, and offset inversion vs inverted insertion-deletion with loss at the insertion site. Grounded in the 66/68 unambiguous benchmark on MakeSVGenome synthetic rearrangements.

## When to use

Use when annotating long-read (PacBio/ONT) SV breakpoints with AgileStructure and the observed split-read/junction pattern could represent either member of a documented ambiguous pair — ring chromosome vs tandem duplication, or offset inversion vs inverted insertion-deletion.

## When NOT to use

- The input is short-read sequencing data — AgileStructure and its breakpoint-spanning read exploration are designed for aligned PacBio or ONT long-read data
- A clinically definitive discrimination between ring chromosome and tandem duplication is required without orthogonal validation — this pair is documented as not readily distinguishable by the tool
- The goal is SNV or small indel calling rather than large-scale structural rearrangement detection and annotation

## Inputs

- Indexed BAM of long reads aligned to hg19/mm10 (minimap2 -ax map-ont --secondary=no, sorted/indexed with Samtools)
- MakeSVGenome-generated synthetic chromosomes containing 32 simple and 36 complex insertion-deletion rearrangements, with faux 10 kb reads at 1 kb intervals in alternating orientations
- AgileStructure example aligned read data from the GitHub repository (optional substitute for self-generated data)

## Outputs

- Per-rearrangement annotation records for each of the 68 synthetic structural rearrangements
- Flags marking the 2 documented ambiguous scenario pairs (ring chromosome vs tandem duplication; offset inversion vs inverted insertion-deletion)
- A Table 2-style tally of unambiguous vs ambiguous annotations (expected: 66/68 unambiguous)

## How to apply

Align long reads to hg19/mm10 with minimap2 (-ax map-ont, --secondary=no), convert/sort/index the SAM to BAM with Samtools, then screen each candidate locus in AgileStructure in blocks of up to 10 Mb; breakpoints are called when 2 or more primary/secondary junction positions cluster within 250 nucleotides. Route events showing 2 columns of split reads (3 for insertions) through the simple-rearrangement pathway (Variant determination > Use soft clip data menus) and events with 3 or 4 columns through the Complex Rearrangement window. For each of the annotated events, record whether the junction/soft-clip signature resolves to a single rearrangement class; only ring chromosome vs tandem duplication and offset inversion vs inverted insertion-deletion (loss at the insertion site) should remain unresolvable. The benchmark expectation on the 68 MakeSVGenome in silico rearrangements (32 simple + 36 complex insertion-deletion) is exactly 66 unambiguous and 2 flagged ambiguous annotations, matching Table 2 of the paper.

## Related tools

- **AgileStructure** (Desktop application whose two-panel interface and simple/complex rearrangement pathways produce the annotations; its resolution limits define the ambiguous pairs) — https://github.com/msjimc/AgileStructure
- **MakeSVGenome** (Generates chromosomes with known structural rearrangements and synthetic fixed-length, alternating-orientation long reads so ground truth for each ambiguous pair is known) — msjimc/MakeSVGenome
- **minimap2** (Long-read-specific aligner producing the split-read alignments that feed AgileStructure breakpoint calling) — https://github.com/lh3/minimap2

## Examples

```
minimap2 -ax map-ont ref.fa ont.fq.gz > aln.sam
```

## Evaluation signals

- Annotation tally over the 68 in silico rearrangements equals Table 2: exactly 66 unambiguous and 2 ambiguous (ring chromosome vs tandem duplication; offset inversion vs inverted insertion-deletion)
- Every called breakpoint is supported by 2 or more primary/secondary junction positions clustering within 250 nucleotides
- Simple-rearrangement pathway used only for loci with 2 columns of split reads (3 for insertions); Complex Rearrangement window used for 3 or 4 columns
- Synthetic data verifiable as non-experimental: MakeSVGenome reads are fixed length (10 kb), evenly spaced (1 kb), alternate orientation, and carry no sequence variants besides the engineered rearrangements

## Limitations

- Ring chromosomes and tandem duplications are not readily distinguishable and produce ambiguous annotations
- Offset inversions and inverted insertion-deletions with loss at the insertion site are not readily distinguishable and produce ambiguous annotations
- MakeSVGenome synthetic data contains no other mutations or sequence variants, so real-data performance (with sequencing error and background variation) is not benchmarked by this exercise

## Evidence

- [results] AgileStructure annotated the synthetic data unambiguously for all but 2 of the 68 in silico rearrangements: [evidence span withheld — anchor status: not_found]
- [abstract] We developed AgileStructure, a desktop application for locating and annotating large-scale genomic rearrangements: "We developed AgileStructure, a desktop application for locating and annotating large‑scale genomic rearrangements using aligned long‑read data."
- [readme] MakeSVGenome creates chromosomes with rearrangements and synthetic long reads for testing SV detection: [evidence span withheld — anchor status: not_found]
