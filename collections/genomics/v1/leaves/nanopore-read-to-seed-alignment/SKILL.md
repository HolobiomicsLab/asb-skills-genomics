---
name: nanopore-read-to-seed-alignment
description: Use when you have FASTA-formatted nanopore reads (e.g. R7.3, R9 1D/2D,
  R9.4 1D MinION runs) plus a set of pre-selected 'not too repetitive' seed sequences
  of uniform length, and need read-to-seed alignments to drive a TULIP seed-layout
  assembly.
when_to_use_negative:
- If you are polishing or aligning reads to an existing contig set (e.g. Canu contigs
  or Illumina data for Pilon), not to sparse 285 bp seed islands — that is a different
  mapping task with different parameters.
- If your seed sequences violate TULIP requirements (repetitive seeds, non-numeric
  or duplicated identifiers including 0, unequal lengths) — the graph untangling becomes
  prohibitively slow or fails.
- If your aligner output is not SAM (e.g. DALIGNER output) — TULIP version 0.4 accepts
  only SAM format.
edam_operation: http://edamontology.org/operation_3198
edam_topics:
- http://edamontology.org/topic_0102
tools:
- name: BWA MEM
  role: Aligns each nanopore read file against the 285 bp seed index with chemistry-specific
    -k/-W seeding parameters; output is clipped to the first six SAM fields for TULIP.
  repo: http://bio-bwa.sourceforge.net
- name: TULIP (tulipseed.perl)
  role: Consumes the per-chemistry .shortsam files via alignments.txt (--config) to
    build the seed graph and ordered scaffolds.
  repo: github.com/Generade-nl/TULIP
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41598-017-07650-6
    title: Rapid de novo assembly of the European eel genome from nanopore sequencing
      reads
  source_content_hash: sha256:777c5a34d8626f027d0f4b16c4a64d85990c43c025ee4a8d1aff63534803eaeb
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - BWA MEM
  - TULIP
  - TULIP (tulipseed.perl)
  - tulipbulb.perl
  edam_operation: http://edamontology.org/operation_3198
  edam_topics:
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# nanopore-read-to-seed-alignment

## Summary

Aligns Oxford Nanopore long reads to sparse 285 bp seed sequences with BWA MEM using per-chemistry parameters, then condenses the SAM output to its first six fields so TULIP can build a seed graph. This alignment step is the input layer that converts raw MinION reads into seed-link evidence for the eel genome assembly.

## When to use

Use when you have FASTA-formatted nanopore reads (e.g. R7.3, R9 1D/2D, R9.4 1D MinION runs) plus a set of pre-selected 'not too repetitive' seed sequences of uniform length, and need read-to-seed alignments to drive a TULIP seed-layout assembly.

## When NOT to use

- If you are polishing or aligning reads to an existing contig set (e.g. Canu contigs or Illumina data for Pilon), not to sparse 285 bp seed islands — that is a different mapping task with different parameters.
- If your seed sequences violate TULIP requirements (repetitive seeds, non-numeric or duplicated identifiers including 0, unequal lengths) — the graph untangling becomes prohibitively slow or fails.
- If your aligner output is not SAM (e.g. DALIGNER output) — TULIP version 0.4 accepts only SAM format.

## Inputs

- eel_seeds_285.fasta — pre-selected, 'not too repetitive' 285 bp seed sequences with unique numeric IDs (excluding 0), all exactly the same length
- Long-read FASTA files from four MinION chemistries: R7.3.fasta, R9_pass_1d.fasta, R9_pass_2d.fasta, R9.4_pass_1d.fasta (one line per read, unique identifiers per file)
- BWA index of the seed FASTA (built with 'bwa index -p 285_seeds eel_seeds_285.fasta')

## Outputs

- Per-chemistry 6-field SAM alignment files, e.g. R73_vs_285.shortsam, R91D_vs_285.shortsam, R92D_vs_285.shortsam, R941D_vs_285.shortsam
- alignments.txt configuration file mapping each .shortsam to its original FASTA ('sam<TAB>alignment<TAB>original fasta')
- Downstream TULIP inputs consumed by tulipseed.perl to produce tulip/eel.graph and tulip/eel.scaffolds

## How to apply

