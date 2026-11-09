---
name: long-read-alignment-command-profiling
description: 'Use when benchmarking a streamed minimap2->HLAminer long-read command:
  wrap it in GNU time -v to capture wall-clock runtime and peak RAM on ONT/PacBio
  WGS FASTQ inputs.'
when_to_use_negative:
- The goal is validating HLA allele-prediction accuracy against clinically confirmed
  HLA types - this skill measures runtime/RAM, not prediction correctness
- The input is Illumina short-read data analyzed via the HPTASR targeted-assembly
  path - the benchmark concerns streamed long-read minimap2 alignments
- You need persistent SAM/BAM alignment files for later inspection - the streaming
  setup deliberately avoids storing bulky alignment files on disk
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_0091
tools:
- name: minimap2
  role: Long-read aligner (run with -t 60 -ax map-ont --MD) whose execution dominates
    the pipeline's run time and RAM; its SAM output is streamed rather than written
    to disk
  repo: https://github.com/lh3/minimap2
- name: HLAminer v1.4
  role: Downstream HLA prediction process (HLAminer.pl with -a stream) that consumes
    piped SAM alignments inside the profiled command
  repo: https://github.com/bcgsc/HLAminer
- name: GNU time
  role: Wraps the streaming command as '/usr/bin/time -v -o <file>.time' to capture
    elapsed wall-clock time and maximum resident set size
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1002/cpz1.70124
    title: Streaming Long‐Read Sequence Alignments for HLA Predictions Using HLAminer
  source_content_hash: sha256:3059332d6274c197883c96c025e9a046ef790c012798e97ee3073426620e844a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/lh3/minimap2
  tools:
  - GNU time
  - HLAminer v1.4
  - minimap2
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# long-read-alignment-command-profiling

## Summary

Profiles the streamed minimap2-to-HLAminer v1.4 long-read HLA-typing command by wrapping it in GNU time -v so wall-clock runtime (~40 min for a ~10x ONT WGS set) and peak RAM (up to 35 GB) can be reproduced against the protocol's Table 2 benchmarks.

## When to use

Use when benchmarking a streamed minimap2->HLAminer long-read command: wrap it in GNU time -v to capture wall-clock runtime and peak RAM on ONT/PacBio WGS FASTQ inputs.

## When NOT to use

- The goal is validating HLA allele-prediction accuracy against clinically confirmed HLA types - this skill measures runtime/RAM, not prediction correctness
- The input is Illumina short-read data analyzed via the HPTASR targeted-assembly path - the benchmark concerns streamed long-read minimap2 alignments
- You need persistent SAM/BAM alignment files for later inspection - the streaming setup deliberately avoids storing bulky alignment files on disk

## Inputs

- ONT PromethION WGS FASTQ (e.g., ERR2585115.fastq.gz, NA19240, ~10x coverage, ENA project PRJEB26791)
- GRCh38-noChr6-HLA-I_II_GEN reference FASTA (GCA_000001405.15 genomic, chr-only, chr6 removed) for minimap2 -x map-ont alignment
- HLAminer v1.4 installation with database/HLA-I_II_GEN.fasta and hla_nom_p.txt (IMGT/HLA P designation file)

## Outputs

- GNU time profile file (e.g., minimap_hlaminerERR2585115-1mod.time) reporting 'Elapsed (wall clock) time' and 'Maximum resident set size'
- Benchmark verdict comparing measured runtime and peak RAM against the protocol's ~40 min / up-to-35 GB report, with HLA-I/II allele predictions as a side output

## How to apply

Obtain the target ONT WGS FASTQ (e.g., ERR2585115.fastq.gz of NA19240 from ENA project PRJEB26791) and the GRCh38-noChr6-HLA-I_II_GEN reference FASTA (GCA_000001405.15, chr-only, chr6 removed), then wrap the single streaming command in GNU time: '/usr/bin/time -v -o <run>.time minimap2 -t 60 -ax map-ont --MD <ref> <reads> | ./HLAminer.pl -h ../database/HLA-I_II_GEN.fasta -s 500 -q 1 -i 1 -p ../database/hla_nom_p.txt -a stream'. Piping minimap2's SAM output directly into HLAminer v1.4 avoids writing bulky SAM files to disk, mirroring the protocol's streaming setup. After completion, parse the .time file for the 'Elapsed (wall clock) time' and 'Maximum resident set size' entries. For the ~10x coverage NA19240 PromethION dataset, the protocol expects approximately 40 minutes wall-clock with peak RAM inside an up-to-35 GB envelope; treat runtime scaling with read count and minimap2's dominance of the pipeline's run time and RAM as the expected behavior.

## Related tools

- **minimap2** (Long-read aligner (run with -t 60 -ax map-ont --MD) whose execution dominates the pipeline's run time and RAM; its SAM output is streamed rather than written to disk) — https://github.com/lh3/minimap2
- **HLAminer v1.4** (Downstream HLA prediction process (HLAminer.pl with -a stream) that consumes piped SAM alignments inside the profiled command) — https://github.com/bcgsc/HLAminer
- **GNU time** (Wraps the streaming command as '/usr/bin/time -v -o <file>.time' to capture elapsed wall-clock time and maximum resident set size)

## Examples

```
/usr/bin/time -v -o minimap_hlaminerERR2585115-1mod.time minimap2 -t 60 -ax map-ont --MD GRCh38-noChr6-HLA-I_II_GEN.fa ERR2585115.fastq.gz | ./HLAminer.pl -a stream
```

## Evaluation signals

- The .time file contains both 'Elapsed (wall clock) time' and 'Maximum resident set size' entries for the streaming run
- Wall-clock time for the ~10x ERR2585115 (NA19240) dataset is approximately 40 minutes, matching the protocol's Table 2 benchmark
- Peak RAM stays within the reported up-to-35 GB envelope
- No SAM file is written to disk - minimap2 output streams directly into HLAminer.pl in stream mode (-a stream)
- Runtime scales with read count: larger ONT datasets take proportionally longer, consistent with minimap2 dominating the pipeline's run time and RAM

## Limitations

- Absolute numbers are hardware- and parameter-dependent (e.g., minimap2 -t 60); the ~40 min reference applies to the ~10x coverage NA19240 PromethION dataset (2018, older chemistry)
- minimap2 dominates the pipeline's run time and RAM, so the profile does not isolate HLAminer's resource cost
- Runtime scales with read count, so denser ONT datasets will legitimately exceed the 40-minute reference value

## Evidence

- [intro] we adapted HLAminer (v1.4) to predict HLA alleles from any long sequence reads, by streaming sequence alignment outputs directly into it: "we adapted HLAminer (v1.4) to predict HLA alleles from any long sequence reads, by streaming sequence alignment outputs directly into it"
- [abstract] This method is as simple as running minimap2, scales efficiently with the number of sequences: "This method is as simple as running minimap2, scales efficiently with the number of sequences"
- [abstract] works with any read aligner compatible with the SAM file format—eliminating the need to store bulky alignment files on disk: "works with any read aligner compatible with the SAM file format—eliminating the need to store bulky alignment files on disk"
- [methods] HLAminer v1.4 provides initial support for HLA prediction from raw uncorrected shotgun nanopore long reads: "HLAminer v1.4 provides initial support for HLA prediction from raw uncorrected shotgun nanopore long reads"
- [readme] Ability to stream the (.sam) output of modern read  aligners, directly into HLAminer.: "Ability to stream the (.sam) output of modern read  aligners, directly into HLAminer."
