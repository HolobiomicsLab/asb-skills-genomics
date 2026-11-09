---
name: compute-resource-profiling
description: Use when comparing the computational cost (wall-clock time and peak RAM)
  of multiple SV callers or similar pipelines on the same dataset and hardware, e.g.
  Blackbird vs Sniffles2/PBSV on HG002.
when_to_use_negative:
- The goal is call accuracy (F1/precision/recall) rather than runtime or memory cost
  — a separate benchmarking protocol is needed.
- The tools cannot run on equivalent hardware or the same input data, so measured
  times are not comparable.
- The tool under test is not wrapped by a timing utility, so wall time and peak RSS
  cannot be captured.
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: Blackbird
  role: SV caller benchmarked in SLR mode (SLR bam only) and hybrid mode (SLR bam
    + PacBio HiFi long reads) with /usr/bin/time -v
  repo: https://github.com/1dayac/Blackbird
- name: SPAdes
  role: Blackbird was forked from the SPAdes codebase; spades_for_blackbird must be
    installed and compiled before benchmarking Blackbird
  repo: https://github.com/1dayac/spades_for_blackbird
- name: BWA
  role: Indexes the GRCh37/hg19 reference genome required by Blackbird's --reference
    option
- name: SvABA
  role: SLR-based competitor called on the same HG002 SLR dataset under timing wrapper
    (65 min / 11.5 Gb reported)
- name: Manta
  role: SLR-based competitor called on the same HG002 SLR dataset under timing wrapper
    (15 min / 6.5 Gb reported)
- name: Sniffles2
  role: Long-read competitor called on the PacBio HiFi dataset under timing wrapper
    (3 min / 0.6 Gb reported)
- name: PBSV
  role: Long-read competitor called on the PacBio HiFi dataset under timing wrapper
    (1 h 19 min / 27 Gb reported)
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioadv/vbaf151
    title: 'Blackbird: structural variant detection using synthetic and low-coverage
      long-reads'
  source_content_hash: sha256:193c7ff727c678136196eb7f6bb847c7bebdb8a86c474d16d598f32e090de778
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/1dayac/Blackbird
  tools:
  - BWA
  - Blackbird
  - Manta
  - PBSV
  - SPAdes
  - Sniffles2
  - SvABA
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7058823529411765
n_claims: 17
---

# compute-resource-profiling

## Summary

Benchmark wall-clock runtime and peak memory (peak RSS) of bioinformatics tools by wrapping each execution with `/usr/bin/time -v`, as done to compare Blackbird (SLR and hybrid modes) against SvABA, Manta, Sniffles2, and PBSV on HG002. Enables reproducible computational-cost comparisons across SV callers on identical inputs and hardware.

## When to use

Use when comparing the computational cost (wall-clock time and peak RAM) of multiple SV callers or similar pipelines on the same dataset and hardware, e.g. Blackbird vs Sniffles2/PBSV on HG002.

## When NOT to use

- The goal is call accuracy (F1/precision/recall) rather than runtime or memory cost — a separate benchmarking protocol is needed.
- The tools cannot run on equivalent hardware or the same input data, so measured times are not comparable.
- The tool under test is not wrapped by a timing utility, so wall time and peak RSS cannot be captured.

## Inputs

- position-sorted and indexed HG002 (NA24385) 10X/stLFR SLR bam with BX barcode tags (GRCh37/hg19)
- position-sorted and indexed PacBio HiFi long-read bam plus matching FASTQ (34x for hybrid mode)
- BWA-indexed GRCh37/hg19 reference genome FASTA

## Outputs

- runtime_memory_benchmark.csv with wall-clock time and peak RSS per tool and mode
- per-tool outputs confirming successful completion (e.g. Blackbird out_50.vcf)
- /usr/bin/time -v logs reporting Elapsed (wall clock) time and Maximum resident set size

## How to apply

