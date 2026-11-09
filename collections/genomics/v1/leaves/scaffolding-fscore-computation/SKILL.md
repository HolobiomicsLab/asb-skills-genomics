---
name: scaffolding-fscore-computation
description: Use when evaluating whether a scaffolder (e.g. a short-read scaffolder
  fed with synthetic libraries) matches a long-read baseline like Links, by counting
  joins, classifying error types, and computing per-dataset F-scores.
when_to_use_negative:
- Input is already a finished benchmark table of F-scores — the skill is for generating
  those counts and scores, not consuming them
- The question is about scaffolding-graph construction speed or synthetic-library
  insert-size distribution rather than scaffolding join correctness
- Genomes or long-read data outside the scope tested here (e.g. without a comparable
  long-read baseline scaffolded by Links) where join/error classification against
  a reference cannot be performed
edam_operation: http://edamontology.org/operation_0310
edam_topics:
- http://edamontology.org/topic_0102
- http://edamontology.org/topic_0196
tools:
- name: Fast-SG
  role: Extracts synthetic mate-pair libraries from uncorrected long reads that are
    then scaffolded by the short-read scaffolders; run via FAST-SG.pl with k in [12-256]
  repo: https://github.com/adigenova/fast-sg
- name: Links
  role: Long-read baseline scaffolder run on the original uncorrected long reads for
    comparison
- name: Opera-LG
  role: Short-read scaffolder run on Fast-SG synthetic libraries as one arm of the
    benchmark
- name: ScaffMatch
  role: Short-read scaffolder run on Fast-SG synthetic libraries as one arm of the
    benchmark
- name: Boss
  role: Short-read scaffolder run on Fast-SG synthetic libraries as one arm of the
    benchmark
- name: KMC
  role: k-mer counter whose binaries (kmc, kmc_dump) are placed in $Fast-SG/KMC/bin
    to support Fast-SG alignment-free graph construction
  repo: https://github.com/refresh-bio/KMC
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/gigascience/giy048
    title: 'Fast-SG: an alignment-free algorithm for hybrid assembly'
  source_content_hash: sha256:771e94b344cbd9eb120357a54ead1cca5c58764bd09d415df245560249448946
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/adigenova/fast-sg
  tools:
  - Boss
  - C++
  - Fast-SG
  - KMC
  - Links
  - Opera-LG
  - PERL
  - ScaffMatch
  edam_operation: http://edamontology.org/operation_0310
  edam_topics:
  - http://edamontology.org/topic_0102
  - http://edamontology.org/topic_0196
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# scaffolding-fscore-computation

## Summary

Benchmark protocol for comparing scaffolding quality across short-read and long-read scaffolders: count correct joins, classify erroneous joins (misjoins, rearrangements), and compute F-scores per dataset, as done when Fast-SG synthetic mate-pair libraries scaffolded by Opera-LG/ScaffMatch/Boss are compared against Links on E. coli K12 and S. cerevisiae W303.

## When to use

Use when evaluating whether a scaffolder (e.g. a short-read scaffolder fed with synthetic libraries) matches a long-read baseline like Links, by counting joins, classifying error types, and computing per-dataset F-scores.

## When NOT to use

- Input is already a finished benchmark table of F-scores — the skill is for generating those counts and scores, not consuming them
- The question is about scaffolding-graph construction speed or synthetic-library insert-size distribution rather than scaffolding join correctness
- Genomes or long-read data outside the scope tested here (e.g. without a comparable long-read baseline scaffolded by Links) where join/error classification against a reference cannot be performed

## Inputs

- Uncorrected long-read libraries (e.g. PacBio/Nanopore FASTA/FASTQ for E. coli K12 at 50X and S. cerevisiae W303 at 30X) plus a Fast-SG read configuration file listing them
- Illumina assemblies of the target genomes supplied as contigs (e.g. contigs.fa.gz)
- Synthetic mate-pair libraries extracted by Fast-SG from the long reads
- Scaffolded assemblies produced by Opera-LG, ScaffMatch, Boss (on synthetic libraries) and Links (on original long reads)

## Outputs

