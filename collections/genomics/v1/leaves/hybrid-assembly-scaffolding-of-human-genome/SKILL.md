---
name: hybrid-assembly-scaffolding-of-human-genome
description: Use when scaffolding a large (e.g. human) genome from an Illumina-based
  contig assembly plus shallow (~5X) uncorrected long Nanopore reads, and the goal
  is a fast, low-cost modular hybrid assembly instead of a full long-read assembly
  pipeline.
when_to_use_negative:
- When high-coverage corrected long reads are available and a full long-read assembler
  (rather than a modular hybrid pipeline) is appropriate.
- When the input long reads are too short to support the desired insert sizes — synthetic
  library insert size is restricted to the actual long-read size.
- When the target genome lacks a reliable contig assembly to scaffold; Fast-SG scaffolds
  existing contigs, it does not assemble the genome de novo.
edam_operation: http://edamontology.org/operation_0525
edam_topics:
- http://edamontology.org/topic_0196
- http://edamontology.org/topic_0622
tools:
- name: Fast-SG
  role: Alignment-free extraction of synthetic mate-pair libraries from uncorrected
    Nanopore reads to build the scaffolding graph
  repo: https://github.com/adigenova/fast-sg
- name: ScaffMatch
  role: Short-read scaffolder run on the Fast-SG synthetic libraries to produce the
    DFS scaffolds
- name: KMC
  role: K-mer counter whose binaries must be installed in $Fast-SG/KMC/bin/ for Fast-SG
    to build
  repo: https://github.com/refresh-bio/KMC
- name: PERL
  role: Runs the FAST-SG.pl wrapper script
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/gigascience/giy048
    title: 'Fast-SG: an alignment-free algorithm for hybrid assembly'
  source_content_hash: sha256:0550cf0d418ada711423b013162f16f3d77a4cced12c31f2c1474c0295fbed83
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/adigenova/fast-sg
  tools:
  - Fast-SG
  - KMC
  - PERL
  - ScaffMatch
  edam_operation: http://edamontology.org/operation_0525
  edam_topics:
  - http://edamontology.org/topic_0196
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7692307692307693
n_claims: 13
---

# hybrid-assembly-scaffolding-of-human-genome

## Summary

Reproduces the DFS hybrid assembly (DiscovarDeNovo + Fast-SG + ScaffMatch) of the human NA12878 genome, using Fast-SG to extract alignment-free synthetic mate-pair libraries from shallow (~5X) uncorrected Nanopore reads and ScaffMatch to scaffold the DiscovarDeNovo contigs into long-range scaffolds.

## When to use

Use when scaffolding a large (e.g. human) genome from an Illumina-based contig assembly plus shallow (~5X) uncorrected long Nanopore reads, and the goal is a fast, low-cost modular hybrid assembly instead of a full long-read assembly pipeline.

## When NOT to use

- When high-coverage corrected long reads are available and a full long-read assembler (rather than a modular hybrid pipeline) is appropriate.
- When the input long reads are too short to support the desired insert sizes — synthetic library insert size is restricted to the actual long-read size.
- When the target genome lacks a reliable contig assembly to scaffold; Fast-SG scaffolds existing contigs, it does not assemble the genome de novo.

## Inputs

- DiscovarDeNovo contig assembly of NA12878 (FASTA)
- Uncorrected ultra-long Oxford Nanopore reads (~5X coverage, FASTA)
- Long-read configuration file listing each Nanopore library (type 'long', library ID, FASTA path, comma-separated insert sizes, single-end flag)

## Outputs

- DFS scaffold sequences for NA12878 (scaffold FASTA from ScaffMatch)
- library_stats.tsv recording pair counts, linking percentage, and outlier percentage per synthetic library
- Approximately 20 synthetic mate-pair libraries with insert sizes up to 180 kb

## How to apply

