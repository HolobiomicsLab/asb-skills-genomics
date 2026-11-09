---
name: scaffolding-error-classification
description: 'Use when comparing scaffolded assemblies (e.g. from Opera-LG, ScaffMatch,
  Boss or Links) and you need join-level quality metrics: correct/erroneous join counts,
  error-type breakdowns, and per-scaffolder F-scores.'
when_to_use_negative:
- Input is a single unassembled contig set with no joins to evaluate — there is no
  scaffolding to classify
- The goal is k-mer counting or synthetic-library extraction (Fast-SG/KMC territory)
  rather than assessing scaffold quality
- You lack the information needed to determine which joins are correct, so F-scores
  cannot be computed
tools:
- name: Fast-SG
  role: Produces the synthetic mate-pair libraries from corrected or uncorrected long
    reads that feed the short-read scaffolders being benchmarked
  repo: https://github.com/adigenova/fast-sg
- name: Links
  role: Long-read scaffolder used as the baseline whose joins and errors are classified
    for comparison against short-read scaffolders
- name: Opera-LG
  role: Short-read scaffolder evaluated on Fast-SG synthetic libraries in the benchmark
- name: ScaffMatch
  role: Short-read scaffolder evaluated on Fast-SG synthetic libraries in the benchmark
- name: Boss
  role: Short-read scaffolder evaluated on Fast-SG synthetic libraries in the benchmark
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/gigascience/giy048
    title: 'Fast-SG: an alignment-free algorithm for hybrid assembly'
  source_content_hash: sha256:3c51c1e8cd7f8e836cfa92c5b19568d66db56e11a7f25235a1b1cb0b79f9c2c1
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/adigenova/fast-sg
  tools:
  - Boss
  - C++
  - Fast-SG
  - Links
  - Opera-LG
  - PERL
  - ScaffMatch
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# scaffolding-error-classification

## Summary

Benchmark-style evaluation of scaffolded assemblies by counting correct versus erroneous joins, classifying scaffolding error types (e.g. misjoins, rearrangements), and computing F-scores per dataset and scaffolder. In the Fast-SG study this is used to compare short-read scaffolders fed with Fast-SG synthetic mate-pair libraries against the long-read scaffolder Links.

## When to use

Use when comparing scaffolded assemblies (e.g. from Opera-LG, ScaffMatch, Boss or Links) and you need join-level quality metrics: correct/erroneous join counts, error-type breakdowns, and per-scaffolder F-scores.

## When NOT to use

- Input is a single unassembled contig set with no joins to evaluate — there is no scaffolding to classify
- The goal is k-mer counting or synthetic-library extraction (Fast-SG/KMC territory) rather than assessing scaffold quality
- You lack the information needed to determine which joins are correct, so F-scores cannot be computed

## Inputs

- Scaffolded assemblies (one per dataset/scaffolder combination) as FASTA
- Fast-SG synthetic mate-pair libraries extracted from uncorrected long reads (e.g. E. coli K12 50X, S. cerevisiae W303 30X)
- Original long-read scaffolds from Links as the long-read baseline
- Reference/dataset identity information needed to classify joins as correct or erroneous

## Outputs

- Per-scaffolder counts of correct joins and erroneous joins per dataset
- Scaffolding error-type classification (e.g. misjoins, rearrangements) per scaffolded assembly
- F-score per dataset and scaffolder
- Benchmark table (e.g. scaffolding_benchmark_results.tsv)

## How to apply

For each scaffolded assembly (e.g. E. coli K12 and S. cerevisiae W303 Illumina assemblies scaffolded either with Opera-LG, ScaffMatch, and Boss on Fast-SG synthetic mate-pair libraries, or with Links on the original long reads), count all joins and classify each as correct or erroneous, breaking errors into types such as misjoins and rearrangements. From these join counts and error classifications compute an F-score per dataset and scaffolder, which combines precision (avoiding erroneous joins) and recall (recovering correct joins). Compile the per-dataset, per-scaffolder results into a benchmark table (e.g. scaffolding_benchmark_results.tsv) so scaffolders can be ranked; in this study a scaffolder was judged better when its F-score was higher, and Links produced more scaffolding errors in two of the three tested datasets.

## Related tools

- **Fast-SG** (Produces the synthetic mate-pair libraries from corrected or uncorrected long reads that feed the short-read scaffolders being benchmarked) — https://github.com/adigenova/fast-sg
- **Links** (Long-read scaffolder used as the baseline whose joins and errors are classified for comparison against short-read scaffolders)
- **Opera-LG** (Short-read scaffolder evaluated on Fast-SG synthetic libraries in the benchmark)
- **ScaffMatch** (Short-read scaffolder evaluated on Fast-SG synthetic libraries in the benchmark)
- **Boss** (Short-read scaffolder evaluated on Fast-SG synthetic libraries in the benchmark)

## Examples

```
./FAST-SG.pl -k 15 -l example/ecoli-reads.txt -r example/ecoli-illumina.fa.gz -p test
```

## Evaluation signals

- Every join in every scaffolded assembly has been counted and assigned a class (correct vs erroneous) with an error type for erroneous joins
- An F-score is reported for every dataset × scaffolder combination (E. coli K12 and S. cerevisiae W303 × Opera-LG, ScaffMatch, Boss, Links)
- The benchmark table is populated and internally consistent: F-scores recompute correctly from the stored join counts and error classifications
- Ranking matches the study expectation that Fast-SG-fed short-read scaffolders are superior to or match Links, with Links producing more errors in two of three datasets
- Scaffolded assemblies used for counting are traceable to a specific library (synthetic Fast-SG vs original long reads)

## Limitations

- Comparison depends on the scaffolder actually receiving the intended input: synthetic libraries are restricted to the actual long-read size (insert size is restricted to the actual long-read size)
- F-score summarization can mask the mix of error types; Links produced more scaffolding errors only in two of the three tested datasets, so per-dataset inspection is required
- No version number stated for Fast-SG itself (only RRID SCR_015934), so reproducibility of the exact benchmarked software may be ambiguous

## Evidence

- [discussion] results that are superior to or match those obtained by Links, a scaffolder specifically designed for hybrid long-read scaffolding: "results that are superior to or match those obtained by Links, a scaffolder specifically designed for hybrid long-read scaffolding"
- [discussion] short-read scaffolders produce accurate scaffolds from Fast-SG synthetic libraries: "short-read scaffolders are able to produce accurate scaffolds when they are fed with the synthetic libraries extracted by Fast-SG"
- [readme] Fast-SG is an alignment-free algorithm for ultrafast scaffolding graph construction from short or long reads.: "Fast-SG is an alignment-free algorithm for ultrafast scaffolding graph construction from short or long reads."
- [methods] KMC dependency and binaries placement for the Fast-SG pipeline underlying the synthetic libraries: "After getting KMC3, put the binaries inside Fast-SG directory"
