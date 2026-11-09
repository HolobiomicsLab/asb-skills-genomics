---
name: cpp-kmer-index-implementation
description: Use when building a scaffolding graph from Illumina mate-pair or long
  reads without a base-level aligner, or when implementing/porting Fast-SG's unique-k-mer
  index, hit-vector scoring, and pair-rescue logic in C++.
when_to_use_negative:
- When base-level alignment identity, CIGAR strings, or per-base mismatches are required
  — the index only reports read locations/scoring, not alignments
- When gap-size estimation from long-range mate-pair technologies (10X Genomics, DOVETAIL
  genomics) is the goal — insert size is restricted to the actual long-read size and
  pair orientation may be unknown
- For k-mer sizes outside the supported 12-256 range, or single-contig-free inputs
  with no reference contigs to index
edam_operation: http://edamontology.org/operation_0296
edam_topics:
- http://edamontology.org/topic_0196
- http://edamontology.org/topic_0622
tools:
- name: Fast-SG
  role: Alignment-free scaffolding-graph constructor whose k-mer index, hit-vector
    scoring, and pair-rescue steps are being implemented
  repo: https://github.com/adigenova/fast-sg
- name: KMC
  role: Disk-based k-mer counter used to count k-mers on the contig FASTA; KMC3 binaries
    are placed inside the Fast-SG directory
  repo: https://github.com/refresh-bio/KMC
- name: kseqcpp
  role: C++ port of kseq.h used to stream contigs and reads during parsing
  repo: https://github.com/ctSkennerton/kseqcpp
- name: ntHash
  role: Rolling hash over read k-mers used to query the unique-k-mer index
  repo: https://github.com/bcgsc/ntHash
- name: BBHash
  role: Minimal perfect hash function over the unique k-mer set; single-header library
    with no dependencies
  repo: https://github.com/rizkg/BBHash
- name: quasi_dictionary
  role: Probabilistic membership/key-value structure over the BBHash function providing
    indexed-k-mer values with controlled false positives
  repo: https://github.com/pierrepeterlongo/quasi_dictionary
- name: g++
  role: Compiler used to build the C++ modules (tested g++ 5.3 on Linux, clang 4.2
    on Mac OSX)
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/gigascience/giy048
    title: 'Fast-SG: an alignment-free algorithm for hybrid assembly'
  source_content_hash: sha256:bc2a02cf80b4db4c3a476e5dc2d25801e1b367f7e90ddd0d40cdf049db0a89f6
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
  edam_operation: http://edamontology.org/operation_0296
  edam_topics:
  - http://edamontology.org/topic_0196
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9333333333333333
n_claims: 15
---

# cpp-kmer-index-implementation

## Summary

Implements Fast-SG's alignment-free C++ k-mer index pipeline: KMC3 counting over contigs, a BBHash minimal perfect hash wrapped in a quasi_dictionary probabilistic membership structure, ntHash rolling-hash look-ups to fill per-read hit vectors, and windowed scoring with pair rescue to emit scaffolding links in SAM format.

## When to use

Use when building a scaffolding graph from Illumina mate-pair or long reads without a base-level aligner, or when implementing/porting Fast-SG's unique-k-mer index, hit-vector scoring, and pair-rescue logic in C++.

## When NOT to use

- When base-level alignment identity, CIGAR strings, or per-base mismatches are required — the index only reports read locations/scoring, not alignments
- When gap-size estimation from long-range mate-pair technologies (10X Genomics, DOVETAIL genomics) is the goal — insert size is restricted to the actual long-read size and pair orientation may be unknown
- For k-mer sizes outside the supported 12-256 range, or single-contig-free inputs with no reference contigs to index

## Inputs

- Contig/assembly FASTA (possibly gzipped, e.g. example/ecoli-illumina.fa.gz) to be k-mer indexed
- Read configuration file listing FASTQ/FASTA libraries (e.g. examples/ecoli-reads.txt) with library type flags (1:single, 2:paired)
- k-mer size parameter -k (scalar, e.g. 15, or a range such as 15-25:5) within 12-256
- Output prefix -p for the SAM-formatted results

## Outputs

- SAM-format file of pair locations / synthetic-library linking records under the -p prefix
- Scaffolding graph edges (linking information) usable by short-read scaffolders or Fast-SG's hybrid assembly pipeline
- KMC3 k-mer database over the indexed contigs (kmc_pre/kmc_suf files)

## How to apply

