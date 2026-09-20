---
name: alignment-free-read-placement
description: Use when scaffolding or mate-pair linking requires read placement on
  contigs for short (Illumina) or long (PacBio/Nanopore) reads and full alignment
  is too slow or unnecessary.
when_to_use_negative:
- Base-level alignment accuracy (e.g., CIGAR strings, mismatch positions) is required
  rather than contig-level placement/linking
- Gap-size estimation depends on precise insert-size distributions of 10X Genomics
  or DOVETAIL mate-pair data, where read-pair orientation may be unknown (article
  flags this as challenging)
- Insert-size requirement exceeds the actual long-read size, since the synthetic library
  insert size is restricted to the actual long-read length
edam_operation: http://edamontology.org/operation_3198
edam_topics:
- http://edamontology.org/topic_0102
- http://edamontology.org/topic_3673
tools:
- name: Fast-SG
  role: Alignment-free scaffolding-graph/read-placement pipeline driven by FAST-SG.pl
  repo: https://github.com/adigenova/fast-sg
- name: KMC
  role: Disk-based k-mer counting to build the unique k-mer set over contigs
  repo: https://github.com/refresh-bio/KMC
- name: ntHash
  role: Rolling hash of read k-mers for index look-up
  repo: https://github.com/bcgsc/ntHash
- name: BBHash
  role: Minimal perfect hash over the unique k-mer set
  repo: https://github.com/rizkg/BBHash
- name: quasi_dictionary
  role: Probabilistic membership/value structure backing the k-mer index
  repo: https://github.com/pierrepeterlongo/quasi_dictionary
- name: kseqcpp
  role: C++ FASTA/FASTQ streaming parser for contigs and reads
  repo: https://github.com/ctSkennerton/kseqcpp
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/gigascience/giy048
    title: 'Fast-SG: an alignment-free algorithm for hybrid assembly'
  source_content_hash: sha256:376f2cb511f905e0d0acad8e71ed5c4d1dc597a3bdf1ec6c18696222164d4af1
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/adigenova/fast-sg
  tools:
  - BBHash
  - Fast-SG
  - KMC
  - g++
  - kseqcpp
  - ntHash
  - quasi_dictionary
  edam_operation: http://edamontology.org/operation_3198
  edam_topics:
  - http://edamontology.org/topic_0102
  - http://edamontology.org/topic_3673
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8125
n_claims: 16
---

# alignment-free-read-placement

## Summary

Place short or long reads onto contigs without base-level alignment using Fast-SG: an ntHash rolling-hash k-mer look-up against a unique-k-mer index (KMC3 + BBHash + quasi_dictionary), windowed hit-vector scoring, and pair rescue, emitting pair locations in SAM format.

## When to use

Use when scaffolding or mate-pair linking requires read placement on contigs for short (Illumina) or long (PacBio/Nanopore) reads and full alignment is too slow or unnecessary.

## When NOT to use

- Base-level alignment accuracy (e.g., CIGAR strings, mismatch positions) is required rather than contig-level placement/linking
- Gap-size estimation depends on precise insert-size distributions of 10X Genomics or DOVETAIL mate-pair data, where read-pair orientation may be unknown (article flags this as challenging)
- Insert-size requirement exceeds the actual long-read size, since the synthetic library insert size is restricted to the actual long-read length

## Inputs

- Contig/assembly FASTA (possibly gzipped, e.g., example/ecoli-illumina.fa.gz) to be indexed
- Read library configuration file listing read files (e.g., examples/ecoli-reads.txt) with paired/single flags (1:single, 2:paired)
- Short or long reads (Illumina mate-pair FASTQ/FASTA, or PacBio/Nanopore long reads, raw or error-corrected)

## Outputs

- SAM-format file of read/pair locations on contigs under the -p output prefix
- Synthetic long-range read-pair (linking) library usable as input to short-read scaffolders or for scaffolding-graph construction

## How to apply

