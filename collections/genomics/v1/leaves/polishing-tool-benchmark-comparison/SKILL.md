---
name: polishing-tool-benchmark-comparison
description: Use when raw long-read assemblies need polishing and a fair CONSENT-vs-RACON
  comparison of error-correction quality, wall-clock time, and memory footprint must
  be produced and tabulated.
when_to_use_negative:
- The goal is long-read self-correction of the reads themselves (CONSENT-correct),
  not polishing of a raw assembly
- Short reads are the polishing input; this skill is grounded in long-read (PacBio/ONT)
  polishing with CONSENT and RACON
- No reference genome is available for QUAST-LG evaluation, so quality metrics comparable
  to the article cannot be computed
edam_operation: http://edamontology.org/operation_0525
edam_topics:
- http://edamontology.org/topic_0196
- http://edamontology.org/topic_0622
tools:
- name: CONSENT
  role: Polishing tool under evaluation; run in polishing mode (CONSENT-polish) with
    default parameters on Miniasm raw assemblies
  repo: https://github.com/morispi/CONSENT
- name: RACON
  role: State-of-the-art polishing comparison tool run on the same raw assemblies
    under equivalent conditions
- name: Miniasm
  role: Produces the raw de novo assemblies that both polishers are applied to
- name: QUAST-LG
  role: Evaluates raw and polished assemblies against reference genomes to yield misassemblies,
    error counts, genome fraction, and NGAx metrics
- name: PacBio
  role: Source technology for simulated/real long-read datasets used in the polishing
    benchmark
- name: ONT
  role: Source technology for simulated/real long-read datasets used in the polishing
    benchmark
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41598-020-80757-5
    title: Scalable long read self-correction and assembly polishing with multiple
      sequence alignment
  source_content_hash: sha256:c791b867d9dc2eac6dac9177dacaf31911adce03f6a44a28ce4e4b2380717b05
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/morispi/CONSENT
  tools:
  - CONSENT
  - Miniasm
  - ONT
  - PacBio
  - QUAST-LG
  - RACON
  edam_operation: http://edamontology.org/operation_0525
  edam_topics:
  - http://edamontology.org/topic_0196
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 11
---

# polishing-tool-benchmark-comparison

## Summary

Benchmark the assembly-polishing feature of CONSENT (CONSENT-polish) against the state-of-the-art polisher RACON on simulated and real PacBio/ONT long-read datasets, comparing polishing quality (QUAST-LG metrics) alongside runtime and peak memory consumption.

## When to use

Use when raw long-read assemblies need polishing and a fair CONSENT-vs-RACON comparison of error-correction quality, wall-clock time, and memory footprint must be produced and tabulated.

## When NOT to use

- The goal is long-read self-correction of the reads themselves (CONSENT-correct), not polishing of a raw assembly
- Short reads are the polishing input; this skill is grounded in long-read (PacBio/ONT) polishing with CONSENT and RACON
- No reference genome is available for QUAST-LG evaluation, so quality metrics comparable to the article cannot be computed

## Inputs

- Raw de novo long-read assemblies (contigs.fasta) produced with Miniasm from PacBio or ONT reads
- Long-read datasets in fasta or fastq format (simulated and real PacBio/ONT, multiple organisms and coverage conditions)
- Reference genomes for QUAST-LG evaluation of raw and polished assemblies

## Outputs

- polishing_benchmark_metrics.tsv with per-dataset CONSENT-vs-RACON quality and resource metrics (misassemblies, error counts, genome fraction, NGAx, wall-clock time, cores, peak RAM)
- Polished assemblies from CONSENT-polish and RACON for each organism and coverage condition

## How to apply

First assemble each simulated and real long-read dataset (PacBio/ONT fasta/fastq inputs) de novo with Miniasm to obtain raw assemblies per organism and coverage condition. Run CONSENT-polish on every raw assembly with default parameters (windowSize 500, minSupport 4, maxSupport 150, merSize k=9, solid 4, anchorSupport 8, minAnchors 2, windowOverlap 50) and, in parallel, polish the same raw assemblies with RACON under equivalent conditions. Evaluate each raw and polished assembly against its reference genome with QUAST-LG to obtain misassemblies, error counts, genome fraction, and NGAx. Record wall-clock time, number of cores (4 where specified), and peak RAM per polishing run (e.g. ~15 sec / 150 MB for a CONSENT-polish run on the example dataset). Compile the per-dataset CONSENT-vs-RACON metrics into a TSV table; validation expects CONSENT to deliver error-correction quality comparable to RACON while being ~2x-38x faster (e.g. 30 sec vs 5 min 55 sec on E. coli 60x) and using up to ~4x less memory.

## Related tools

- **CONSENT** (Polishing tool under evaluation; run in polishing mode (CONSENT-polish) with default parameters on Miniasm raw assemblies) — https://github.com/morispi/CONSENT
- **RACON** (State-of-the-art polishing comparison tool run on the same raw assemblies under equivalent conditions)
- **Miniasm** (Produces the raw de novo assemblies that both polishers are applied to)
- **QUAST-LG** (Evaluates raw and polished assemblies against reference genomes to yield misassemblies, error counts, genome fraction, and NGAx metrics)
- **PacBio** (Source technology for simulated/real long-read datasets used in the polishing benchmark)
- **ONT** (Source technology for simulated/real long-read datasets used in the polishing benchmark)

## Examples

```
./CONSENT-polish --contigs example/rawAssembly.fasta --reads example/reads.fasta --out example/polishedAssembly.fasta
```

## Evaluation signals

- QUAST-LG polished-assembly quality metrics (misassemblies, error counts, genome fraction, NGAx) for CONSENT are highly comparable to RACON on every tested dataset
- Recorded wall-clock times show CONSENT-polish faster than RACON across all datasets, spanning roughly 2x to 38x (e.g. 30 sec vs 5 min 55 sec on E. coli 60x)
- Peak RAM of CONSENT-polish runs is up to about 4x lower than RACON's (example run: ~15 sec, at most 150 MB RAM on 4 cores)
- The compiled polishing_benchmark_metrics.tsv covers each organism and coverage condition and matches the article's reported polishing table within expected variation

## Limitations

- Runtime of CONSENT's correction step tends to be higher than other state-of-the-art methods (the speed advantage cited applies to polishing)
- CONSENT's assembly covers a smaller proportion of the reference sequence; unaligned contigs were not analyzed in detail
- Only assembly polishing was benchmarked against RACON; published correction tools were not yet evaluated for their ability to polish assemblies
- CONSENT is developed and tested on x86-64 GNU/Linux; other platforms are untested

## Evidence

- [abstract] CONSENT is 2-38x times faster than other polishing tools, while providing comparable results.: "CONSENT is 2-38x times faster than other polishing tools, while providing comparable results."
- [abstract] CONSENT implements a polishing feature, allowing to correct raw assemblies.: "CONSENT implements a polishing feature, allowing to correct raw assemblies."
- [readme] This should take about 15 sec and use at most 150 MB of RAM, using 4 cores.: "This should take about 15 sec and use at most 150 MB of RAM, using 4 cores."
- [methods] --minSupport INT, -s INT:      Minimum support to consider a window for correction. (default: 4): "--minSupport INT, -s INT:      Minimum support to consider a window for correction. (default: 4)"
- [intro] Additionally, CONSENT is also able to polish assemblies generated from raw long reads.: "Additionally, CONSENT is also able to polish assemblies generated from raw long reads."
- [discussion] it would also be interesting to evaluate already published correction tools on their ability to polish assemblies: "it would also be interesting to evaluate already published correction tools on their ability to polish assemblies"
