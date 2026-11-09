---
name: genome-assembly-qc-metrics-interpretation
description: Use when comparing genome assemblies from different assemblers on the
  same datasets and you need contiguity/accuracy metrics (NGA50, genome fraction,
  misassemblies) plus wall-clock runtime and peak memory to rank tools.
when_to_use_negative:
- Input is a single assembly with no comparator — there is no benchmark table to produce
  and per-tool runtime/memory ranking is meaningless
- The task is polishing/consensus correction (e.g. Pilon/Racon) rather than cross-assembler
  quality comparison
- Long reads are high-accuracy (HiFi/Q20) or ultra-long ONT, which are outside the
  HASLR benchmark's error-prone PacBio CLR / standard ONT scope
edam_operation: http://edamontology.org/operation_3180
edam_topics:
- http://edamontology.org/topic_0196
- http://edamontology.org/topic_0622
tools:
- name: QUAST
  role: Evaluates all resulting assemblies to obtain contiguity and accuracy metrics
    (NGA50, genome fraction, misassemblies)
- name: HASLR
  role: Assembler under evaluation; source code and issue tracker at github.com/vpc-ccg/haslr
  repo: https://github.com/vpc-ccg/haslr
- name: PBcR
  role: Comparator hybrid assembler run under matched compute resources
- name: MaSuRCA
  role: Comparator hybrid assembler run under matched compute resources
- name: Pilon
  role: Polishing tool relevant to interpreting indel/mismatch rates in evaluated
    assemblies
- name: Racon
  role: Polishing tool relevant to interpreting indel/mismatch rates in evaluated
    assemblies
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.isci.2020.101389
    title: 'HASLR: Fast Hybrid Assembly of Long Reads'
  source_content_hash: sha256:8bda09f0ceceb13fad6fc8265b27889bc7c14b4984c9ce8a330bc6c6a2b49b17
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
  edam_operation: http://edamontology.org/operation_3180
  edam_topics:
  - http://edamontology.org/topic_0196
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7857142857142857
n_claims: 14
---

# genome-assembly-qc-metrics-interpretation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Evaluate de novo genome assemblies with QUAST to obtain contiguity and accuracy metrics (NGA50, genome fraction, misassemblies) and pair them with runtime, peak memory, and tool-failure records to rank competing assemblers.

## When to use

Use when comparing genome assemblies from different assemblers on the same datasets and you need contiguity/accuracy metrics (NGA50, genome fraction, misassemblies) plus wall-clock runtime and peak memory to rank tools.

## When NOT to use

- Input is a single assembly with no comparator — there is no benchmark table to produce and per-tool runtime/memory ranking is meaningless
- The task is polishing/consensus correction (e.g. Pilon/Racon) rather than cross-assembler quality comparison
- Long reads are high-accuracy (HiFi/Q20) or ultra-long ONT, which are outside the HASLR benchmark's error-prone PacBio CLR / standard ONT scope

## Inputs

- Assembled contig/scaffold FASTA files from each assembler (e.g. HASLR, PBcR, MaSuRCA outputs)
- Reference genome for QUAST-based evaluation (E. coli, yeast, C. elegans, CHM1 human)
- Runtime logs and peak-memory records per assembler run

## Outputs

- QUAST metric table per assembly (NGA50, genome fraction, misassemblies, indel rate)
- benchmark_results.csv compiling metrics, runtime, memory, and failure records per dataset/tool
- Failure log documenting which assembler/dataset combinations failed and why (e.g. memory limits, time budget exceeded)

## How to apply

Assemble each dataset with every assembler under matched compute resources (e.g. 64 CPU threads), then run QUAST on every resulting assembly to extract contiguity and accuracy metrics such as NGA50, genome fraction, and misassembly counts. Record wall-clock runtime and peak memory per dataset/tool combination, and explicitly log any assembler failures (e.g. a tool exceeding memory limits or not finishing within the time budget) rather than leaving the cell blank. Interpret results jointly: in the HASLR benchmark, the headline claim is that HASLR is the fastest assembler with the lowest number of misassemblies on most samples while remaining on par with other tools in contiguity and accuracy. Do not treat contiguity alone as success — the article notes HASLR can have a higher indel rate and, especially versus Canu on the human genome, lower genome fraction and/or NGA50, so report indel rate and reference-based metrics alongside speed.

## Related tools

- **QUAST** (Evaluates all resulting assemblies to obtain contiguity and accuracy metrics (NGA50, genome fraction, misassemblies))
- **HASLR** (Assembler under evaluation; source code and issue tracker at github.com/vpc-ccg/haslr) — https://github.com/vpc-ccg/haslr
- **PBcR** (Comparator hybrid assembler run under matched compute resources)
- **MaSuRCA** (Comparator hybrid assembler run under matched compute resources)
- **Pilon** (Polishing tool relevant to interpreting indel/mismatch rates in evaluated assemblies)
- **Racon** (Polishing tool relevant to interpreting indel/mismatch rates in evaluated assemblies)

## Examples

```
quast.py haslr_assembly.fasta -r reference.fasta --labels HASLR -o quast_out
```

## Evaluation signals

- QUAST output contains NGA50, genome fraction, and misassembly counts for every assembler/dataset combination
- benchmark_results.csv reproduces the reported pattern: HASLR fastest with lowest misassemblies on most samples (e.g. CHM1 human assembled in less than 10 hours using 64 CPU threads)
- Failure cells are explicitly recorded (e.g. Flye, SPAdes, hybridSPAdes, Unicycler failed on human due to memory limits; Unicycler did not finish E. coli within one month)
- Indel rate is reported separately, since HASLR usually has a higher indel rate than other tools
- Compute resources (threads, memory) are matched across assemblers before comparing runtime

## Limitations

- HASLR sometimes generates assemblies with relatively lower genome fraction and/or NGA50 compared with other tools, especially Canu on the human genome
- HASLR usually has a higher indel rate compared with other tools
- Benchmark conclusions are specific to error-prone long reads (PacBio CLR, standard ONT) with Illumina short reads; ultra-long ONT and phased diploid assemblies are out of scope
- Some failures (e.g. Unicycler on E. coli within one month) require long time budgets, so incomplete benchmarks may miss late-terminating competitors

## Evidence
