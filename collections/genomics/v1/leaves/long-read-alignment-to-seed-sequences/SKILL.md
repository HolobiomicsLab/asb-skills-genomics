---
name: long-read-alignment-to-seed-sequences
description: Use when you have long-read FASTA files and a uniform-length seed FASTA
  (unique numeric IDs, excluding 0) and need seed-to-read alignment files in SAM format
  to build a TULIP seed graph.
when_to_use_negative:
- Input reads are short Illumina reads rather than nanopore long reads — TULIP's seed
  graph relies on long reads co-aligning multiple seeds.
- Seeds vary in length or have non-unique/non-numeric identifiers — TULIP strictly
  requires uniform seed length and unique numeric IDs (excluding 0).
- You intend to use an aligner whose output is not SAM (e.g. DALIGNER); the TULIP
  prototype version only accepts SAM format.
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3188
tools:
- name: TULIP (tulipseed.perl / tulipbulb.perl)
  role: Downstream consumer of the SAM alignment files and the alignments.txt configuration
    to build the seed graph and bundle long reads into scaffolds
  repo: github.com/Generade-nl/TULIP
- name: BWA MEM
  role: Aligns each nanopore long-read FASTA against the indexed seed set; other aligners
    should work but only SAM output is accepted
provenance:
  source_task_ids:
  - task_004
  - task_008
  source_papers:
  - doi: 10.1038/s41598-017-07650-6
    title: Rapid de novo assembly of the European eel genome from nanopore sequencing
      reads
  source_content_hash: sha256:7e32c709e08eea6ee0a25f4617bdc0c2018e81a78fa9fe3431df9dc10cbe63cd
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - BWA MEM
  - TULIP
  - TULIP (tulipseed.perl / tulipbulb.perl)
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3188
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5333333333333333
n_claims: 15
---

# long-read-alignment-to-seed-sequences

## Summary

Aligns nanopore long reads (e.g. MinION R7.3, R9 1D/2D, R9.4) against a set of sparse, pre-selected short seed sequences with BWA MEM, retaining only the first six SAM fields up to the CIGAR string to serve as input for TULIP's seed graph construction.

## When to use

Use when you have long-read FASTA files and a uniform-length seed FASTA (unique numeric IDs, excluding 0) and need seed-to-read alignment files in SAM format to build a TULIP seed graph.

## When NOT to use

- Input reads are short Illumina reads rather than nanopore long reads — TULIP's seed graph relies on long reads co-aligning multiple seeds.
- Seeds vary in length or have non-unique/non-numeric identifiers — TULIP strictly requires uniform seed length and unique numeric IDs (excluding 0).
- You intend to use an aligner whose output is not SAM (e.g. DALIGNER); the TULIP prototype version only accepts SAM format.

## Inputs

- eel_seeds_285.fasta — pre-selected 'not too repetitive' seed sequences, all exactly the same length (285 bp), with unique numeric identifiers excluding 0
- Long-read FASTA files: R7.3.fasta, R9_pass_1d.fasta, R9_pass_2d.fasta, R9.4_pass_1d.fasta, one read per line, unique read identifiers per file

## Outputs

- R73_vs_285.shortsam, R91D_vs_285.shortsam, R92D_vs_285.shortsam, R941D_vs_285.shortsam — 6-field SAM alignment files (QNAME, FLAG, RNAME, POS, MAPQ, CIGAR)
- alignments.txt — configuration file listing type[tab]alignment[tab]original_fasta for each read set

## How to apply

Index the seed FASTA once with `bwa index -p 285_seeds eel_seeds_285.fasta`, then run BWA MEM separately for each nanopore read file, tuning -k and -W per chemistry (R7.3: -k 14 -W 45; R9_pass_1d: -k 16 -W 50; R9_pass_2d: -k 19 -W 60; R9.4_pass_1d: -k 16 -W 60) with common flags -t 4 -r 10 -A 1 -B 1 -O 1 -E 1 -L 0. Pipe each run through `cut -f 1,2,3,4,5,6` so only the first six SAM fields (up to and including the CIGAR string) are kept, writing .shortsam files such as R73_vs_285.shortsam. TULIP currently accepts SAM format only, so other aligners (e.g. DALIGNER) are not usable with the prototype version. Finally, list all alignment files in a configuration file with the format type[tab]alignment[tab]original_fasta, one line per read set (sam, .shortsam, original .fasta), which tulipseed.perl consumes. The rationale is that TULIP only needs read-seed co-alignment information (identifiers, flag, position, map quality, CIGAR), so trimming unused SAM columns reduces file size without losing graph-relevant evidence.

## Related tools

- **TULIP (tulipseed.perl / tulipbulb.perl)** (Downstream consumer of the SAM alignment files and the alignments.txt configuration to build the seed graph and bundle long reads into scaffolds) — github.com/Generade-nl/TULIP
- **BWA MEM** (Aligns each nanopore long-read FASTA against the indexed seed set; other aligners should work but only SAM output is accepted)

## Examples

```
bwa mem -t 4 -k 16 -W 50 -r 10 -A 1 -B 1 -O 1 -E 1 -L 0 285_seeds R9_pass_1d.fasta | cut -f 1,2,3,4,5,6 > R91D_vs_285.shortsam
```

## Evaluation signals

- Each .shortsam file contains exactly 6 tab-separated columns per alignment line, ending with the CIGAR string.
- Read identifiers are unique within each input FASTA file (concatenated data can break this).
- alignments.txt has three tab-separated fields per line: 'sam', the .shortsam path, and the corresponding original .fasta path.
- Alignment parameters differ per chemistry as specified (-k 14/16/19 and -W 45/50/60), with -A 1 -B 1 -O 1 -E 1 -L 0 applied uniformly.
- Seed identifiers in eel_seeds_285.fasta are unique nonzero numbers and all seeds are exactly the same length.

## Limitations

- The tulipbulb.perl prototype contains a bug preventing sequence addition in rare cases, inserting gaps of NNNs into scaffolds.
- Only the first 6 SAM fields are used; downstream graph evidence depends entirely on the sensitivity of BWA MEM parameters — overly strict settings miss co-alignments between seeds.
- Support for other alignment formats (e.g. DALIGNER output) was present in earlier TULIP versions and might reappear, but is absent in this prototype.

## Evidence

- [methods] We used BWA MEM to align reads to seeds, but other aligners should work. Currently, TULIP accepts SAM format: "We used BWA MEM to align reads to seeds, but other aligners should work. Currently, TULIP accepts SAM format"
- [readme] Only the first 6 fields of the SAM alignment information is used (up to and including the CIGAR string): "Only the first 6 fields of the SAM alignment information is used (up to and including the CIGAR string), so you might want to clip off the rest."
- [methods] bwa mem -t 4 -k 16 -W 50 -r 10 -A 1 -B 1 -O 1 -E 1 -L 0 285_seeds R9_pass_1d.fasta   | cut -f 1,2,3,4,5,6 > R91D_vs_285.shortsam: "bwa mem -t 4 -k 16 -W 50 -r 10 -A 1 -B 1 -O 1 -E 1 -L 0 285_seeds R9_pass_1d.fasta   | cut -f 1,2,3,4,5,6 > R91D_vs_285.shortsam"
- [readme] Strict requirements are that seed sequence identifiers are unique numbers (excluding 0), and that all seeds are of exactly the same length.: "Strict requirements are that seed sequence identifiers are unique numbers (excluding 0), and that all seeds are of exactly the same length."
- [methods] When using multiple alignment files, you should specify their locations in a short configuration file: "When using multiple alignment files, you should specify their locations in a short configuration file"
