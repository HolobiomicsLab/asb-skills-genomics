---
name: hybrid-assembly-benchmark-tabulation
description: Use when comparing scaffolding quality of Fast-SG synthetic mate-pair
  libraries (from raw vs error-corrected ONT/PacBio reads) against a long-read baseline
  scaffolder such as Links, and a metrics table must be produced.
when_to_use_negative:
- When the goal is scaffolding long reads directly with a purpose-built long-read
  scaffolder (e.g. Links) rather than benchmarking Fast-SG synthetic libraries against
  it
- When insert sizes required for the synthetic library exceed the actual long-read
  length — the article notes the insert size is restricted to the actual long-read
  size
- When input reads are short Illumina reads rather than long ONT/PacBio reads plus
  a short-read assembly to scaffold
edam_operation: http://edamontology.org/operation_3211
edam_topics:
- http://edamontology.org/topic_0102
- http://edamontology.org/topic_0196
tools:
- name: Fast-SG
  role: Alignment-free builder of the scaffolding graph; extracts synthetic mate-pair
    libraries from raw or corrected long reads via FAST-SG.pl
  repo: https://github.com/adigenova/fast-sg
- name: KMC
  role: k-mer counter whose kmc/kmc_dump binaries are placed in $Fast-SG/KMC/bin to
    support k-mer recall computation
  repo: https://github.com/refresh-bio/KMC
- name: Links
  role: Long-read baseline scaffolder used for direct comparison of scaffolding quality
- name: Opera-LG
  role: Short-read scaffolder fed with Fast-SG synthetic mate-pair libraries
- name: ScaffMatch
  role: Short-read scaffolder fed with Fast-SG synthetic mate-pair libraries
- name: Boss
  role: Short-read scaffolder fed with Fast-SG synthetic mate-pair libraries
provenance:
  source_task_ids:
  - task_001
  - task_002
  source_papers:
  - doi: 10.1093/gigascience/giy048
    title: 'Fast-SG: an alignment-free algorithm for hybrid assembly'
  source_content_hash: sha256:08fdfc686f542d967a4dae201d5f5c5854a0cbadacba05685cba347e9445002c
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
  edam_operation: http://edamontology.org/operation_3211
  edam_topics:
  - http://edamontology.org/topic_0102
  - http://edamontology.org/topic_0196
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# hybrid-assembly-benchmark-tabulation

## Summary

A benchmarking and tabulation procedure for Fast-SG hybrid assembly experiments: run the alignment-free scaffolding-graph builder on ONT/PacBio long reads against Illumina contigs, scaffold with short-read scaffolders, then tabulate k-mer recall, synthetic mate-pair read recall, outlier percentages, join counts and F-scores per technology/correction-state combination.

## When to use

Use when comparing scaffolding quality of Fast-SG synthetic mate-pair libraries (from raw vs error-corrected ONT/PacBio reads) against a long-read baseline scaffolder such as Links, and a metrics table must be produced.

## When NOT to use

- When the goal is scaffolding long reads directly with a purpose-built long-read scaffolder (e.g. Links) rather than benchmarking Fast-SG synthetic libraries against it
- When insert sizes required for the synthetic library exceed the actual long-read length — the article notes the insert size is restricted to the actual long-read size
- When input reads are short Illumina reads rather than long ONT/PacBio reads plus a short-read assembly to scaffold

## Inputs