- Per-dataset, per-scaffolder join counts and classified scaffolding error types (misjoins, rearrangements)
- F-score values per dataset and scaffolder
- Compiled benchmark table, e.g. scaffolding_benchmark_results.tsv

## How to apply

Extract synthetic mate-pair libraries from uncorrected long reads with Fast-SG (run FAST-SG.pl with k in [12-256], a read configuration file such as ecoli-reads.txt, and the Illumina assembly as contigs), then scaffold those libraries with the short-read scaffolders Opera-LG, ScaffMatch, and Boss. As a baseline, scaffold the original uncorrected long reads against the same assemblies with Links. For every scaffolded assembly, count joins and classify scaffolding error types (misjoins, rearrangements) per dataset and scaffolder. Finally compute an F-score per dataset and scaffolder from the join counts and error classifications, and compile the results into a benchmark table (e.g. scaffolding_benchmark_results.tsv). The F-score balances correct joins against erroneous joins, so the scaffolder with the higher F-score — here the Fast-SG-fed short-read scaffolders relative to Links — is judged to produce superior or comparable scaffolding.

## Related tools

- **Fast-SG** (Extracts synthetic mate-pair libraries from uncorrected long reads that are then scaffolded by the short-read scaffolders; run via FAST-SG.pl with k in [12-256]) — https://github.com/adigenova/fast-sg
- **Links** (Long-read baseline scaffolder run on the original uncorrected long reads for comparison)
- **Opera-LG** (Short-read scaffolder run on Fast-SG synthetic libraries as one arm of the benchmark)
- **ScaffMatch** (Short-read scaffolder run on Fast-SG synthetic libraries as one arm of the benchmark)
- **Boss** (Short-read scaffolder run on Fast-SG synthetic libraries as one arm of the benchmark)
- **KMC** (k-mer counter whose binaries (kmc, kmc_dump) are placed in $Fast-SG/KMC/bin to support Fast-SG alignment-free graph construction) — https://github.com/refresh-bio/KMC

## Examples

```
./FAST-SG.pl -k 15 -l example/ecoli-reads.txt -r example/ecoli-illumina.fa.gz -p test
```

## Evaluation signals

- Every scaffolded assembly has join counts and error classes (misjoins, rearrangements) recorded per dataset and scaffolder before F-score computation
- F-scores are computed per dataset (E. coli K12 and S. cerevisiae W303) and per scaffolder, and compiled into scaffolding_benchmark_results.tsv
- Expected qualitative outcome: Fast-SG-fed short-read scaffolders reach better or comparable F-scores than Links, with Links producing more scaffolding errors in two of the three tested datasets
- Fast-SG runs complete with 'make test' passing and kmc/kmc_dump present in $Fast-SG/KMC/bin, confirming the synthetic libraries were generated correctly

## Limitations

- Synthetic library insert size is restricted to the actual long-read size, which bounds the linking distance Fast-SG can provide
- The synthetic libraries come from a single fixed insert size per read, unlike 10X Genomics or DOVETAIL mate-pair data where gap-size estimation is harder because linking information spans a range of insert sizes and read-pair orientation may be unknown
- Benchmark conclusions were drawn on E. coli K12 and S. cerevisiae W303 only; no version number is stated for the Fast-SG software itself (only RRID SCR_015934)

## Evidence

- [discussion] results that are superior to or match those obtained by Links, a scaffolder specifically designed for hybrid long-read scaffolding: "results that are superior to or match those obtained by Links, a scaffolder specifically designed for hybrid long-read scaffolding"
- [discussion] Long read scaffolding benchmark results for E. coli K12 and S. cerevisiae W303: "Long read scaffolding benchmark results for E. coli K12 and S. cerevisiae W303"
- [methods] Compilation was tested with g++ version 5.3 (Linux) and clang version 4.2 (Mac OSX): "Compilation was tested with g++ version 5.3 (Linux) and clang version 4.2 (Mac OSX)"
- [discussion] The insert size is restricted to the actual long-read size.: "The insert size is restricted to the actual long-read size."
- [methods] Example of a read configuration file can be found in examples/ecoli-reads.txt: "Example of a read configuration file can be found in examples/ecoli-reads.txt"
