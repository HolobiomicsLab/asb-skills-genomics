---
name: structural-variant-breakpoint-detection
description: Use when an indexed long-read BAM (ONT reads aligned with minimap2 '-ax
  map-ont --secondary=no') must be screened for structural rearrangement breakpoints
  and classified as simple (2 columns of split reads; 3 for insertions) or complex
  (3 or 4 columns).
when_to_use_negative:
- Input reads are short-read (Illumina) alignments, which are stated to fail to resolve
  the underlying pathogenic variant in approximately 50% of rare-disease patients
- The BAM lacks linked secondary alignments for split reads (e.g. aligned without
  split-read or primary/secondary linking information), so primary/secondary junction
  positions cannot be extracted
- The goal is small variant calling (SNVs or small indels) rather than large-scale
  structural rearrangement detection
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_3168
tools:
- name: AgileStructure
  role: Desktop application that clusters primary/secondary split-read junction positions
    within 250 nucleotides to call breakpoints and annotates events as simple or complex
    in a two-panel user-guided interface
  repo: https://github.com/msjimc/AgileStructure
- name: minimap2
  role: Long-read-specific aligner producing the split-read BAM input (presets -ax
    map-ont with --secondary=no)
  repo: https://github.com/lh3/minimap2
- name: Samtools
  role: Converts, sorts, and indexes the minimap2 SAM output into an indexed BAM for
    AgileStructure
- name: MakeSVGenome
  role: Generates synthetic chromosomes and faux 10 kb long-read data used to benchmark
    breakpoint calling and simple/complex classification accuracy
  repo: msjimc/MakeSVGenome
provenance:
  source_task_ids:
  - task_001
  - task_006
  - task_002
  source_papers:
  - doi: 10.1093/bioinformatics/btag294
    title: Utilization of long-read sequencing for the detection of structural rearrangements
      with AgileStructure
  source_content_hash: sha256:5462a62df8365ccf3825c0462c1a7e57a19b3a1689169059b56870db03c3fa70
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
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# structural-variant-breakpoint-detection

## Summary

Detects and annotates large-scale genomic rearrangements (deletions, insertions, inversions, duplications, translocations, ring chromosomes, complex insertion-deletion events) from aligned long-read BAM data using AgileStructure's split-read junction clustering and two-panel user-guided annotation interface.

## When to use

Use when an indexed long-read BAM (ONT reads aligned with minimap2 '-ax map-ont --secondary=no') must be screened for structural rearrangement breakpoints and classified as simple (2 columns of split reads; 3 for insertions) or complex (3 or 4 columns).

## When NOT to use

- Input reads are short-read (Illumina) alignments, which are stated to fail to resolve the underlying pathogenic variant in approximately 50% of rare-disease patients
- The BAM lacks linked secondary alignments for split reads (e.g. aligned without split-read or primary/secondary linking information), so primary/secondary junction positions cannot be extracted
- The goal is small variant calling (SNVs or small indels) rather than large-scale structural rearrangement detection

## Inputs

- Indexed long-read BAM (ONT reads aligned with minimap2 -ax map-ont --secondary=no, sorted and indexed with Samtools)
- Reference genome (hg19/mm10)
- Optional: MakeSVGenome synthetic test chromosomes with faux 10 kb reads at 1 kb intervals

## Outputs

- breakpoints.tsv with chromosome, position, and supporting read count for each called breakpoint
- classification_results.csv with per-rearrangement column counts and predicted simple/complex labels
- Annotations of each rearrangement as simple or complex in AgileStructure's two-panel interface

## How to apply

Align ONT long reads to hg19/mm10 with minimap2 using the '-ax map-ont' preset and '--secondary=no', then sort and index the BAM with Samtools. Load the indexed BAM into AgileStructure and screen candidate loci in blocks of up to 10 Mb via the two-panel interface; the tool retains genomic positions of junctions between primary and secondary alignments of split reads and calls a putative breakpoint when 2 or more junction positions fall within 250 nucleotides of each other. At each breakpoint, count the distinct columns (grouped start/stop positions) of split-read alignments: 2 columns (3 for insertions) indicates a simple rearrangement to be annotated via the soft-clip menus, while 3 or 4 columns indicates a complex rearrangement to be handled in the Complex Rearrangement window. Record chromosome, position, and supporting read count for each called breakpoint.

## Related tools

- **AgileStructure** (Desktop application that clusters primary/secondary split-read junction positions within 250 nucleotides to call breakpoints and annotates events as simple or complex in a two-panel user-guided interface) — https://github.com/msjimc/AgileStructure
- **minimap2** (Long-read-specific aligner producing the split-read BAM input (presets -ax map-ont with --secondary=no)) — https://github.com/lh3/minimap2
- **Samtools** (Converts, sorts, and indexes the minimap2 SAM output into an indexed BAM for AgileStructure)
- **MakeSVGenome** (Generates synthetic chromosomes and faux 10 kb long-read data used to benchmark breakpoint calling and simple/complex classification accuracy) — msjimc/MakeSVGenome

## Examples

```
minimap2 -ax map-ont --secondary=no hg19.fa synthetic_reads.fq > aln.sam && samtools sort -o aln.bam aln.sam && samtools index aln.bam
```

## Evaluation signals

- breakpoints.tsv exists, is a well-formed TSV with coordinate and support-count fields, and every reported breakpoint is supported by at least 2 junction positions within a 250-nucleotide window
- On the 68 MakeSVGenome synthetic rearrangements (32 simple, 36 complex), predicted labels match the guide's expected classes with at most the two documented ambiguous disagreements
- Column-count rule applied correctly: 2 columns (3 for insertions) labeled simple, 3 or 4 columns labeled complex
- Exactly 66 of 68 in silico rearrangements annotated unambiguously, matching the paper's Table 2 outcome
- BAM inputs verified as indexed and produced with the -ax map-ont and --secondary=no parameters so primary/secondary links are preserved

## Limitations

- Two rearrangement scenarios are inherently ambiguous and cannot be readily distinguished: ring chromosome vs tandem duplication, and offset inversion vs inverted insertion-deletion with loss at the insertion site
- AgileStructure is a Windows application; on Linux, BSD or macOS it must be run via WINE on Intel or AMD CPUs
- Complex rearrangements are expected to present as either 3 or 4 columns of split-read alignments depending on whether one or both sides of the rearrangement are complex; events outside this pattern fall outside the validated classification path

## Evidence

- [abstract] AgileStructure developed for long-read SV detection: [evidence span withheld — anchor status: not_found]
- [intro] Alignment prerequisite with long-read-specific aligner: "the data is typically aligned to the appropriate genome using a long-read-specific aligner such as minimap2"
- [full_text] Breakpoint criterion: 2+ junction positions within 250 nt: [evidence span withheld — anchor status: not_found]
- [readme] Synthetic benchmark read design: [evidence span withheld — anchor status: external_doc]
- [other] Availability of example aligned read data: "Source code, binaries, user guide, and example aligned read data, are available on GitHub: https://github.com/msjimc/AgileStructure."
