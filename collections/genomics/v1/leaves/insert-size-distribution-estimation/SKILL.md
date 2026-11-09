---
name: insert-size-distribution-estimation
description: Use when Fast-SG synthetic mate-pair libraries (e.g. 200 bp pairs at
  1000/2000/3000/5000 bp insert sizes) have been extracted from long reads and per-library
  scaffold-linking parameters are needed before scaffolding.
when_to_use_negative:
- The linking data are real long-range mate-pair libraries (e.g. 10X Genomics or DOVETAIL
  Genomics) rather than Fast-SG synthetic libraries — insert sizes span a range and
  orientation may be unknown, so the fixed-D estimation scheme does not apply
- Synthetic pairs have not yet been extracted — the estimation step requires aligned
  within-contig synthetic pairs as input
edam_topics:
- http://edamontology.org/topic_0196
- http://edamontology.org/topic_0102
tools:
- name: Fast-SG
  role: Extracts synthetic forward/reverse 200 bp mate pairs at user-specified insert
    sizes via a 100 bp moving window; the aligned within-contig pairs provide the
    insert-size distribution summarized here
  repo: https://github.com/adigenova/fast-sg
- name: KMC
  role: k-mer counter bundled with Fast-SG and used by its alignment-free scaffolding-graph
    pipeline over which the synthetic libraries are evaluated
  repo: https://github.com/refresh-bio/KMC
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/gigascience/giy048
    title: 'Fast-SG: an alignment-free algorithm for hybrid assembly'
  source_content_hash: sha256:12cfbe06d69e074c588c64a2df268db8e5635629708199e9655612b7d5c236ad
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/adigenova/fast-sg
  tools:
  - C++
  - Fast-SG
  - KMC
  - PERL
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0196
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# insert-size-distribution-estimation

## Summary

Estimates per-library insert-size mean, standard deviation, and read-pair orientation (FR/RF) by aligning Fast-SG synthetic mate-pair reads back to the contig set and summarizing the within-contig insert-size distribution.

## When to use

Use when Fast-SG synthetic mate-pair libraries (e.g. 200 bp pairs at 1000/2000/3000/5000 bp insert sizes) have been extracted from long reads and per-library scaffold-linking parameters are needed before scaffolding.

## When NOT to use

- The linking data are real long-range mate-pair libraries (e.g. 10X Genomics or DOVETAIL Genomics) rather than Fast-SG synthetic libraries — insert sizes span a range and orientation may be unknown, so the fixed-D estimation scheme does not apply
- Synthetic pairs have not yet been extracted — the estimation step requires aligned within-contig synthetic pairs as input

## Inputs

- Read configuration file (space-separated: type, library ID, path, insert sizes, pairing flag; e.g. examples/ecoli-reads.txt) listing long-read libraries and target insert sizes
- FASTA/FASTQ long-read libraries (PacBio or Oxford Nanopore, corrected or uncorrected)
- Contig set (FASTA) against which synthetic pairs are aligned

## Outputs

- library_stats.tsv: per-library table with library ID, estimated insert size, standard deviation, and orientation (FR/RF)
- FASTQ files of synthetic forward/reverse 200 bp read pairs per library

## How to apply

Parse the space-separated read configuration file (type, library ID, path, insert sizes, pairing flag) to obtain each long-read library and its target insert sizes. For each library, Fast-SG slides a 100 bp moving window along each long read and, at each specified insert size D, excises a forward 200 bp read from one end and a reverse 200 bp read from the opposite end, emitting FASTQ synthetic pairs restricted to the actual long-read length. Align the synthetic pairs back to the contig set and retain only pairs mapping within the same contig. For each library, trim the lowest and highest 10% of the insert-size distribution as outliers, then compute the mean insert size and standard deviation from the remaining within-contig aligned pairs; infer FR/RF orientation from the mapping coordinates. Rationale: synthetic libraries give a known insert size per pair, making gap-size estimation more tractable than for 10X Genomics or DOVETAIL Genomics data where insert sizes vary and orientation may be unknown.

## Related tools

- **Fast-SG** (Extracts synthetic forward/reverse 200 bp mate pairs at user-specified insert sizes via a 100 bp moving window; the aligned within-contig pairs provide the insert-size distribution summarized here) — https://github.com/adigenova/fast-sg
- **KMC** (k-mer counter bundled with Fast-SG and used by its alignment-free scaffolding-graph pipeline over which the synthetic libraries are evaluated) — https://github.com/refresh-bio/KMC

## Examples

```
./FAST-SG.pl  -k 15 -l example/ecoli-reads.txt -r example/ecoli-illumina.fa.gz -p test
```

## Evaluation signals

- library_stats.tsv exists and contains exactly one row per library configured in the read configuration file
- Each estimated insert size is consistent with the user-specified insert size (e.g. 1000, 2000, 3000, 5000 bp) after 10% low/high outlier trimming
- Standard deviation and orientation (FR/RF) fields are populated for every library row
- Insert sizes never exceed the actual long-read length; pairs spanning more than one contig are excluded from the statistics
- Orientation inferred from mapping coordinates matches the forward/reverse (FR) design of the synthetic pairs

## Limitations

- The insert size is restricted to the actual long-read size, so reads shorter than the target insert size yield truncated or absent pairs
- Only pairs mapping within the same contig contribute to the statistics, so highly fragmented assemblies reduce the usable pair count
- Estimation assumes the synthetic-library regime; gap-size estimation with real long-range mate-pair technologies (10X Genomics, DOVETAIL Genomics) is more challenging because linking information comes from a range of insert sizes and orientation may be unknown
- No version number is stated for Fast-SG itself (only RRID SCR_015934)

## Evidence

- [discussion] Insert size restricted to long-read length: "The insert size is restricted to the actual long-read size."
- [discussion] Synthetic libraries ease gap-size estimation versus 10X/DOVETAIL: "An estimation of the gap size with the existing long-range mate-pair technologies (10X genomics and DOVETAIL genomics) is more challenging than with the synthetic libraries"
- [methods] Read configuration file example location: "Example of a read configuration file can be found in examples/ecoli-reads.txt"
- [readme] Fast-SG purpose (README): "Fast-SG is an alignment-free algorithm for ultrafast scaffolding graph construction from short or long reads."
- [discussion] Large synthetic library insert sizes achievable: "Fast-SG is able to extract synthetic libraries of even bacterial artificial chromosome clone sizes with insert sizes of 150–180 kb"