Run each tool on the identical input dataset under `/usr/bin/time -v` on equivalent hardware, so wall time and peak RSS are captured automatically. For Blackbird, index the GRCh37/hg19 reference with BWA, then run the blackbird binary with `--bam` (position-sorted, indexed SLR bam with BX tags), `--reference`, `--output`, and `--threads 32`; add `--long-bam` and `--long-read` for hybrid mode with the PacBio HiFi 34x data. Run the competitors on the same SLR bam (SvABA, Manta) or the same PacBio HiFi data (Sniffles2, PBSV), each under the same timing wrapper. Record runtime and peak memory for every tool and mode in a benchmark CSV, e.g. Blackbird SLR 6 h / 12 Gb, Blackbird hybrid 11 h / 117 Gb, SvABA 65 min / 11.5 Gb, Manta 15 min / 6.5 Gb, Sniffles2 3 min / 0.6 Gb, PBSV 1 h 19 min / 27 Gb. Accept the benchmark only if every tool completes successfully and measured values agree with reported values within hardware-dependent tolerance.

## Related tools

- **Blackbird** (SV caller benchmarked in SLR mode (SLR bam only) and hybrid mode (SLR bam + PacBio HiFi long reads) with /usr/bin/time -v) — https://github.com/1dayac/Blackbird
- **SPAdes** (Blackbird was forked from the SPAdes codebase; spades_for_blackbird must be installed and compiled before benchmarking Blackbird) — https://github.com/1dayac/spades_for_blackbird
- **BWA** (Indexes the GRCh37/hg19 reference genome required by Blackbird's --reference option)
- **SvABA** (SLR-based competitor called on the same HG002 SLR dataset under timing wrapper (65 min / 11.5 Gb reported))
- **Manta** (SLR-based competitor called on the same HG002 SLR dataset under timing wrapper (15 min / 6.5 Gb reported))
- **Sniffles2** (Long-read competitor called on the PacBio HiFi dataset under timing wrapper (3 min / 0.6 Gb reported))
- **PBSV** (Long-read competitor called on the PacBio HiFi dataset under timing wrapper (1 h 19 min / 27 Gb reported))

## Examples

```
/usr/bin/time -v blackbird -b NA24385.GRCh37.phased_possorted_bam.bam -r genome.fa -o blackbird_10X_chr1 -t 32 -l long.bam -m long.fastq -s spades.py
```

## Evaluation signals

- Every tool completes successfully and produces its expected output (e.g. Blackbird's position-sorted out_50.vcf with insertions/deletions > 50 bp).
- Measured wall time and peak memory match reported values within hardware-dependent tolerance: Blackbird SLR 6 h / 12 Gb, Blackbird hybrid 11 h / 117 Gb, SvABA 65 min / 11.5 Gb, Manta 15 min / 6.5 Gb, Sniffles2 3 min / 0.6 Gb, PBSV 1 h 19 min / 27 Gb.
- Each run is wrapped with /usr/bin/time -v so both 'Elapsed (wall clock) time' and 'Maximum resident set size' are captured per invocation.
- All tools run on identical inputs (same SLR bam for SLR-based callers, same PacBio HiFi data for long-read callers) and equivalent hardware.
- runtime_memory_benchmark.csv contains one entry per tool/mode with time and peak RSS populated.

## Limitations

- Blackbird is memory- and time-intensive in hybrid mode (11 h / 117 Gb), far exceeding Sniffles2 (3 min / 0.6 Gb) and PBSV (1 h 19 min / 27 Gb); comparisons are only fair on equivalent hardware.
- Reported timings are hardware-dependent, so measured values must be compared to reported ones within reasonable tolerance, not exactly.
- Blackbird requires additional inputs beyond the SLR bam: a BWA-indexed reference, and for hybrid mode both a --long-bam and paired --long-read FASTQ; more threads than the default of 1 are beneficial.

## Evidence

- [results] Reported Blackbird SLR and hybrid runtimes: [evidence span withheld — anchor status: not_found]
- [readme] SLR bam input requirement: "`--bam` or `-b` [required] - position-sorted and indexed SLR Read bam-file with BX tags"
- [readme] Long-read hybrid mode pairing requirement: "`--long-bam` or `-l` - Position-sorted and indexed long read file (optional, should be paired with long-read option)"
- [readme] Output VCF validation artifact: "`out_50.vcf` is a position sorted VCF file, that contains insertion and deletion calls longer than 50 bp."
- [methods] Compilation prerequisite: "To compile Blackbird in assembler foder execute:

``n./spades_compile.sh"
- [methods] BWA-indexed reference requirement: "`--rerefence` or `-r` [required] - BWA-indexed reference genome"
- [methods] Hybrid-mode performance claim motivating benchmarking: "In a hybrid-mode Blackbird demonstrated results similar to state-of-the-art long read tools, but requires less long reads to achieve same results."
