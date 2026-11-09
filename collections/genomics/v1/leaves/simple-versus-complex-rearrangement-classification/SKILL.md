---
name: simple-versus-complex-rearrangement-classification
description: Use when Minimap2-aligned long-read BAM data show clustered junction/split-read
  positions at a candidate locus and each event must be annotated as a simple or complex
  structural rearrangement with recorded ambiguity status.
when_to_use_negative:
- When the goal is fully automated high-throughput SV calling without user-guided
  inspection — AgileStructure is explicitly a desktop application for user-guided
  exploration of breakpoint-spanning reads
- When reads have not been aligned with a long-read-specific aligner or the BAM lacks
  soft-clip/junction information needed for the two-panel interface
- When synthetic reads are intended for anything other than algorithm testing — MakeSVGenome
  data contain no mutations other than the engineered structural variants
edam_operation: http://edamontology.org/operation_3196
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: AgileStructure
  role: Desktop application providing the two-panel interface, breakpoint calling
    (≥2 junction positions within 250 nt), simple-rearrangement pathway and Complex
    Rearrangement window for annotation
  repo: https://github.com/msjimc/AgileStructure
- name: MakeSVGenome
  role: Generates synthetic chromosomes and faux 10 kb reads with alternating orientations
    containing known simple and complex rearrangements for validating annotation accuracy
  repo: msjimc/MakeSVGenome
- name: minimap2
  role: Long-read-specific aligner producing the aligned BAM input with primary/secondary
    junction and soft-clip data used for breakpoint calling
  repo: https://github.com/lh3/minimap2
- name: Samtools
  role: Converts, sorts, and indexes the minimap2 SAM output to BAM for loading into
    AgileStructure
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioinformatics/btag294
    title: Utilization of long-read sequencing for the detection of structural rearrangements
      with AgileStructure
  source_content_hash: sha256:ce4dfd3f2522c3d79019b6e5c6989e58d5656262a32b4b07dedccc8ccf9a3489
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/msjimc/AgileStructure
  tools:
  - AgileStructure
  - MakeSVGenome
  - Samtools
  - minimap2
  edam_operation: http://edamontology.org/operation_3196
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# simple-versus-complex-rearrangement-classification

## Summary

A user-guided annotation workflow in AgileStructure that classifies aligned long-read breakpoint evidence into simple rearrangements (deletions, insertions, inversions, duplications, translocations, ring chromosomes) versus complex insertion-deletion rearrangements, based on the number of split-read columns at each breakpoint.

## When to use

Use when Minimap2-aligned long-read BAM data show clustered junction/split-read positions at a candidate locus and each event must be annotated as a simple or complex structural rearrangement with recorded ambiguity status.

## When NOT to use

- When the goal is fully automated high-throughput SV calling without user-guided inspection — AgileStructure is explicitly a desktop application for user-guided exploration of breakpoint-spanning reads
- When reads have not been aligned with a long-read-specific aligner or the BAM lacks soft-clip/junction information needed for the two-panel interface
- When synthetic reads are intended for anything other than algorithm testing — MakeSVGenome data contain no mutations other than the engineered structural variants

## Inputs

- Sorted, indexed BAM file of long reads aligned to hg19/mm10 (aligned with minimap2 -ax map-ont --secondary=no)
- Candidate rearrangement loci on chromosomes of interest (e.g. human chromosomes 7 and 8)
- Optionally, MakeSVGenome-generated in silico rearrangement definitions (32 simple and 36 complex insertion-deletion events) as ground truth

## Outputs

- Per-event structural rearrangement annotations (deletion, insertion, inversion, duplication, translocation, ring chromosome, or complex insertion-deletion class)
- Breakpoint positions derived from clustered split-read junctions
- Ambiguity flags identifying unresolvable pairs (ring chromosome vs tandem duplication; offset inversion vs inverted insertion-deletion)

## How to apply

Load the indexed BAM into AgileStructure and screen candidate loci in blocks of up to 10 Mb; call breakpoints when 2 or more primary/secondary junction positions cluster within 250 nucleotides. Select the split reads at each breakpoint, then choose the pathway by counting split-read columns: use the simple-rearrangement pathway (Variant determination > Use soft clip data menus) when 2 columns of split reads are present (3 columns for insertions), and the Complex Rearrangement window when 3 or 4 columns are present. Record each annotation and flag events that resolve into the two documented ambiguous pairs — ring chromosome versus tandem duplication, and offset inversion versus inverted insertion-deletion with loss at the insertion site.

## Related tools

- **AgileStructure** (Desktop application providing the two-panel interface, breakpoint calling (≥2 junction positions within 250 nt), simple-rearrangement pathway and Complex Rearrangement window for annotation) — https://github.com/msjimc/AgileStructure
- **MakeSVGenome** (Generates synthetic chromosomes and faux 10 kb reads with alternating orientations containing known simple and complex rearrangements for validating annotation accuracy) — msjimc/MakeSVGenome
- **minimap2** (Long-read-specific aligner producing the aligned BAM input with primary/secondary junction and soft-clip data used for breakpoint calling) — https://github.com/lh3/minimap2
- **Samtools** (Converts, sorts, and indexes the minimap2 SAM output to BAM for loading into AgileStructure)

## Examples

```
minimap2 -ax map-ont --secondary=no hg19.fa synthetic_reads.fq > aln.sam; samtools sort -o aln.bam aln.sam; samtools index aln.bam  # then inspect aln.bam in AgileStructure
```

## Evaluation signals

- Reproduced annotation of the 68 MakeSVGenome in silico rearrangements yields exactly 66 of 68 unambiguously annotated, matching the paper's Table 2 outcome
- The only unresolved cases are the two documented ambiguous pairs: ring chromosome vs tandem duplication, and offset inversion vs inverted insertion-deletion with loss at the insertion site
- Breakpoints are only called where ≥2 primary/secondary junction positions cluster within 250 nucleotides
- Simple-rearrangement pathway is applied only to events showing 2 split-read columns (3 for insertions); events with 3 or 4 columns are routed to the Complex Rearrangement window
- BAM input was produced with minimap2 -ax map-ont and --secondary=no, then sorted/indexed so AgileStructure can read junction and soft-clip data

## Limitations

- Ring chromosomes cannot be readily distinguished from tandem duplications
- Offset inversions cannot be readily distinguished from inverted insertion-deletions with loss at the insertion site
- Annotation is user-guided rather than automated, so throughput is limited by manual locus-by-locus screening in blocks of up to 10 Mb

## Evidence

- [results] AgileStructure annotated the synthetic data unambiguously for all but 2 of the 68 in silico rearrangements: only ring chromosomes vs tandem duplications and offset inversions vs inverted insertion-deletions were not readily distinguishable.: [evidence span withheld — anchor status: not_found]
- [methods] which are called when 2 or more primary/secondary junction positions cluster within 250 nucleotides: [evidence span withheld — anchor status: not_found]
- [methods] apply the simple-rearrangement pathway (Variant determination > Use soft clip data menus) for events showing 2 columns of split reads (3 for insertions) and the Complex Rearrangement window for events showing 3 or 4 columns: [evidence span withheld — anchor status: not_found]
- [abstract] We developed AgileStructure, a desktop application for locating and annotating large-scale genomic rearrangements using aligned long-read data.: [evidence span withheld — anchor status: not_found]
- [readme] The read data produced, consists of reads of fixed length and spacing, that alternate between forward and reverse orientations.: [evidence span withheld — anchor status: external_doc]
