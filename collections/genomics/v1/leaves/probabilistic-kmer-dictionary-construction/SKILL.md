---
name: probabilistic-kmer-dictionary-construction
description: Use when you need to index the unique k-mer content of reference contigs
  for fast k-mer look-up in reads (e.g. scaffolding-graph construction), especially
  for k-mer sizes in the 12–256 range and memory-frugal probabilistic storage.
when_to_use_negative:
- When exact (zero false-positive) k-mer look-up is required — quasi_dictionary returns
  values for non-indexed keys at a rate below 1/2^fingerprint_size, so it is unsuitable
  where any false positive is intolerable.
- When k-mer size outside the supported 12–256 range is needed, or when the dictionary
  would be better served by an exact hash map for a small reference.
- When you need base-level alignments rather than alignment-free k-mer linking — this
  index supports Fast-SG's scaffolding-graph construction, not read alignment/variant
  calling.
edam_operation: null
edam_topics:
- http://edamontology.org/topic_0102
- http://edamontology.org/topic_3168
tools:
- name: KMC
  role: Disk-based k-mer counter used to count k-mers over the contig sequences and
    supply the unique k-mer set for the index
  repo: https://github.com/refresh-bio/KMC
- name: BBHash
  role: Builds the minimal perfect hash function over the unique k-mer set (~3 bits/element,
    fast construction, no additional memory)
  repo: https://github.com/rizkg/BBHash
- name: quasi_dictionary
  role: Probabilistic membership/key-value structure wrapped around the BBHash MPHF,
    adding fingerprints so indexed keys return correct values with false positives
    below 1/2^fingerprint_size
  repo: https://github.com/pierrepeterlongo/quasi_dictionary
- name: ntHash
  role: Rolling hash function used to hash read k-mers for querying the unique-k-mer
    index
  repo: https://github.com/bcgsc/ntHash
- name: kseqcpp
  role: C++ FASTA/FASTQ parser used to stream contigs and reads into the indexing
    and look-up pipeline
  repo: https://github.com/ctSkennerton/kseqcpp
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/gigascience/giy048
    title: 'Fast-SG: an alignment-free algorithm for hybrid assembly'
  source_content_hash: sha256:0e4ee00a3be9fd1840d0b75eb8cfb39687f61f08b80859af277d08aac6f5768d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/refresh-bio/KMC
  tools:
  - BBHash
  - Fast-SG
  - KMC
  - g++
  - kseqcpp
  - ntHash
  - quasi_dictionary
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0102
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5454545454545454
n_claims: 11
---

# probabilistic-kmer-dictionary-construction

## Summary

Builds a light-weight, alignment-free k-mer dictionary over contig sequences by counting k-mers with KMC3 and storing the unique set in a BBHash minimal perfect hash wrapped in a quasi_dictionary probabilistic membership structure, as used by Fast-SG to enable ultrafast scaffolding-graph construction.

## When to use

Use when you need to index the unique k-mer content of reference contigs for fast k-mer look-up in reads (e.g. scaffolding-graph construction), especially for k-mer sizes in the 12–256 range and memory-frugal probabilistic storage.

## When NOT to use

- When exact (zero false-positive) k-mer look-up is required — quasi_dictionary returns values for non-indexed keys at a rate below 1/2^fingerprint_size, so it is unsuitable where any false positive is intolerable.
- When k-mer size outside the supported 12–256 range is needed, or when the dictionary would be better served by an exact hash map for a small reference.
- When you need base-level alignments rather than alignment-free k-mer linking — this index supports Fast-SG's scaffolding-graph construction, not read alignment/variant calling.

## Inputs

- Reference/contig sequences in (gzipped) FASTA, e.g. example/ecoli-illumina.fa.gz from the Fast-SG test set
- Chosen k-mer size in the supported 12–256 range (e.g. -k 15)
- Read FASTA/FASTQ files listed in a read configuration file such as examples/ecoli-reads.txt

## Outputs

- A KMC3 k-mer count database (.kmc_pre / .kmc_suf) over the contig sequences
- A BBHash minimal perfect hash function over the unique k-mer set
- A quasi_dictionary probabilistic index (~87 bits/k-mer) mapping unique k-mers to contig positions for downstream k-mer look-up

## How to apply

