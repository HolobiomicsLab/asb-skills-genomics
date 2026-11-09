---
name: fasta-read-pair-simulation
description: Use when long reads (PacBio, Oxford Nanopore, raw or error-corrected)
  must be converted into synthetic mate-pair FASTQ libraries with defined insert sizes
  for scaffolding or library parameter estimation.
when_to_use_negative:
- The input is already short-read Illumina mate-pair or paired-end data — no simulation/extraction
  is needed and real insert-size distributions should be estimated from the aligned
  reads directly.
- Long reads are too short to span the requested insert sizes — since the insert size
  is restricted to the actual long-read size, no pairs will be produced at that insert
  size; use smaller D or longer reads.
- Gap-size estimation requiring true insert-size distributions from commercial long-range
  technologies (10X Genomics, DOVETAIL Genomics), where linking information comes
  from a range of insert sizes and read-pair orientation may be unknown — synthetic
  libraries are comparatively easier but not equivalent.
edam_topics:
- http://edamontology.org/topic_0196
- http://edamontology.org/topic_0102
tools:
- name: Fast-SG
  role: Extracts synthetic mate-pair libraries from long reads and builds scaffolding
    graphs alignment-free; implemented in C++ and PERL
  repo: https://github.com/adigenova/fast-sg
- name: PERL
  role: Driver language for the FAST-SG.pl wrapper that orchestrates library extraction
    runs
- name: C++
  role: Implementation language of the Fast-SG core extraction/scaffolding-graph algorithms
- name: KMC
  role: k-mer counter whose binaries are bundled with Fast-SG for the underlying alignment-free
    k-mer indexing
  repo: https://github.com/refresh-bio/KMC
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/gigascience/giy048
    title: 'Fast-SG: an alignment-free algorithm for hybrid assembly'
  source_content_hash: sha256:4dbdebbad8c0937ae5c8f9fd2271e37ee239043aaa9e8b17234ff74163aa9a32
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
yes_rate: 0.6666666666666666
n_claims: 6
---

# fasta-read-pair-simulation

## Summary

Extracts synthetic mate-pair libraries from long-read (FASTA/FASTQ) sequences in the Fast-SG style: forward/reverse 200 bp reads are excised at user-specified insert sizes so short-read algorithms can exploit long-range linking information without an aligner.

## When to use

Use when long reads (PacBio, Oxford Nanopore, raw or error-corrected) must be converted into synthetic mate-pair FASTQ libraries with defined insert sizes for scaffolding or library parameter estimation.

## When NOT to use

- The input is already short-read Illumina mate-pair or paired-end data — no simulation/extraction is needed and real insert-size distributions should be estimated from the aligned reads directly.
- Long reads are too short to span the requested insert sizes — since the insert size is restricted to the actual long-read size, no pairs will be produced at that insert size; use smaller D or longer reads.
- Gap-size estimation requiring true insert-size distributions from commercial long-range technologies (10X Genomics, DOVETAIL Genomics), where linking information comes from a range of insert sizes and read-pair orientation may be unknown — synthetic libraries are comparatively easier but not equivalent.

## Inputs

- Long-read sequence files (FASTA/FASTQ; PacBio or Oxford Nanopore, raw or error-corrected)
- Read configuration file (space-separated: type, library ID, path, insert sizes, pairing flag), e.g. examples/ecoli-reads.txt
- Contig set for aligning synthetic pairs during library parameter estimation

## Outputs

- Synthetic mate-pair FASTQ read pairs (200 bp forward/reverse reads separated by insert size D)
- Per-library statistics table library_stats.tsv (library ID, estimated insert size, standard deviation, FR/RF orientation)

## How to apply

Parse a space-separated read configuration file (type, library ID, path, insert sizes, pairing flag) to obtain the long-read libraries and target insert sizes (e.g. 1000, 2000, 3000, 5000; Fast-SG also supports bacterial-artificial-chromosome-scale inserts of 150–180 kb). For each library, slide a default 100 bp moving window along each long read and, at each user-specified insert size D, excise a forward 200 bp read from one end and a reverse 200 bp read from the opposite end, emitting synthetic FASTQ pairs. Critically, the insert size is restricted to the actual long-read length, so pairs are only emitted where the long read is long enough to span D. After alignment of the synthetic pairs back to the contigs, retain only same-contig pairs, trim the lowest and highest 10% of the insert-size distribution as outliers, and estimate the mean insert size, standard deviation, and FR/RF orientation per library.

## Related tools

- **Fast-SG** (Extracts synthetic mate-pair libraries from long reads and builds scaffolding graphs alignment-free; implemented in C++ and PERL) — https://github.com/adigenova/fast-sg
- **PERL** (Driver language for the FAST-SG.pl wrapper that orchestrates library extraction runs)
- **C++** (Implementation language of the Fast-SG core extraction/scaffolding-graph algorithms)
- **KMC** (k-mer counter whose binaries are bundled with Fast-SG for the underlying alignment-free k-mer indexing) — https://github.com/refresh-bio/KMC

## Examples

```
./FAST-SG.pl -k 15 -l example/ecoli-reads.txt -r example/ecoli-illumina.fa.gz -p test
```

## Evaluation signals

- library_stats.tsv exists and contains exactly one row per library configured in the read configuration file.
- Estimated insert sizes in library_stats.tsv are consistent with the user-specified insert sizes (e.g. 1000, 2000, 3000, 5000).
- The table includes populated standard-deviation and orientation (FR/RF) fields for each library.
- After aligning synthetic pairs back to the contigs, only pairs mapping within the same contig enter the insert-size statistics.
- Insert-size statistics remain finite and sensible after trimming the lowest and highest 10% of the distribution as outliers.

## Limitations

- The insert size is restricted to the actual long-read size, so pairs at a given insert size D are only generated from long reads long enough to span D.
- No version number is stated for the Fast-SG software itself (only RRID SCR_015934), so reproduce against the repository snapshot.
- Read-pair orientation must be inferred from mapping coordinates (FR/RF), unlike technologies where orientation is fixed by protocol.

## Evidence

- [discussion] Extraction of synthetic libraries from long reads: "Fast-SG is able to extract synthetic libraries of even bacterial artificial chromosome clone sizes with insert sizes of 150–180 kb"
- [discussion] Insert size restricted to read length: "The insert size is restricted to the actual long-read size."
- [methods] Read configuration file: "Example of a read configuration file can be found in examples/ecoli-reads.txt"
- [discussion] Long-read support: "near-perfect synthetic libraries are obtained with Fast-SG from either corrected or uncorrected PacBio and Nanopore long reads"
- [readme] Fast-SG wrapper invocation: "./FAST-SG.pl  -k 15 -l example/ecoli-reads.txt -r example/ecoli-illumina.fa.gz -p test"
