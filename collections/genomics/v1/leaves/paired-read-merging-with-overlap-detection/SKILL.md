---
name: paired-read-merging-with-overlap-detection
description: Use when paired-end Illumina read pairs overlap and must be converted
  into single merged fragments before downstream k-mer counting or seed construction,
  e.g. to derive 'not too repetitive' seeds for TULIP.
when_to_use_negative:
- Mate pairs do not overlap (no shared sequence between R1 and R2), so merging cannot
  join them
- The goal is direct de novo assembly of the read pairs rather than constructing short
  seeds for a long-read scaffolding pipeline like TULIP
- Input reads are already single-end or already merged; re-merging adds no information
edam_topics:
- http://edamontology.org/topic_0102
- http://edamontology.org/topic_0622
tools:
- name: Sickle
  role: Sliding-window quality and length trimming of the FASTQ read pairs before
    merging
  repo: https://github.com/najoshi/sickle
- name: FLASh
  role: Merges overlapping paired-end reads into single fragments, requiring a minimum
    overlap of 15 nt between mate pairs
- name: Jellyfish
  role: Counts 25-mers in merged fragments to identify fragments containing 25-mers
    occurring more than 25 times for filtering
- name: TULIP
  role: Downstream consumer of the merged and filtered seed sequences (eel_seeds FASTA
    files)
  repo: https://github.com/Generade-nl/TULIP
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41598-017-07650-6
    title: Rapid de novo assembly of the European eel genome from nanopore sequencing
      reads
  source_content_hash: sha256:89bf0f8e75b8415d5a0996e5eaa61dbcb524323ae1d3cfa91a9dc0dc3a7dd3db
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/najoshi/sickle
  tools:
  - FLASh
  - Jellyfish
  - Sickle
  - TULIP
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0102
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# Paired-read merging with overlap detection

## Summary

Merges overlapping paired-end Illumina read mates into single contiguous fragments using FLASh with a minimum mate overlap of 15 nt, producing merged fragments that serve as raw material for k-mer-counting seed selection in the TULIP assembly of the European eel genome.

## When to use

Use when paired-end Illumina read pairs overlap and must be converted into single merged fragments before downstream k-mer counting or seed construction, e.g. to derive 'not too repetitive' seeds for TULIP.

## When NOT to use

- Mate pairs do not overlap (no shared sequence between R1 and R2), so merging cannot join them
- The goal is direct de novo assembly of the read pairs rather than constructing short seeds for a long-read scaffolding pipeline like TULIP
- Input reads are already single-end or already merged; re-merging adds no information

## Inputs

- Paired-end Illumina read pairs in FASTQ format (R1/R2 files, optionally quality-trimmed with Sickle)
- Minimum mate overlap parameter (15 nt used for the eel genome)

## Outputs

- Merged single-end fragments in FASTA/FASTQ combining the overlapping mates
- Downstream seed files, e.g. 5,019,778 sequences of exactly 270 bp and 873,058 sequences of exactly 285 bp, with unique numeric sequence identifiers (excluding 0)

## How to apply

First quality-trim the Illumina paired-end FASTQ files with Sickle before merging. Then merge the trimmed mate pairs with FLASh, requiring a minimum overlap of 15 nt between mate pairs so overlapping read pairs are stitched into single fragments. Count 25-mers in the merged fragments with Jellyfish, then discard any fragment containing a 25-mer occurring more than 25 times, implementing the 'not too repetitive' seed-selection criterion. Size-select the remaining fragments into uniform-length sets — 5,019,778 seeds of exactly 270 bp (1.58x coverage of the ~860 Mbp eel genome) and 873,058 seeds of exactly 285 bp (0.29x coverage) — writing each set to FASTA with unique numeric identifiers (excluding 0). Uniform seed length and unique identifiers are strict TULIP input requirements.

## Related tools

- **Sickle** (Sliding-window quality and length trimming of the FASTQ read pairs before merging) — https://github.com/najoshi/sickle
- **FLASh** (Merges overlapping paired-end reads into single fragments, requiring a minimum overlap of 15 nt between mate pairs)
- **Jellyfish** (Counts 25-mers in merged fragments to identify fragments containing 25-mers occurring more than 25 times for filtering)
- **TULIP** (Downstream consumer of the merged and filtered seed sequences (eel_seeds FASTA files)) — https://github.com/Generade-nl/TULIP

## Examples

```
flash --min-overlap 15 reads_R1.fastq reads_R2.fastq -o merged_eel
```

## Evaluation signals

- Seed files contain exactly 5,019,778 sequences of 270 bp and 873,058 sequences of 285 bp
- Coverage of the ~860 Mbp eel genome computes to 1.58x (270 bp set) and 0.29x (285 bp set)
- All sequence identifiers within a seed FASTA are unique numeric values excluding 0
- All seeds within a file are of exactly the same length, as required by tulipseed.perl
- No retained fragment contains a 25-mer occurring more than 25 times

## Limitations

- Large repetitive regions and (sub)telomeric repeats may not contain frequent unique 285 bp islands and can be absent from the resulting assembly
- Seeds containing repetitive sequences would make it much longer to untangle the TULIP graph and to optimize seed numbers, motivating the strict 'not too repetitive' filter
- Only merged fragments of exactly uniform length (270 bp or 285 bp) satisfy TULIP seed input requirements; variable-length output must be re-selected

## Evidence

- [methods] Merge paired-end Illumina reads with FLASh, requiring a minimum overlap of 15 nt between mate pairs.: [evidence span withheld — anchor status: not_found]
- [results] the seed-selection step yielded 5019778 fragments of 270 bp or 873058 fragments of 285 bp: [evidence span withheld — anchor status: not_found]
- [readme] These are pre-selected seed sequenced, with selection criteria 'not too repetitive'.: "These are pre-selected seed sequenced, with selection criteria 'not too repetitive'. Seeds containing repetitive sequences should be fine, but it will then take much longer to untangle the graph, and"
- [readme] Strict requirements are that seed sequence identifiers are unique numbers (excluding 0), and that all seeds are of exactly the same length.: "Strict requirements are that seed sequence identifiers are unique numbers (excluding 0), and that all seeds are of exactly the same length."