- Raw ONT R9.2 and PacBio Sequel long-read FASTA/FASTA.gz files (e.g. E. coli K12 50X, S. cerevisiae W303 30X)
- Lordec hybrid error-corrected long-read libraries
- Read configuration file in space-separated long-read format (#type libID Path Insert-sizes SAM), e.g. 'long pac <reads.fasta.gz> 1000,2000,3000,5000 1'
- Illumina assembly contigs in FASTA/FASTA.gz (E. coli K12 and S. cerevisiae W303 reference assemblies)

## Outputs

- Fast-SG synthetic mate-pair libraries extracted from long reads
- Scaffolded assemblies per dataset and scaffolder (Opera-LG, ScaffMatch, Boss, Links)
- ecoli_k12_recall_metrics.tsv: k-mer recall, synthetic mate-pair read recall, and outlier percentage per technology/correction-state row
- scaffolding_benchmark_results.tsv: join counts, error classifications, and F-scores per dataset and scaffolder

## How to apply

Author a space-separated read configuration file (#type libID Path(long read) Insert-sizes SAM) listing raw and Lordec-corrected libraries with insert sizes spanning 0.5-8 kb. Run FAST-SG.pl with k=15 (-k), the configuration file (-l), the Illumina assembly contigs in FASTA/FASTA.gz (-r), and an output prefix (-p) to build the scaffolding graph and extract synthetic mate-pair libraries. Feed the synthetic libraries to the short-read scaffolders Opera-LG, ScaffMatch, and Boss, and scaffold the original long reads directly with Links as the long-read baseline. Count correct/erroneous joins, classify error types (misjoins, rearrangements), compute F-scores per dataset and scaffolder, and tabulate k-mer recall, mate-pair read recall, and outlier percentage per technology (ONT R9.2 vs PacBio Sequel) and correction state into a TSV such as ecoli_k12_recall_metrics.tsv or scaffolding_benchmark_results.tsv.

## Related tools

- **Fast-SG** (Alignment-free builder of the scaffolding graph; extracts synthetic mate-pair libraries from raw or corrected long reads via FAST-SG.pl) — https://github.com/adigenova/fast-sg
- **KMC** (k-mer counter whose kmc/kmc_dump binaries are placed in $Fast-SG/KMC/bin to support k-mer recall computation) — https://github.com/refresh-bio/KMC
- **Links** (Long-read baseline scaffolder used for direct comparison of scaffolding quality)
- **Opera-LG** (Short-read scaffolder fed with Fast-SG synthetic mate-pair libraries)
- **ScaffMatch** (Short-read scaffolder fed with Fast-SG synthetic mate-pair libraries)
- **Boss** (Short-read scaffolder fed with Fast-SG synthetic mate-pair libraries)

## Examples

```
./FAST-SG.pl -k 15 -l example/ecoli-reads.txt -r example/ecoli-illumina.fa.gz -p test
```

## Evaluation signals

- The output table must contain a metrics row for each of the four technology/correction-state combinations (ONT R9.2 vs PacBio Sequel, raw vs Lordec-corrected)
- All recall and outlier values must parse as percentages in the range [0, 100]
- Reference numbers: synthetic mate-pair read recall of 49.42% (raw ONT) and 31.65% (raw PacBio), rising to 75.12% (corrected ONT) and 65.02% (corrected PacBio) after hybrid error correction
- F-scores must be recomputable from the recorded join counts and error classifications per dataset and scaffolder
- Fast-SG compilation passes 'make all' and 'make test' with g++ 5.3 (Linux) or clang 4.2 (Mac), with kmc and kmc_dump present in $Fast-SG/KMC/bin

## Limitations

- The synthetic library insert size is restricted to the actual long-read size
- Gap size estimation with existing long-range mate-pair technologies (10X genomics and DOVETAIL genomics) is challenging because linking information comes from a range of insert sizes and read-pair orientation may be unknown
- No version number is stated for the Fast-SG software itself (only RRID SCR_015934), and no changelog is available for the repository

## Evidence

- [discussion] short-read scaffolders are able to produce accurate scaffolds when they are fed with the synthetic libraries extracted by Fast-SG, thus leading to results that are superior to or match those obtained: "short-read scaffolders are able to produce accurate scaffolds when they are fed with the synthetic libraries extracted by Fast-SG, thus leading to results that are superior to or match those obtained"
- [abstract] This allows the reuse of efficient algorithms designed for short-read data and permits the definition of novel modular hybrid assembly pipelines: "This allows the reuse of efficient algorithms designed for short-read data and permits the definition of novel modular hybrid assembly pipelines"
- [methods] ./FAST-SG.pl  -k 15 -l example/ecoli-reads.txt -r example/ecoli-illumina.fa.gz -p test: "./FAST-SG.pl  -k 15 -l example/ecoli-reads.txt -r example/ecoli-illumina.fa.gz -p test"
- [readme] Fast-SG is an alignment-free algorithm for ultrafast scaffolding graph construction from short or long reads.: "Fast-SG is an alignment-free algorithm for ultrafast scaffolding graph construction from short or long reads."
- [discussion] The insert size is restricted to the actual long-read size.: "The insert size is restricted to the actual long-read size."