First count k-mers over the target contig FASTA (e.g. example/ecoli-illumina.fa.gz) using KMC3, whose disk-based counting scales to large genomes and supports k up to 256. Select a k-mer size within the supported 12–256 range and restrict the KMC database to the unique k-mer set, since only unique k-mers give unambiguous read-to-contig placements. Feed the unique set to BBHash to construct a minimal perfect hash (BBHash costs roughly 3 bits/element and construction is fast and memory-light), then wrap it in the quasi_dictionary, which associates each key with a fingerprint value so indexed elements always return correct values while false-positive queries occur at a rate below 1/2^fingerprint_size. The paper documents a combined footprint of ~87 bits per k-mer for this structure, which is what allows Fast-SG to use 'light-weight data structures' for scaffolding-graph construction from short or long reads. Reads are then parsed with kseqcpp and their k-mers hashed with ntHash rolling hashes to query the dictionary and fill per-read hit vectors against the indexed contigs.

## Related tools

- **KMC** (Disk-based k-mer counter used to count k-mers over the contig sequences and supply the unique k-mer set for the index) — https://github.com/refresh-bio/KMC
- **BBHash** (Builds the minimal perfect hash function over the unique k-mer set (~3 bits/element, fast construction, no additional memory)) — https://github.com/rizkg/BBHash
- **quasi_dictionary** (Probabilistic membership/key-value structure wrapped around the BBHash MPHF, adding fingerprints so indexed keys return correct values with false positives below 1/2^fingerprint_size) — https://github.com/pierrepeterlongo/quasi_dictionary
- **ntHash** (Rolling hash function used to hash read k-mers for querying the unique-k-mer index) — https://github.com/bcgsc/ntHash
- **kseqcpp** (C++ FASTA/FASTQ parser used to stream contigs and reads into the indexing and look-up pipeline) — https://github.com/ctSkennerton/kseqcpp

## Examples

```
git clone https://github.com/adigenova/fastsg && make all && make test; ./FAST-SG.pl -k 15 -l example/ecoli-reads.txt -r example/ecoli-illumina.fa.gz -p test
```

## Evaluation signals

- Compilation succeeds with g++ (tested with version 5.3 on Linux) or clang 4.2 (Mac OSX), and 'make test' passes on the small E. coli test set
- The KMC database is produced for the chosen k (within 12–256) and restricted to unique k-mers before MPHF construction
- Index memory footprint is consistent with the documented ~87 bits per k-mer for the BBHash + quasi_dictionary combination
- Downstream k-mer look-up with ntHash rolling hashes fills hit vectors and yields pair locations for the paired libraries declared in ecoli-reads.txt
- Values returned by the quasi_dictionary for indexed k-mers are correct; false-positive rate stays below 1/2^fingerprint_size as documented

## Limitations

- quasi_dictionary queries have false positives (rate below 1/2^fingerprint_size) for keys that were never indexed; only values for indexed elements are guaranteed correct.
- K-mer size is restricted to the supported 12–256 range, and KMC's MAX_K (default 256) must be redefined at compile time for larger k.
- KMC creates hundreds of intermediate/temporary files (and on MacOS the open-file limit may need raising, e.g. 'ulimit -n 2048').
- No version number is stated for Fast-SG itself (only RRID SCR_015934), so exact release provenance of the index code must be taken from the pinned library checkouts.

## Evidence

- [abstract] uses a new ultrafast alignment-free algorithm specifically designed for constructing a scaffolding graph using light-weight data structures: "we propose a new method, called Fast-SG, that uses a new ultrafast alignment-free algorithm specifically designed for constructing a scaffolding graph using light-weight data structures"
- [methods] KMC binaries and version: "wget https://github.com/refresh-bio/KMC/releases/download/v3.0.0/KMC3.linux.tar.gz"
- [methods] Pinned library dependencies: "quasi_dictionary (https://github.com/pierrepeterlongo/quasi_dictionary.git checkout 9e8c64b150b129035f92d010a12085bd6c9490f0)"
- [readme] quasi_dictionary semantics: [evidence span withheld — anchor status: external_doc]
- [readme] BBHash size/performance: [evidence span withheld — anchor status: external_doc]
- [readme] KMC k-mer length limit: [evidence span withheld — anchor status: external_doc]
