---
name: snv-caller-performance-tabulation
description: Use when comparing the computational cost (wall-clock time, peak memory
  footprint) of SNV callers on ONT long-read BAMs, e.g. benchmarking miniSNV against
  Clair3 and Longshot on a Dorado sup-basecalled GIAB HG002 dataset.
when_to_use_negative:
- The goal is variant-calling accuracy (F1-score, precision, recall against GIAB truth)
  rather than runtime and memory benchmarking — use the callers' accuracy evaluation
  workflow instead
- The sequencing data are short-read Illumina or PacBio — the miniSNV benchmark is
  optimized for ONT long reads and its efficiency claims do not transfer
- The input BAM is not samtools-indexed or no reference FASTA/index is available —
  miniSNV's chr_chunk_task.py requires both indexed BAM and indexed FASTA
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0622
tools:
- name: miniSNV
  role: Benchmark subject; lightweight statistical SNV caller whose runtime, memory
    footprint, and F1-score/recall are measured; run via chr_chunk_task.py with a
    1000 Genomes population VCF index
  repo: https://github.com/CuiMiao-HIT/miniSNV
- name: Clair3
  role: Benchmark subject; pileup + full-alignment DL caller run with the latest ONT-trained
    rerio model (clair3_models) on the HG002 sup BAM
  repo: https://github.com/nanoporetech/rerio
- name: Longshot
  role: Benchmark subject; diploid SNV caller using a pair hidden Markov model, included
    in the runtime/memory comparison
- name: DeepVariant
  role: Benchmark subject; DL-based variant caller included in the single-thread and
    default-threading resource comparison
- name: PEPPER-Margin-DeepVariant
  role: Benchmark subject; integrated PEPPER + Margin + DeepVariant caller included
    in the resource benchmark
- name: Nanocaller
  role: Benchmark subject; pileup-based caller with a three-layer CNN, included in
    the resource benchmark
- name: WhatsHap
  role: Dependency installed (conda bioconda::whatshap) for miniSNV's read-based phasing
    and haplotagging step, which runs inside each timed miniSNV invocation
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bib/bbae473
    title: 'miniSNV: accurate and fast single nucleotide variant calling from nanopore
      sequencing data'
  source_content_hash: sha256:b7f045e511068f4ac1a85b438c3855d7c1360586fcf8fa0f87b9adf4074961b7
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-4.0
    url: https://github.com/CuiMiao-HIT/miniSNV
  repo_url: https://github.com/CuiMiao-HIT/miniSNV
  tools:
  - Clair3
  - DeepVariant
  - Longshot
  - Nanocaller
  - PEPPER-Margin-DeepVariant
  - WhatsHap
  - miniSNV
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# snv-caller-performance-tabulation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

A benchmarking workflow that measures wall-clock runtime (seconds) and peak resident set size (GB) of ONT SNV callers — miniSNV, Clair3, Longshot, DeepVariant, PEPPER-Margin-DeepVariant, Nanocaller — under single-thread and default threading on the HG002 Dorado sup BAM, and tabulates results into performance_table.csv.

## When to use

Use when comparing the computational cost (wall-clock time, peak memory footprint) of SNV callers on ONT long-read BAMs, e.g. benchmarking miniSNV against Clair3 and Longshot on a Dorado sup-basecalled GIAB HG002 dataset.

## When NOT to use

- The goal is variant-calling accuracy (F1-score, precision, recall against GIAB truth) rather than runtime and memory benchmarking — use the callers' accuracy evaluation workflow instead
- The sequencing data are short-read Illumina or PacBio — the miniSNV benchmark is optimized for ONT long reads and its efficiency claims do not transfer
- The input BAM is not samtools-indexed or no reference FASTA/index is available — miniSNV's chr_chunk_task.py requires both indexed BAM and indexed FASTA

## Inputs

- Dorado sup-basecalled ONT BAM from GIAB HG002 (kit v14 chemistry, R10.4.1 flowcells), samtools-indexed
- GRCh38 reference FASTA, samtools-indexed
- miniSNV index folder built from the 1000 Genomes population VCF (five files, used with --useindex)
- ONT rerio-trained Clair3 model for the matching Dorado basecaller version

