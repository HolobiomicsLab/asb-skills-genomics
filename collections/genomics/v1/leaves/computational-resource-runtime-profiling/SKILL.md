---
name: computational-resource-runtime-profiling
description: Use when benchmarking assembly tools on real datasets (E. coli, yeast,
  C. elegans, human CHM1) and you must record runtime, peak memory, and failures per
  dataset/tool combination under matched compute resources.
when_to_use_negative:
- When the goal is only assembly quality assessment without timing or memory claims
  — QUAST alone suffices and no resource profiling is needed
- When datasets or compute resources cannot be matched across assemblers, since runtime
  and memory comparisons would be confounded
edam_topics:
- http://edamontology.org/topic_0196
- http://edamontology.org/topic_3174
tools:
- name: HASLR
  role: Hybrid assembler being profiled for runtime, memory, and misassemblies on
    real datasets
  repo: https://github.com/vpc-ccg/haslr
- name: PBcR
  role: Comparator hybrid assembler benchmarked under matched compute resources
- name: MaSuRCA
  role: Comparator hybrid assembler benchmarked under matched compute resources
- name: QUAST
  role: Evaluates all resulting assemblies for contiguity and accuracy metrics (NGA50,
    genome fraction, misassemblies)
- name: Minia
  role: Assembles short reads into contigs (k=49) used by HASLR during the profiled
    runs
  repo: https://github.com/GATB/minia
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.isci.2020.101389
    title: 'HASLR: Fast Hybrid Assembly of Long Reads'
  source_content_hash: sha256:276738a6087a41e983d55dbdfd58b4d6c173e68a82deb0cf8a77699acfb5c260
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/vpc-ccg/haslr
  repo_url: https://github.com/vpc-ccg/haslr
  tools:
  - HASLR
  - MaSuRCA
  - Minia
  - Oxford Nanopore
  - PBcR
  - Pilon
  - QUAST
  - Racon
  - fastutils
  - minimap2
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0196
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6875
n_claims: 16
---

# computational-resource-runtime-profiling

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Profiles wall-clock runtime, peak memory usage, and tool failures when benchmarking genome assemblers (e.g., HASLR vs PBcR and MaSuRCA) on real datasets, so speed and resource claims in the paper can be reproduced and compared under matched compute resources.

## When to use

Use when benchmarking assembly tools on real datasets (E. coli, yeast, C. elegans, human CHM1) and you must record runtime, peak memory, and failures per dataset/tool combination under matched compute resources.

## When NOT to use

- When the goal is only assembly quality assessment without timing or memory claims — QUAST alone suffices and no resource profiling is needed
- When datasets or compute resources cannot be matched across assemblers, since runtime and memory comparisons would be confounded

## Inputs

- Hybrid sequencing datasets: PacBio or Oxford Nanopore long reads plus Illumina short reads from the same sample (E. coli Loman ultra-long ONT + MG1655 MiSeq; yeast ERX1725434/ERX1725435/ERX1725441 + ERX1943903; C. elegans DevNet PacBio + SRR065390; CHM1 PacBio SRP044331 + Illumina SRX652547)
- Assembler configurations with matched compute resources (e.g., 64 CPU threads)
- Assembled contigs from each assembler (e.g., k=49 Minia short-read contigs, long-read selections such as lr25x.fasta)

## Outputs

- benchmark_results.csv with QUAST metrics (NGA50, genome fraction, misassemblies), wall-clock runtime, peak memory, and failure records per dataset/tool combination
- Failure log noting tools that exceeded memory limits or time budgets (e.g., Flye, SPAdes, hybridSPAdes, Unicycler on human; Unicycler >1 month on E. coli)

## How to apply

Assemble each dataset with each assembler (e.g., HASLR via haslr.py with 64 threads, -x {pacbio,nanopore} for the long-read type, -g with k/m/g genome-size suffixes, default --cov-lr 25, k=49 Minia contigs) and comparators such as PBcR and MaSuRCA under matched compute resources. For every dataset/tool combination, record wall-clock runtime, peak memory, and any assembler failure, including timeout thresholds (e.g., Unicycler did not finish E. coli within one month; several tools failed on the human genome due to memory limits). Evaluate all assemblies with QUAST to obtain contiguity and accuracy metrics such as NGA50, genome fraction, and misassemblies. Compile metrics, runtime, memory, and failure records into benchmark_results.csv, then validate against the expected pattern that HASLR is the fastest assembler with the lowest number of misassemblies on most samples (e.g., CHM1 human assembled in less than 10 hours using 64 CPU threads).

## Related tools

- **HASLR** (Hybrid assembler being profiled for runtime, memory, and misassemblies on real datasets) — https://github.com/vpc-ccg/haslr
- **PBcR** (Comparator hybrid assembler benchmarked under matched compute resources)
- **MaSuRCA** (Comparator hybrid assembler benchmarked under matched compute resources)
- **QUAST** (Evaluates all resulting assemblies for contiguity and accuracy metrics (NGA50, genome fraction, misassemblies))
- **Minia** (Assembles short reads into contigs (k=49) used by HASLR during the profiled runs) — https://github.com/GATB/minia

## Examples

```
haslr.py -t 64 -x nanopore -g 4.6m -l ont_reads.fq.gz -s illumina_R1.fq.gz illumina_R2.fq.gz
```

## Evaluation signals

- benchmark_results.csv contains runtime, peak memory, QUAST misassemblies, and a failure field for every dataset/tool combination
- Reproduced pattern: HASLR is the fastest assembler with the lowest number of misassemblies on most samples
- CHM1 human assembly completes in less than 10 hours using 64 CPU threads with HASLR
- Failures are recorded rather than silently dropped, e.g., tools exceeding memory limits on human or Unicycler not finishing E. coli within one month
- QUAST contiguity/accuracy metrics (NGA50, genome fraction) show HASLR on par with the best-performing tools

## Limitations

- HASLR usually has a higher indel rate compared with other tools, and sometimes produces lower genome fraction and/or NGA50 (clearest versus Canu on the human genome)
- High heterozygosity regions are not resolved because the Minia short read contigs for these regions are fragmented and filtered out
- Backbone graph connectivity depends on the existence of unique SR contigs; their absence yields multiple connected components, affecting both quality and runtime
- Some competing tools fail outright under memory limits or long runtimes, so per-tool failures must be recorded rather than averaged

## Evidence
