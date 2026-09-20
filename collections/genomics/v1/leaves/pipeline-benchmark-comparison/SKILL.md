---
name: pipeline-benchmark-comparison
description: Decision support for choosing between correct-then-assemble and assemble-then-polish
  on ultra-long ONT read sets, when runtime, RAM, and assembly metrics (NGA50, errors
  per 100 kbp) must be compared on identical inputs.
when_to_use_negative:
- Input is short-read or Illumina data — CONSENT is a self-correction/polishing method
  for long PacBio/ONT reads
- The dataset is small enough that both pipelines run quickly anyway — the resource
  trade-off this benchmark quantifies (days of runtime, TB of RAM) only becomes decisive
  at genome/human scale with ultra-long reads
- You only need assembly quality, not a resource-vs-quality trade-off analysis — a
  single pipeline run suffices
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_0196
- http://edamontology.org/topic_0102
tools:
- name: CONSENT
  role: 'Provides both arms of the benchmark: CONSENT-correct (--type ONT) corrects
    reads before assembly, and CONSENT-polish corrects the raw Miniasm assembly in
    the alternative arm; its runtime and peak RAM are the measured resource quantities'
  repo: https://github.com/morispi/CONSENT
- name: Flye
  role: Assembles the CONSENT-corrected reads to produce the correct-then-assemble
    assembly in the first arm of the comparison
- name: Miniasm
  role: Produces the raw assembly (rawAssembly.fasta) that is subsequently polished
    with CONSENT-polish in the second arm
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41598-020-80757-5
    title: Scalable long read self-correction and assembly polishing with multiple
      sequence alignment
  source_content_hash: sha256:a3da93b15bd236a68dc1f490869ba493ad1f76ff9cf9f87fceb9df05d2f86511
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/morispi/CONSENT
  tools:
  - CONSENT
  - Flye
  - Miniasm
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_0196
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 6
---

# pipeline-benchmark-comparison

## Summary

Benchmarks two long-read pipelines on the same ultra-long ONT dataset — correcting reads with CONSENT-correct then assembling with Flye, versus assembling raw reads with Miniasm then polishing the assembly with CONSENT-polish — comparing both resource usage and assembly quality to decide which strategy to deploy at human scale.

## When to use

Decision support for choosing between correct-then-assemble and assemble-then-polish on ultra-long ONT read sets, when runtime, RAM, and assembly metrics (NGA50, errors per 100 kbp) must be compared on identical inputs.

## When NOT to use

- Input is short-read or Illumina data — CONSENT is a self-correction/polishing method for long PacBio/ONT reads
- The dataset is small enough that both pipelines run quickly anyway — the resource trade-off this benchmark quantifies (days of runtime, TB of RAM) only becomes decisive at genome/human scale with ultra-long reads
- You only need assembly quality, not a resource-vs-quality trade-off analysis — a single pipeline run suffices

## Inputs

- Raw ultra-long Oxford Nanopore reads (FASTA/FASTQ, reads up to ~1.5 Mbp)
- Reference genome for reference-based assembly evaluation
- Raw de novo assembly of the long reads (e.g. Miniasm contigs, FASTA)
- Corrected long reads (CONSENT-correct output, result.fasta)
- Assembly from corrected reads (e.g. Flye contigs, FASTA)

## Outputs

- Corrected long reads (result.fasta) from the correct-then-assemble arm
- Polished assembly (polishedAssembly.fasta) from the assemble-then-polish arm
- Side-by-side comparison table (e.g. consent_pipeline_comparison.tsv) with NGA50, NGA75, genome coverage, errors per 100 kbp, misassemblies, runtime, and peak RAM per pipeline

## How to apply

Run both pipelines on the same raw long-read file: (1) CONSENT-correct with --type ONT to produce corrected reads (result.fasta), then assemble with Flye; (2) assemble raw reads with Miniasm, then run CONSENT-polish on the raw assembly. Record wall-clock runtime and peak RAM for every step, then compute reference-based metrics for both assemblies: NGA50, NGA75, genome coverage, errors per 100 kbp, and misassembly counts. Tabulate quality and resources side by side (e.g. consent_pipeline_comparison.tsv) and check that the trade-off is judged on both axes simultaneously — in the paper's full human dataset the correction + assembly pipeline needed >17 days and 1 TB RAM while assembly + polishing finished in 7.5 days using <400 GB, with better assembly metrics. The rationale is that CONSENT's k-mer-chained MSA polishing acts on the assembly directly, avoiding the cost of correcting every read first.

## Related tools

- **CONSENT** (Provides both arms of the benchmark: CONSENT-correct (--type ONT) corrects reads before assembly, and CONSENT-polish corrects the raw Miniasm assembly in the alternative arm; its runtime and peak RAM are the measured resource quantities) — https://github.com/morispi/CONSENT
- **Flye** (Assembles the CONSENT-corrected reads to produce the correct-then-assemble assembly in the first arm of the comparison)
- **Miniasm** (Produces the raw assembly (rawAssembly.fasta) that is subsequently polished with CONSENT-polish in the second arm)

## Examples

```
./CONSENT-correct --in longReads.fastq --out result.fasta --type ONT ; ./CONSENT-polish --contigs rawAssembly.fasta --reads longReads.fastq --out polishedAssembly.fasta
```

## Evaluation signals

- The comparison table must reproduce the qualitative reported outcome: assembling raw data then polishing consumes fewer resources (7.5 days, <400 GB RAM) than correcting then assembling (>17 days, 1 TB RAM) while yielding better assembly metrics
- Both arms must run on the identical raw long-read input file and use the same reference for metric computation, so NGA50/NGA75/errors per 100 kbp/misassemblies are directly comparable
- Resource columns (runtime and peak RAM) must be recorded for every step (correction, assembly, polishing), not only the end-to-end pipeline
- The polished assembly should show higher NGA50 and lower errors per 100 kbp and misassembly counts than the correct-then-assemble assembly
- CONSENT-polish invocation must use the raw Miniasm contigs plus the original raw reads (not corrected reads) as polishing evidence

## Limitations

- The runtime of CONSENT's correction step tends to be higher than that of other state-of-the-art methods, which inflates the correct-then-assemble arm's cost
- CONSENT correction of a full human dataset requires about 5 TB of disk space, driven by the overlaps file (storable on an external drive)
- The strategy of selecting only the N highest-identity overlaps may be inefficient when repeated regions grow longer
- CONSENT-based assemblies may cover a smaller proportion of the reference sequence; unaligned contigs were not analyzed in detail in the paper
- Correction of larger and more complex organisms than human had not been performed

## Evidence

- [abstract] On a human dataset, assembling raw data and polishing the assembly is less resource consuming than correcting then assembling reads, while providing better results: "on a human dataset, assembling the raw data and polishing the assembly is less resource consuming than correcting and then assembling the reads, while providing better results."
- [abstract] CONSENT scales to ultra-long reads and full human dataset: "CONSENT is the only method that efficiently scales to ultra-long reads, and allows to process a full human dataset, containing reads reaching up to 1.5 Mbp, in 10 days."
- [readme] CONSENT-polish CLI invocation from README: "`./CONSENT-polish --contigs example/rawAssembly.fasta --reads example/reads.fasta --out example/polishedAssembly.fasta`"
- [readme] CONSENT-correct CLI invocation from README: "`./CONSENT-correct --in longReads.fast[a|q] --out result.fasta --type readsTechnology`"
- [discussion] Assembly polishing feature results: "CONSENT’s assembly polishing feature also offers promising results."
