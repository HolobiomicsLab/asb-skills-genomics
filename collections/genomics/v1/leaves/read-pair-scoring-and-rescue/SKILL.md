---
name: read-pair-scoring-and-rescue
description: Use when constructing a scaffolding graph from Illumina mate-pair (or
  long) reads without a traditional aligner, and you need to score k-mer hits against
  contigs and rescue weakly-placed mates before writing pair locations as SAM.
when_to_use_negative:
- 'Input is raw unlinked long reads where insert size matters: synthetic-library insert
  size is restricted to the actual long-read size, so gap-size estimation differs
  from 10X/DOVETAIL-style mate-pair data'
- You need full base-level alignments with mismatches/CIGAR strings — Fast-SG is alignment-free
  and only reports pair locations, not alignments
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0622
tools:
- name: Fast-SG
  role: Main tool implementing the alignment-free scoring and pair-rescue pipeline;
    run via FAST-SG.pl
  repo: https://github.com/adigenova/fast-sg
- name: ntHash
  role: Rolling hash function computed on every read k-mer to query the unique k-mer
    index
  repo: https://github.com/bcgsc/ntHash
- name: quasi_dictionary
  role: Probabilistic membership structure backing the unique k-mer dictionary (false-positive
    rate below 1/2^fingerprint_size)
  repo: https://github.com/pierrepeterlongo/quasi_dictionary
- name: BBHash
  role: Minimal perfect hash function mapping unique k-mers to positions in the index
  repo: https://github.com/rizkg/BBHash
- name: KMC
  role: Counts k-mers over the contig sequences to define the unique k-mer set
  repo: https://github.com/refresh-bio/KMC
- name: kseqcpp
  role: C++ FASTA/FASTQ parser streaming contigs and reads into the scoring pipeline
  repo: https://github.com/ctSkennerton/kseqcpp
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/gigascience/giy048
    title: 'Fast-SG: an alignment-free algorithm for hybrid assembly'
  source_content_hash: sha256:c655bc72e175dd66984621ccb7d6633fde84b67bcc440854ef460e562e91a5a4
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
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# read-pair-scoring-and-rescue

## Summary

Fast-SG's alignment-free placement of read pairs: ntHash rolling-hash k-mer look-ups against a unique k-mer index fill per-read hit vectors, windowed scoring localizes each mate, and a rescue step (score thresholds 3/5/4) retains pairs whose individual placement is weak but joint placement is consistent, emitting pair locations in SAM format.

## When to use

Use when constructing a scaffolding graph from Illumina mate-pair (or long) reads without a traditional aligner, and you need to score k-mer hits against contigs and rescue weakly-placed mates before writing pair locations as SAM.

## When NOT to use

- Input is raw unlinked long reads where insert size matters: synthetic-library insert size is restricted to the actual long-read size, so gap-size estimation differs from 10X/DOVETAIL-style mate-pair data
- You need full base-level alignments with mismatches/CIGAR strings — Fast-SG is alignment-free and only reports pair locations, not alignments

## Inputs

- Contig/assembly FASTA sequences to index (e.g. example/ecoli-illumina.fa.gz), k-merized via KMC3
- Read files (gzipped FASTA/FASTQ) plus a read configuration file listing libraries and read type (1:single, 2:paired), e.g. examples/ecoli-reads.txt
- K-mer size (12-256 supported; example uses -k 15)

## Outputs

- SAM-format pair location file written under the -p output prefix
- Per-read hit vectors / windowed scores used internally for placement decisions

## How to apply

Build the unique k-mer index over the contig FASTA by counting k-mers with KMC3 and storing the unique set in a BBHash minimal perfect hash backed by the quasi_dictionary probabilistic membership structure (~87 bits/k-mer, k-mer size in the supported 12-256 range). Stream reads via kseqcpp and compute ntHash rolling hashes of every read k-mer to query the index, accumulating per-read hit vectors against the indexed contigs. Apply windowed scoring over the hit vectors to localize each read on the contigs; report a pair location only when the forward read scores above 3 and each read scores above 5, otherwise run the pair-rescue step using the documented Illumina thresholds (3/5/4) to retain mate pairs with consistent joint placement. Emit pair locations as a SAM-format file under the -p output prefix, using SAM flag 1 for single and 2 for paired reads as declared in the read configuration file (e.g. examples/ecoli-reads.txt).

## Related tools

- **Fast-SG** (Main tool implementing the alignment-free scoring and pair-rescue pipeline; run via FAST-SG.pl) — https://github.com/adigenova/fast-sg
- **ntHash** (Rolling hash function computed on every read k-mer to query the unique k-mer index) — https://github.com/bcgsc/ntHash
- **quasi_dictionary** (Probabilistic membership structure backing the unique k-mer dictionary (false-positive rate below 1/2^fingerprint_size)) — https://github.com/pierrepeterlongo/quasi_dictionary
- **BBHash** (Minimal perfect hash function mapping unique k-mers to positions in the index) — https://github.com/rizkg/BBHash
- **KMC** (Counts k-mers over the contig sequences to define the unique k-mer set) — https://github.com/refresh-bio/KMC
- **kseqcpp** (C++ FASTA/FASTQ parser streaming contigs and reads into the scoring pipeline) — https://github.com/ctSkennerton/kseqcpp

## Examples

```
./FAST-SG.pl -k 15 -l example/ecoli-reads.txt -r example/ecoli-illumina.fa.gz -p test
```

## Evaluation signals

- Compile with g++ (>=5.3 Linux / clang 4.2 OSX) and 'make test' completes without error
- The SAM file emitted under the -p prefix exists and parses as valid SAM
- SAM records appear for the paired libraries declared in ecoli-reads.txt with flag 2 (paired), not flag 1 (single)
- Pair locations are only reported when the forward-read score exceeds 3 and each read exceeds 5, or pairs were retained via the rescue step with thresholds 3/5/4
- Running over a k-mer range (-k 15-25:5) yields pair locations for each k value without index errors

## Limitations

- Score thresholds (3/5/4) are documented for Illumina mate-pair reads; rescue behavior for other library types is not specified
- quasi_dictionary is probabilistic, so queries can return false positives at a rate below 1/2^fingerprint_size
- Synthetic-library insert size is restricted to the actual long-read size, complicating gap-size estimation compared with 10X/DOVETAIL long-range mate-pair technologies

## Evidence

- [abstract] we propose a new method, called Fast-SG, that uses a new ultrafast alignment-free algorithm specifically designed for constructing a scaffolding graph using light-weight data structures: "we propose a new method, called Fast-SG, that uses a new ultrafast alignment-free algorithm specifically designed for constructing a scaffolding graph using light-weight data structures"
- [methods] Example of a read configuration file can be found in examples/ecoli-reads.txt: "Example of a read configuration file can be found in examples/ecoli-reads.txt"
- [methods] ntHash (https://github.com/bcgsc/ntHash.git checkout ff326a8c9ccf6186f42c1f49950c1ebaadbd7f7a): "ntHash (https://github.com/bcgsc/ntHash.git checkout ff326a8c9ccf6186f42c1f49950c1ebaadbd7f7a)"
- [readme] Fast-SG is an alignment-free algorithm for ultrafast scaffolding graph construction from short or long reads.: "Fast-SG is an alignment-free algorithm for ultrafast scaffolding graph construction from short or long reads."