## Outputs

- performance_table.csv with per-caller wall-clock runtime (seconds) and peak RSS (GB) under single-thread and default threading
- Per-run resource monitor logs (e.g. /usr/bin/time -v output) for each caller invocation
- Caller VCF outputs used to cross-check that accuracy (F1-score, recall > 99.7%) accompanies the efficiency results

## How to apply

Obtain the HG002 sup basecalled ONT BAM from the GIAB Ashkenazi Trio deposit (PromethION, ligation sequencing kit v14 chemistry, R10.4.1 flowcells, ~200 Gbases), the GRCh38 reference, and the miniSNV index built from the 1000 Genomes population VCF of known common variants. Run each caller on the sup BAM in two configurations — single thread and default threading (miniSNV default is --threads 16) — wrapping every invocation with a resource monitor such as /usr/bin/time -v to capture wall-clock seconds and peak RSS in GB. Use the ONT rerio-trained Clair3 model matching the Dorado basecaller version, and samtools-index the BAM and FASTA before running miniSNV via chr_chunk_task.py. Tabulate per-caller runtime and peak RSS into performance_table.csv. Validate that miniSNV shows the lowest peak memory (4.76 GB, versus 10.84 GB for Clair3 and 88.13 GB for Longshot — roughly 2× and 20× smaller) and the shortest single-thread runtime while its F1-score and recall on the HG002 sup-dataset exceed 99.7%.

## Related tools

- **miniSNV** (Benchmark subject; lightweight statistical SNV caller whose runtime, memory footprint, and F1-score/recall are measured; run via chr_chunk_task.py with a 1000 Genomes population VCF index) — https://github.com/CuiMiao-HIT/miniSNV
- **Clair3** (Benchmark subject; pileup + full-alignment DL caller run with the latest ONT-trained rerio model (clair3_models) on the HG002 sup BAM) — https://github.com/nanoporetech/rerio
- **Longshot** (Benchmark subject; diploid SNV caller using a pair hidden Markov model, included in the runtime/memory comparison)
- **DeepVariant** (Benchmark subject; DL-based variant caller included in the single-thread and default-threading resource comparison)
- **PEPPER-Margin-DeepVariant** (Benchmark subject; integrated PEPPER + Margin + DeepVariant caller included in the resource benchmark)
- **Nanocaller** (Benchmark subject; pileup-based caller with a three-layer CNN, included in the resource benchmark)
- **WhatsHap** (Dependency installed (conda bioconda::whatshap) for miniSNV's read-based phasing and haplotagging step, which runs inside each timed miniSNV invocation)

## Examples

```
/usr/bin/time -v python chr_chunk_task.py --fin_ref GRCh38.fa --fin_bam HG002_sup.bam --workDir out --threads 1
```

## Evaluation signals

- miniSNV exhibits the lowest peak RSS of 4.76 GB among all callers on the HG002 sup-dataset
- Clair3 peak RSS is ~10.84 GB and Longshot peak RSS is ~88.13 GB, i.e. miniSNV is smaller by about 2× and 20× respectively
- miniSNV records the shortest single-thread wall-clock runtime among all benchmarked callers
- miniSNV F1-score and recall on the HG002 sup-dataset both exceed 99.7% while runtime/memory are measured
- performance_table.csv contains runtime and peak RSS rows for every caller under both single-thread and default threading configurations

## Limitations

- Peak RSS and wall-clock measurements are configuration-dependent: single-thread vs default threading (miniSNV default --threads 16) yields different numbers, so both configurations must be recorded
- Results are specific to the GIAB Ashkenazi Trio ONT data (kit v14, R10.4.1, ~200 Gbases per sample) Dorado sup-basecalled BAMs; error profiles and coverage will shift absolute runtime and memory
- Clair3 accuracy depends on model–basecaller matching: rerio Dorado sup models must match the Dorado version used, and rerio Clair3 models are compatible with Clair3 v1.x but not v2.x

## Evidence

- [results] benchmarks show miniSNV lowest memory of 4.76 GB vs 10.84 GB Clair3: [evidence span withheld — anchor status: not_found]
- [other] run callers under threading configs and record time and peak RSS: [evidence span withheld — anchor status: not_found]