Count k-mers on the target contigs (e.g. example/ecoli-illumina.fa.gz) with KMC3, choosing a k within the supported 12-256 range, and load the unique k-mer set into a BBHash minimal perfect hash backed by the quasi_dictionary probabilistic membership structure (~87 bits/k-mer footprint). Parse contigs and reads with kseqcpp, streaming the libraries listed in the read configuration file (e.g. examples/ecoli-reads.txt), and compute ntHash rolling hashes for every read k-mer to query the index, accumulating per-read hit vectors against the indexed contigs. Apply windowed scoring over the hit vectors to localize each read; for Illumina mate-pair libraries, report pair locations only when the forward read scores above 3 and each read scores above 5, otherwise attempt pair rescue using the documented threshold triplet (3/5/4) to retain mates whose individual placement is weak but joint placement is consistent. Emit pair locations as SAM with the read-configuration flag convention (1:single, 2:paired), honoring the -p output prefix. Rationale: this light-weight index lets Fast-SG construct the scaffolding graph faster than state-of-the-art short-read aligners while reusing downstream Illumina algorithms for hybrid assembly.

## Related tools

- **Fast-SG** (Alignment-free scaffolding-graph constructor whose k-mer index, hit-vector scoring, and pair-rescue steps are being implemented) — https://github.com/adigenova/fast-sg
- **KMC** (Disk-based k-mer counter used to count k-mers on the contig FASTA; KMC3 binaries are placed inside the Fast-SG directory) — https://github.com/refresh-bio/KMC
- **kseqcpp** (C++ port of kseq.h used to stream contigs and reads during parsing) — https://github.com/ctSkennerton/kseqcpp
- **ntHash** (Rolling hash over read k-mers used to query the unique-k-mer index) — https://github.com/bcgsc/ntHash
- **BBHash** (Minimal perfect hash function over the unique k-mer set; single-header library with no dependencies) — https://github.com/rizkg/BBHash
- **quasi_dictionary** (Probabilistic membership/key-value structure over the BBHash function providing indexed-k-mer values with controlled false positives) — https://github.com/pierrepeterlongo/quasi_dictionary
- **g++** (Compiler used to build the C++ modules (tested g++ 5.3 on Linux, clang 4.2 on Mac OSX))

## Examples

```
./FAST-SG.pl -k 15 -l example/ecoli-reads.txt -r example/ecoli-illumina.fa.gz -p test
```

## Evaluation signals

- 'make test' compiles and passes on the bundled E. coli example, producing a SAM file under the -p prefix
- The emitted SAM file parses as valid SAM and contains pair records for every paired library declared in the read configuration file
- Index memory footprint is near the documented ~87 bits/k-mer, and all queried k-mer sizes fall within 12-256
- Pair locations respect the documented Illumina thresholds: forward read score > 3, each read score > 5, with pair rescue attempted (3/5/4 triplet) otherwise
- Linking records recapitulate contig adjacencies, e.g. synthetic mate-pair libraries from the E. coli K12 dataset scaffold the DiscovarDeNovo assembly

## Limitations

- Dependency versions are pinned to specific commits in the paper (e.g. kseqcpp cfa50bcd17bbcb3225d431df4a2c1396f58a0993, ntHash ff326a8c9ccf6186f42c1f49950c1ebaadbd7f7a, BBHash 99c905828a58fa119979df5c26bdbea93f0a7696, quasi_dictionary 9e8c64b150b129035f92d010a12085bd6c9490f0) and the README is truncated in context
- quasi_dictionary is probabilistic: queries of non-indexed k-mers can return false positives at a rate below 1/2^fingerprint_size, so hit vectors are approximate
- KMC creates hundreds of temporary files and on Mac OS X the default open-file limit may need raising (ulimit -n 2048); KMC needs C++17 features and zlib for gzipped input
- The insert size of extracted synthetic libraries is restricted to the actual long-read size, and gap-size estimation with long-range mate-pair technologies (10X Genomics, DOVETAIL genomics) is challenging because linking information comes from a range of insert sizes with possibly unknown read-pair orientation
- No version number is stated for Fast-SG itself (only RRID SCR_015934) and no changelog is available

## Evidence

- [readme] Alignment-free scaffolding graph construction from short or long reads: "Fast-SG is an alignment-free algorithm for ultrafast scaffolding graph construction from short or long reads."
- [discussion] Compiler support for building the C++ modules: "Compilation was tested with g++ version 5.3 (Linux) and clang version 4.2 (Mac OSX)"
- [methods] Dependency pinning for kseqcpp in the methods: "kseqcpp (https://github.com/ctSkennerton/kseqcpp.git checkout cfa50bcd17bbcb3225d431df4a2c1396f58a0993)"
- [readme] quasi_dictionary design over BBHash: [evidence span withheld — anchor status: external_doc]
- [readme] BBHash suitability as the index backbone: [evidence span withheld — anchor status: external_doc]
- [abstract] Fast-SG outperforms short-read aligners for scaffolding graph construction: "Fast-SG outperforms the state-of-the-art short-read aligners when building the scaffoldinggraph"
- [discussion] Long-read linking capability of the same index: "Fast-SG is able to extract synthetic libraries of even bacterial artificial chromosome clone sizes"