Obtain the NA12878 hybrid-scaffolding inputs referenced by the Fast-SG wiki (DiscovarDeNovo contig assembly and ultra-long Nanopore read set), then build Fast-SG from source after placing KMC3 binaries in $Fast-SG/KMC/bin/ and validating with 'make test'. Write a long-read configuration file where each Nanopore library is a space-separated line with type 'long', a library ID, the FASTA path, comma-separated insert sizes, and the single-end flag, mirroring examples/ecoli-reads.txt. Run FAST-SG.pl with k-mer size 22, the read config (-l), the NA12878 contigs (-r), and an output prefix (-p), repeating the extraction across insert sizes up to BAC-like 180 kb to obtain ~20 synthetic libraries — remembering that insert size is bounded by the actual long-read length. Feed the extracted synthetic libraries to ScaffMatch, which scaffolds the DiscovarDeNovo contigs into the final scaffold FASTA.

## Related tools

- **Fast-SG** (Alignment-free extraction of synthetic mate-pair libraries from uncorrected Nanopore reads to build the scaffolding graph) — https://github.com/adigenova/fast-sg
- **ScaffMatch** (Short-read scaffolder run on the Fast-SG synthetic libraries to produce the DFS scaffolds)
- **KMC** (K-mer counter whose binaries must be installed in $Fast-SG/KMC/bin/ for Fast-SG to build) — https://github.com/refresh-bio/KMC
- **PERL** (Runs the FAST-SG.pl wrapper script)

## Examples

```
./FAST-SG.pl -k 22 -l reads.txt -r na12878-contigs.fa -p na12878_hybrid
```

## Evaluation signals

- Exactly ~20 synthetic libraries are extracted, with insert sizes spanning up to 180 kb (BAC-like sizes).
- library_stats.tsv exists and records pair counts, linking percentage, and outlier percentage for every library.
- The final scaffold FASTA file exists and is valid FASTA.
- 'make test' passes after compiling Fast-SG, confirming the build (with KMC binaries in $Fast-SG/KMC/bin/).
- Scaffold accuracy/long-range continuity matches the reported DFS result: accurate, long-range reconstruction of the diploid human genome at shallow (~5X) Nanopore coverage.

## Limitations

- Synthetic library insert size is restricted to the actual long-read size, so very short Nanopore reads cap the achievable insert sizes.
- Requires shallow but ultra-long Nanopore coverage; results depend on the ~5X long-read regime used in the paper.
- The pipeline depends on a pre-built DiscovarDeNovo contig assembly; it scaffolds rather than assembles.
- Fast-SG compilation is sensitive to toolchain (tested with g++ 5.3 on Linux and clang 4.2 on Mac OSX) and requires KMC3 binaries placed in $Fast-SG/KMC/bin/.

## Evidence

- [abstract] hybrid assembly approach using Fast-SG with shallow long-read coverage (5X): "a hybrid assembly approach using Fast-SG with shallow long-read coverage (5X) and moderate computational resources can produce long-range and accurate reconstructions"
- [methods] wiki page full example: "The [wiki page](https://github.com/adigenova/fast-sg/wiki/Hybrid-scaffolding-of-NA12878) provide a full example of the use of Fast-SG for the hybrid assembly of NA12878."
- [methods] example read configuration file: "Example of a read configuration file can be found in examples/ecoli-reads.txt"
- [methods] FAST-SG.pl invocation: "./FAST-SG.pl  -k 15 -l example/ecoli-reads.txt -r example/ecoli-illumina.fa.gz -p test"
- [discussion] BAC-like synthetic libraries from ultralong Nanopore: "using ultralong Nanopore reads,Fast-SG is able to extract synthetic libraries of even bacterial artificial chromosome clone sizes"
- [discussion] insert size restricted to read length: "The insert size is restricted to the actual long-read size."
- [discussion] short-read scaffolders fed with synthetic libraries: "short-read scaffolders are able to produce accurate scaffolds when they are fed with the synthetic libraries extracted by Fast-SG"
- [readme] KMC binaries placement: "## Get KMC (k-mer counter)"