Build a unique-k-mer index over the contigs (e.g., example/ecoli-illumina.fa.gz) by counting k-mers with KMC3 and storing the unique set in a BBHash minimal perfect hash wrapped by the quasi_dictionary probabilistic membership structure (~87 bits/k-mer footprint; k-mer size 12-256, e.g., -k 15). Stream contigs and reads with kseqcpp from the read list in the library file (e.g., examples/ecoli-reads.txt), compute ntHash rolling hashes of every read k-mer to query the index, and accumulate per-read hit vectors against the indexed contigs. Apply windowed scoring over the hit vectors to localize each read; for Illumina mate-pair reads, report a pair location only when the forward read scores above 3 and each read scores above 5, otherwise attempt pair rescue using the documented thresholds (3/5/4) to retain mates whose individual placement is weak but joint placement is consistent. Emit pair locations as a SAM file under the -p output prefix, honoring the read-configuration flag convention (1:single, 2:paired). Compile with g++ (tested with g++ 5.3 on Linux, clang 4.2 on Mac OSX) and validate with 'make test'.

## Related tools

- **Fast-SG** (Alignment-free scaffolding-graph/read-placement pipeline driven by FAST-SG.pl) — https://github.com/adigenova/fast-sg
- **KMC** (Disk-based k-mer counting to build the unique k-mer set over contigs) — https://github.com/refresh-bio/KMC
- **ntHash** (Rolling hash of read k-mers for index look-up) — https://github.com/bcgsc/ntHash
- **BBHash** (Minimal perfect hash over the unique k-mer set) — https://github.com/rizkg/BBHash
- **quasi_dictionary** (Probabilistic membership/value structure backing the k-mer index) — https://github.com/pierrepeterlongo/quasi_dictionary
- **kseqcpp** (C++ FASTA/FASTQ streaming parser for contigs and reads) — https://github.com/ctSkennerton/kseqcpp

## Examples

```
./FAST-SG.pl -k 15 -l example/ecoli-reads.txt -r example/ecoli-illumina.fa.gz -p test
```

## Evaluation signals

- 'make test' passes with g++ 5.3 (Linux) / clang 4.2 (Mac OSX) compilation
- Emitted SAM file exists, parses as valid SAM, and contains pair records for the paired libraries declared in ecoli-reads.txt
- Forward-read score > 3 and per-read score > 5 gate pair reporting; rescue threshold 4 retains jointly consistent weak mates
- Index footprint matches the documented ~87 bits/k-mer for the chosen k-mer size within 12-256
- Placed pair counts on the assembly (e.g., DiscovarDeNovo E. coli K12 / S. cerevisiae W303 / A. thaliana Ler-0 / human NA12878) are consistent with reported benchmark expectations

## Limitations

- Synthetic library insert size is restricted to the actual long-read size
- Gap-size estimation with 10X Genomics or DOVETAIL long-range mate-pair data is challenging because linking information comes from a range of insert sizes and read-pair orientation may be unknown
- quasi_dictionary queries have false positives at a rate below 1/2^fingerprint_size; indexed elements return correct values
- KMC's maximal k-mer length is 256; Fast-SG supports k-mer sizes 12-256

## Evidence

- [intro] Fast-SG is an alignment-free algorithm for ultrafast scaffolding graph construction from short or long reads.: "Fast-SG is an alignment-free algorithm for ultrafast scaffolding graph construction from short or long reads."
- [methods] kseqcpp parser: "kseqcpp (https://github.com/ctSkennerton/kseqcpp.git checkout cfa50bcd17bbcb3225d431df4a2c1396f58a0993)"
- [methods] FAST-SG.pl invocation: "./FAST-SG.pl  -k 15 -l example/ecoli-reads.txt -r example/ecoli-illumina.fa.gz -p test"
- [readme] quasi_dictionary false positives: [evidence span withheld — anchor status: external_doc]
- [discussion] Fast-SG faster than aligners: "Fast-SG is faster than the current state-of-the-art short-read aligners"
- [discussion] synthetic library insert size limit: "The insert size is restricted to the actual long-read size."