Index the seed FASTA (e.g. eel_seeds_285.fasta) with bwa index, then run bwa mem once per nanopore read file with chemistry-specific seeding parameters: -k 14 -W 45 for R7.3, -k 16 -W 50 for R9 1D, -k 19 -W 60 for R9 2D, and -k 16 -W 60 for R9.4 1D, all with -r 10 -A 1 -B 1 -O 1 -E 1 -L 0 and -t 4. Pipe each alignment through 'cut -f 1,2,3,4,5,6' because TULIP only consumes the first six SAM fields up to the CIGAR string. Record each resulting .shortsam file in a configuration file in 'format [tab] alignment [tab] original fasta' form (e.g. sam<TAB>R92D_vs_285.shortsam<TAB>R9_pass_2d.fasta) so tulipseed.perl can locate both alignments and the source long reads. Seeds must have unique numeric identifiers (excluding 0) and be exactly the same length (285 bp here), and read identifiers must be unique within each FASTA to avoid collisions when multiple flow-cell files are used.

## Related tools

- **BWA MEM** (Aligns each nanopore read file against the 285 bp seed index with chemistry-specific -k/-W seeding parameters; output is clipped to the first six SAM fields for TULIP.) — http://bio-bwa.sourceforge.net
- **TULIP (tulipseed.perl)** (Consumes the per-chemistry .shortsam files via alignments.txt (--config) to build the seed graph and ordered scaffolds.) — github.com/Generade-nl/TULIP

## Examples

```
bwa mem -t 4 -k 16 -W 60 -r 10 -A 1 -B 1 -O 1 -E 1 -L 0 285_seeds R9.4_pass_1d.fasta | cut -f 1,2,3,4,5,6 > R941D_vs_285.shortsam
```

## Evaluation signals

- All four chemistry-specific bwa mem commands completed and each produced a .shortsam containing only six SAM fields (ending at the CIGAR string).
- alignments.txt lists one line per alignment file in 'format [tab] alignment [tab] original fasta' form and every referenced .shortsam and FASTA exists.
- Seed FASTA passes TULIP preconditions: unique numeric seed identifiers (excluding 0) and all seeds exactly 285 bp.
- When fed to tulipseed.perl --seedlength 285 --diploid, the resulting .graph file shows seed links with evidence counts averaging ~7.4 long-read alignments per link, matching the published candidate assembly.
- Downstream TULIP run reproduces the reported 2366 scaffolds (431 bp–8.7 Mbp) with 232 scaffolds >1 Mbp comprising 56% of assembly length.

## Limitations

- TULIP v0.4 accepts only SAM format; support for other formats such as DALIGNER output was present in earlier versions and might reappear.
- Read identifiers must be unique in each file, so concatenating flow-cell FASTA files without renaming can cause collisions.
- The TULIP prototype may emit NNN gaps because tulipbulb.perl has a bug that occasionally prevents sequence addition.
- With repetitive seeds, untangling the graph takes much longer, so seed selection ('not too repetitive') directly affects runtime and success.

## Evidence

- [methods] We used BWA MEM to align reads to seeds, but other aligners should work.: "We used BWA MEM to align reads to seeds, but other aligners should work."
- [readme] bwa mem -t 4 -k 14 -W 45 -r 10 -A 1 -B 1 -O 1 -E 1 -L 0 285_seeds R7.3.fasta: "bwa mem -t 4 -k 14 -W 45 -r 10 -A 1 -B 1 -O 1 -E 1 -L 0 285_seeds R7.3.fasta"
- [readme] Only the first 6 fields of the SAM alignment information is used (up to and including the CIGAR string), so you might want to clip off the rest.: "Only the first 6 fields of the SAM alignment information is used (up to and including the CIGAR string), so you might want to clip off the rest."
- [readme] Strict requirements are that seed sequence identifiers are unique numbers (excluding 0), and that all seeds are of exactly the same length.: "Strict requirements are that seed sequence identifiers are unique numbers (excluding 0), and that all seeds are of exactly the same length."
- [readme] Tulipseed takes as input alignments files of long reads to sparse short seeds, and outputs a graph and scaffold structures.: "Tulipseed takes as input alignments files of long reads to sparse short seeds, and outputs a graph and scaffold structures."
