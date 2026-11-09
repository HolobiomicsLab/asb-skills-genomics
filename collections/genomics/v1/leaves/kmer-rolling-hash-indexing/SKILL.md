---
name: kmer-rolling-hash-indexing
description: Use when constructing a scaffolding graph or read-placement map alignment-free
  from short (Illumina mate-pair) or long reads against contig sequences, without
  a full read aligner.
when_to_use_negative:
- 'A base-quality-aware or base-level alignment is required (this is alignment-free:
  it yields read locations/linking information, not base-resolved alignments)'
- Insert-size or orientation information is needed for long-range mate-pair technologies
  like 10X Genomics or DOVETAIL, where gap-size estimation is challenging because
  linking comes from a range of insert sizes and orientation may be unknown
- The k-mer size needed falls outside the supported 12-256 range
edam_operation: http://edamontology.org/operation_3198
edam_topics:
- http://edamontology.org/topic_0102
- http://edamontology.org/topic_0622
tools:
- name: Fast-SG
  role: alignment-free scaffolding-graph builder that owns the unique k-mer index,
    rolling-hash queries, windowed scoring, pair rescue, and SAM output
  repo: https://github.com/adigenova/fast-sg
- name: KMC
  role: disk-based k-mer counter used to enumerate unique k-mers of the contig set
    for the index (KMC3 binaries placed inside the Fast-SG directory)
  repo: https://github.com/refresh-bio/KMC
- name: BBHash
  role: minimal perfect hash function storing the unique k-mer set (~3 bits/element,
    single-header, no dependencies)
  repo: https://github.com/rizkg/BBHash
- name: quasi_dictionary
  role: probabilistic membership/key-value structure over the BBHash MPHF; correct
    values for indexed k-mers with false positives below 1/2^fingerprint_size
  repo: https://github.com/pierrepeterlongo/quasi_dictionary
- name: ntHash
  role: rolling hash of read k-mers used to query the unique k-mer index while streaming
    reads
  repo: https://github.com/bcgsc/ntHash
- name: kseqcpp
  role: C++ FASTA/FASTQ streaming parser (port of kseq.h) used to read contigs and
    read libraries
  repo: https://github.com/ctSkennerton/kseqcpp
- name: g++ 5.3
  role: compiler used to build the index and Fast-SG on Linux (clang 4.2 on Mac OSX)
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/gigascience/giy048
    title: 'Fast-SG: an alignment-free algorithm for hybrid assembly'
  source_content_hash: sha256:1754382b6f2df38a002f41cfbfc35e1ed2ccb18ea1281198135768c21e5c9b39
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/adigenova/fast-sg
  tools:
  - BBHash
  - Fast-SG
  - KMC
  - g++
  - g++ 5.3
  - kseqcpp
  - ntHash
  - quasi_dictionary
  edam_operation: http://edamontology.org/operation_3198
  edam_topics:
  - http://edamontology.org/topic_0102
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6923076923076923
n_claims: 13
---

# kmer-rolling-hash-indexing

## Summary

Builds a light-weight unique k-mer index over contig sequences (KMC3 counting, BBHash minimal perfect hash, quasi_dictionary probabilistic membership, ~87 bits/k-mer) and queries it with ntHash rolling hashes of read k-mers to localize reads alignment-free for scaffolding graph construction.

## When to use

Use when constructing a scaffolding graph or read-placement map alignment-free from short (Illumina mate-pair) or long reads against contig sequences, without a full read aligner.

## When NOT to use

- A base-quality-aware or base-level alignment is required (this is alignment-free: it yields read locations/linking information, not base-resolved alignments)
- Insert-size or orientation information is needed for long-range mate-pair technologies like 10X Genomics or DOVETAIL, where gap-size estimation is challenging because linking comes from a range of insert sizes and orientation may be unknown
- The k-mer size needed falls outside the supported 12-256 range

## Inputs

- FASTA/FASTQ contig sequences to index (e.g. example/ecoli-illumina.fa.gz, gzipped supported)
- Read libraries in FASTA/FASTQ declared in a read configuration file (e.g. examples/ecoli-reads.txt)
- k-mer size or range (12-256; e.g. -k 15 or -k 15-25:5)

## Outputs

- Unique k-mer index (KMC3 database backed by BBHash minimal perfect hash + quasi_dictionary membership structure)
- Per-read hit vectors and windowed placement scores
- Pair/read locations in SAM format under the output prefix given by -p (SAM flags: 1:single, 2:paired)

