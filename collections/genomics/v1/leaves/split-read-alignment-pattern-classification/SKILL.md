---
name: split-read-alignment-pattern-classification
description: Use when you have Minimap2-aligned long-read (PacBio/ONT) BAMs spanning
  putative rearrangements and need to label each event simple vs complex from the
  number of split-read alignment columns.
when_to_use_negative:
- The data are short-read sequencing data — AgileStructure and this split-read column
  logic target long-read (PacBio/ONT) alignments of large-scale rearrangements unresolved
  by short reads
- You need precise breakpoint coordinates or variant calling rather than a simple/complex
  structural classification of an already-suspected event
- The event falls in an inherently ambiguous scenario the paper reports (ring chromosome
  vs tandem duplication; offset inversion vs inverted insertion-deletion with deletion
  at the insertion site) where column counts cannot discriminate
edam_operation: http://edamontology.org/operation_2990
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: AgileStructure
  role: Reference implementation whose split-read alignment column patterns define
    the simple (2, or 3 for insertions) vs complex (3–4 columns) classification rule
  repo: https://github.com/msjimc/AgileStructure
- name: minimap2
  role: Long-read-specific aligner used with the '-ax map-ont' preset and '--secondary=no'
    to produce the split-read alignments fed to the classifier
  repo: https://github.com/lh3/minimap2
- name: MakeSVGenome
  role: Generates synthetic chromosomes with structural rearrangements and faux fixed-length,
    fixed-spacing long reads used as benchmark inputs for validating the classifier
  repo: https://github.com/msjimc/MakeSVGenome
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bioinformatics/btag294
    title: Utilization of long-read sequencing for the detection of structural rearrangements
      with AgileStructure
  source_content_hash: sha256:8877e170bf24e16daf2a72db00cd10b114db2ca43b6cd1084a77b0e808382e7b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/msjimc/AgileStructure
  tools:
  - AgileStructure
  - MakeSVGenome
  - minimap2
  edam_operation: http://edamontology.org/operation_2990
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# split-read-alignment-pattern-classification

## Summary

Classifies structural rearrangements as simple or complex by counting distinct columns (grouped start/stop positions) of split-read alignments in a BAM: 2 columns (3 for insertions) indicate simple, while 3 or 4 columns indicate complex. This mirrors the split-read pattern logic underlying AgileStructure's interpretation of long-read alignments.

## When to use

Use when you have Minimap2-aligned long-read (PacBio/ONT) BAMs spanning putative rearrangements and need to label each event simple vs complex from the number of split-read alignment columns.

## When NOT to use

- The data are short-read sequencing data — AgileStructure and this split-read column logic target long-read (PacBio/ONT) alignments of large-scale rearrangements unresolved by short reads
- You need precise breakpoint coordinates or variant calling rather than a simple/complex structural classification of an already-suspected event
- The event falls in an inherently ambiguous scenario the paper reports (ring chromosome vs tandem duplication; offset inversion vs inverted insertion-deletion with deletion at the insertion site) where column counts cannot discriminate

## Inputs

- BAM file of long reads (faux 10 kb reads at 1 kb intervals, alternating orientations) aligned to hg19/mm10 with Minimap2 '-ax map-ont --secondary=no'
- Sorted, indexed BAM (Samtools) from which split-read (primary/secondary) alignment junctions are extracted
- Guide-annotated expected class labels (simple vs complex) for benchmark rearrangements, e.g. from the AgileStructure guide

## Outputs

- Per-rearrangement label: simple (2 columns; 3 for insertions) or complex (3 or 4 columns)
- Per-example counts of distinct split-read alignment columns and breakpoint positions
- A results table, e.g. classification_results.csv, with one row per synthetic rearrangement plus mismatches vs expected class

## How to apply

Align the reads to a reference such as hg19/mm10 with Minimap2 using the '-ax map-ont' preset and '--secondary=no', then sort and index the BAMs with Samtools. Parse each indexed BAM to extract split-read alignments and cluster genomic positions of primary/secondary alignment junctions, calling a putative breakpoint when 2 or more junction positions fall within 250 nucleotides. Count the distinct columns (grouped start/stop positions) of split-read alignments per event and apply the decision rule: 2 columns (3 for insertions) means simple, whereas 3 or 4 columns means complex — AgileStructure expects complex rearrangements to present as 3 or 4 columns depending on whether one or both sides are complex. Validate by comparing predicted labels against guide-annotated expected classes across synthetic MakeSVGenome examples, recording per-example predictions, column counts, and mismatches.

## Related tools

- **AgileStructure** (Reference implementation whose split-read alignment column patterns define the simple (2, or 3 for insertions) vs complex (3–4 columns) classification rule) — https://github.com/msjimc/AgileStructure
- **minimap2** (Long-read-specific aligner used with the '-ax map-ont' preset and '--secondary=no' to produce the split-read alignments fed to the classifier) — https://github.com/lh3/minimap2
- **MakeSVGenome** (Generates synthetic chromosomes with structural rearrangements and faux fixed-length, fixed-spacing long reads used as benchmark inputs for validating the classifier) — https://github.com/msjimc/MakeSVGenome

## Examples

```
minimap2 -ax map-ont --secondary=no hg19.fa reads.fq | samtools sort -o aln.bam && samtools index aln.bam
```

## Evaluation signals

- Predicted labels match the AgileStructure guide's expected classes across all 68 MakeSVGenome examples (32 simple and 36 complex insertion-deletion rearrangements)
- Simple events show exactly 2 distinct split-read alignment columns (3 for insertions); complex events show 3 or 4 columns
- Breakpoint calls require 2 or more junction positions clustering within 250 nucleotides — isolated single junctions must not be called
- Disagreement is permitted only for the two inherently ambiguous scenarios reported in the paper (ring chromosome vs tandem duplication; offset inversion vs inverted insertion-deletion with deletion at the insertion site)
- Synthetic inputs are recognisable as test data: reads are of fixed length and spacing with alternating orientations and no other mutations or sequence variants added

## Limitations

- Ring chromosomes and tandem duplications are inherently ambiguous — their split-read patterns can coincide
- Offset inversions and inverted insertion-deletions with a deletion at the insertion site are inherently ambiguous by this column-count rule
- Complex events present as 3 or 4 columns depending on whether one or both sides of the rearrangement are complex, so column count alone does not localise complexity

## Evidence

- [abstract] AgileStructure is a desktop application for locating and annotating large-scale genomic rearrangements using aligned long-read data: "We developed AgileStructure, a desktop application for locating and annotating large‑scale genomic rearrangements using aligned long‑read data."
- [intro] Long-read data aligned with minimap2 before structural interpretation: "the data is typically aligned to the appropriate genome using a long-read-specific aligner such as minimap2"
- [readme] MakeSVGenome synthetic read design enables unambiguous benchmark inputs: [evidence span withheld — anchor status: external_doc]
- [abstract] Split-read-spanning reads support interpretation of complex events: "The software enables user‑guided exploration of breakpoint‑spanning reads, supporting accurate interpretation of complex events"
