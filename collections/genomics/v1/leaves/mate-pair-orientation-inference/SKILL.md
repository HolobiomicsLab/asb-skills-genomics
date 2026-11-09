---
name: mate-pair-orientation-inference
description: Use when synthetic mate-pair libraries have been extracted from long
  reads (PacBio or Oxford Nanopore) and aligned back to a contig set, and per-library
  insert size, standard deviation, and FR/RF orientation statistics must be estimated
  before scaffolding.
when_to_use_negative:
- The input is real long-range mate-pair data (e.g. 10X Genomics or DOVETAIL Genomics
  libraries) rather than Fast-SG synthetic libraries extracted from long reads — such
  data span a range of insert sizes with possibly unknown orientation and need different
  gap-size estimation.
- The requested insert size D exceeds the actual long-read length, since Fast-SG restricts
  the insert size to the actual long-read size.
- Statistics are needed for libraries whose read pairs map across different contigs
  — only within-contig aligned pairs are used for the estimation.
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: Fast-SG
  role: Extracts the synthetic 200 bp forward/reverse mate-pair libraries from long
    reads via a 100 bp sliding window at each user-specified insert size, then aligns
    them back to contigs and estimates per-library insert size, standard deviation,
    and FR/RF orientation
  repo: https://github.com/adigenova/fast-sg
- name: FAST-SG.pl
  role: PERL driver that runs Fast-SG with k-mer size, library configuration file,
    contigs/reference, and output prefix
  repo: https://github.com/adigenova/fastsg
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/gigascience/giy048
    title: 'Fast-SG: an alignment-free algorithm for hybrid assembly'
  source_content_hash: sha256:5405eee09705e6d8316b99b7920d315ca07305dc224265fe2a787be42769e343
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/adigenova/fast-sg
  tools:
  - C++
  - FAST-SG.pl
  - Fast-SG
  - PERL
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# mate-pair-orientation-inference

## Summary

Estimates per-library insert-size statistics (mean, standard deviation) and read-pair orientation (FR/RF) by aligning Fast-SG synthetic mate pairs back to contigs and keeping only within-contig pairs, after trimming the extreme 10% of the insert-size distribution as outliers.

## When to use

Use when synthetic mate-pair libraries have been extracted from long reads (PacBio or Oxford Nanopore) and aligned back to a contig set, and per-library insert size, standard deviation, and FR/RF orientation statistics must be estimated before scaffolding.

## When NOT to use

- The input is real long-range mate-pair data (e.g. 10X Genomics or DOVETAIL Genomics libraries) rather than Fast-SG synthetic libraries extracted from long reads — such data span a range of insert sizes with possibly unknown orientation and need different gap-size estimation.
- The requested insert size D exceeds the actual long-read length, since Fast-SG restricts the insert size to the actual long-read size.
- Statistics are needed for libraries whose read pairs map across different contigs — only within-contig aligned pairs are used for the estimation.

## Inputs

- Long-read library FASTQ files (raw or error-corrected PacBio / Oxford Nanopore reads)
- Read configuration file, space-separated: type, library ID, path, insert sizes (e.g. 1000, 2000, 3000, 5000), pairing flag
- Contig set (assembly) to which synthetic pairs are aligned
- Alignments of synthetic 200 bp forward/reverse pairs back to the contigs

## Outputs

- library_stats.tsv with one row per library: library ID, estimated insert size, standard deviation, orientation (FR/RF)
- Synthetic mate-pair FASTQ files of 200 bp forward/reverse reads separated by insert size D

## How to apply

Parse the read configuration file (space-separated fields: type, library ID, path, insert sizes, pairing flag) to obtain the long-read libraries and their target insert sizes (e.g. 1000, 2000, 3000, 5000). For each library, slide a 100 bp window along each long read and, at each user-specified insert size D, excise a forward 200 bp read from one end and a reverse 200 bp read from the opposite end, emitting FASTQ synthetic pairs restricted to the actual long-read length. Align the synthetic pairs back to the contig set and retain only pairs mapping within the same contig for statistics. For each library, trim the lowest and highest 10% of the within-contig insert-size distribution as outliers, compute the mean and standard deviation, and infer the read-pair orientation (FR/RF) from the mapping coordinates. Write the results as a per-library table (library ID, estimated insert size, standard deviation, orientation) to library_stats.tsv.

## Related tools

- **Fast-SG** (Extracts the synthetic 200 bp forward/reverse mate-pair libraries from long reads via a 100 bp sliding window at each user-specified insert size, then aligns them back to contigs and estimates per-library insert size, standard deviation, and FR/RF orientation) — https://github.com/adigenova/fast-sg
- **FAST-SG.pl** (PERL driver that runs Fast-SG with k-mer size, library configuration file, contigs/reference, and output prefix) — https://github.com/adigenova/fastsg

## Examples

```
./FAST-SG.pl -k 15 -l example/ecoli-reads.txt -r example/ecoli-illumina.fa.gz -p test
```

## Evaluation signals

- library_stats.tsv exists and contains exactly one row per library configured in the read configuration file
- Estimated insert sizes are consistent with the user-specified insert sizes (e.g. 1000, 2000, 3000, 5000 bp)
- The orientation field is populated with FR or RF and is consistent with the mapping coordinates of the aligned pairs
- Standard deviations are computed after trimming the lowest and highest 10% of the insert-size distribution; estimates should be tight enough to reflect the fixed user-specified D
- Only synthetic pairs mapping within the same contig contribute to the statistics

## Limitations

- The insert size is restricted to the actual long-read size, so requested insert sizes larger than the long reads cannot be realized.
- Gap-size estimation with existing long-range mate-pair technologies (10X Genomics and DOVETAIL Genomics) is more challenging than with these synthetic libraries, because linking information comes from a range of insert sizes and read-pair orientation may be unknown.
- The 10% tail-trimming of the insert-size distribution may bias mean/standard-deviation estimates for skewed or noisy long-read alignments.

## Evidence

- [discussion] The insert size is restricted to the actual long-read size.: "The insert size is restricted to the actual long-read size."
- [discussion] An estimation of the gap size with the existing long-range mate-pair technologies (10X genomics and DOVETAIL genomics) is more challenging than with the synthetic libraries: "An estimation of the gap size with the existing long-range mate-pair technologies (10X genomics and DOVETAIL genomics) is more challenging than with the synthetic libraries"
- [discussion] near-perfect synthetic libraries are obtained with Fast-SG from either corrected or uncorrected PacBio and Nanopore long reads: "near-perfect synthetic libraries are obtained with Fast-SG from either corrected or uncorrected PacBio and Nanopore long reads"
- [methods] Example of a read configuration file can be found in examples/ecoli-reads.txt: "Example of a read configuration file can be found in examples/ecoli-reads.txt"
- [readme] ./FAST-SG.pl  -k 15 -l example/ecoli-reads.txt -r example/ecoli-illumina.fa.gz -p test: "./FAST-SG.pl  -k 15 -l example/ecoli-reads.txt -r example/ecoli-illumina.fa.gz -p test"