## How to apply

Count k-mers on the target contigs (e.g. example/ecoli-illumina.fa.gz) with KMC3 for a k-mer size in the supported 12-256 range, and store the unique k-mer set in a BBHash minimal perfect hash wrapped by a quasi_dictionary (false-positive rate controlled by fingerprint size, ~87 bits/k-mer footprint). Parse contigs and reads with kseqcpp, streaming the libraries listed in the read configuration file (e.g. examples/ecoli-reads.txt). Compute ntHash rolling hashes of every read k-mer to query the index and accumulate per-read hit vectors against indexed contigs, then apply windowed scoring over the hit vectors to localize each read. For paired Illumina reads, report a pair location only when the forward read scores above 3 and each read scores above 5; otherwise attempt pair rescue with the documented thresholds (3/5/4) to retain mates whose individual placement is weak but joint placement is consistent. Emit pair locations in SAM format honoring the read configuration flags (1:single, 2:paired); k can also be swept over a range (e.g. -k 15-25:5).

## Related tools

- **Fast-SG** (alignment-free scaffolding-graph builder that owns the unique k-mer index, rolling-hash queries, windowed scoring, pair rescue, and SAM output) — https://github.com/adigenova/fast-sg
- **KMC** (disk-based k-mer counter used to enumerate unique k-mers of the contig set for the index (KMC3 binaries placed inside the Fast-SG directory)) — https://github.com/refresh-bio/KMC
- **BBHash** (minimal perfect hash function storing the unique k-mer set (~3 bits/element, single-header, no dependencies)) — https://github.com/rizkg/BBHash
- **quasi_dictionary** (probabilistic membership/key-value structure over the BBHash MPHF; correct values for indexed k-mers with false positives below 1/2^fingerprint_size) — https://github.com/pierrepeterlongo/quasi_dictionary
- **ntHash** (rolling hash of read k-mers used to query the unique k-mer index while streaming reads) — https://github.com/bcgsc/ntHash
- **kseqcpp** (C++ FASTA/FASTQ streaming parser (port of kseq.h) used to read contigs and read libraries) — https://github.com/ctSkennerton/kseqcpp
- **g++ 5.3** (compiler used to build the index and Fast-SG on Linux (clang 4.2 on Mac OSX))

## Examples

```
./FAST-SG.pl -k 15 -l example/ecoli-reads.txt -r example/ecoli-illumina.fa.gz -p test
```

## Evaluation signals

- 'make test' on the small bundled test compiles and runs, and the emitted SAM file exists and parses as valid SAM
- The SAM output contains pair records (flag 2) for every paired library declared in ecoli-reads.txt and single records (flag 1) for single-end libraries
- Index memory footprint is near the documented ~87 bits/k-mer for the chosen k within 12-256
- Pair-rescue behavior: pairs with weak individual placement (forward read score <= 3 or per-read score <= 5) are retained only when joint placement is consistent under the 3/5/4 thresholds
- Sweeping k over a range (e.g. -k 15-25:5) yields consistent scaffolding-linking locations rather than divergent placements

## Limitations

- quasi_dictionary queries have false positives at rate < 1/2^fingerprint_size (values for non-indexed k-mers may be returned; indexed k-mers always return correct values)
- Synthetic library insert size is restricted to the actual long-read size
- Gap-size estimation is harder for long-range mate-pair technologies (10X Genomics, DOVETAIL) because linking comes from a range of insert sizes and read-pair orientation may be unknown
- Compilation tested with g++ 5.3 (Linux) and clang 4.2 (Mac OSX); KMC creates hundreds of temporary files and needs a raised open-file limit on Mac OS (ulimit -n 2048)

## Evidence

- [abstract] Fast-SG is an ultrafast alignment-free method for constructing a scaffolding graph: "uses a new ultrafast alignment-free algorithm specifically designed for constructing a scaffolding graph using light-weight data structures"
- [methods] Pair scoring thresholds and rescue: [evidence span withheld — anchor status: not_found]
- [methods] Index structure and footprint: [evidence span withheld — anchor status: not_found]
- [methods] Example invocation with k-mer range: "./FAST-SG.pl  -k 15-25:5 -l example/ecoli-reads.txt -r example/ecoli-illumina.fa.gz -p test"
- [methods] Supported k-mer range: [evidence span withheld — anchor status: not_found]
- [discussion] Compilation environment: "Compilation was tested with g++ version 5.3 (Linux) and clang version 4.2 (Mac OSX)"
