---
name: wallclock-time-measurement-of-assembly-runs
description: Use when comparing computational cost (running time and peak RAM) of
  multiple transcriptome assemblers on large FASTQ RNA-Seq datasets, e.g. to reproduce
  Table 5-style resource benchmarks before choosing an assembler for big SRA libraries.
when_to_use_negative:
- The goal is assembly quality comparison (assembled genes/isoforms, misassemblies)
  rather than resource usage — use rnaQUAST-based evaluation instead of timing benchmarks
- Input is metatranscriptome data, which the rnaSPAdes approach was not designed or
  evaluated for
- Datasets are small simulated or low-coverage libraries where the large-scale RAM/time
  differences (tens to hundreds of GB) do not manifest
edam_operation: http://edamontology.org/operation_0564
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_3173
- http://edamontology.org/topic_0196
tools:
- name: rnaSPAdes
  role: Benchmarked transcriptome assembler; measured to run about twice as fast as
    SPAdes with lower RAM usage
  repo: github.com/ablab/spades
- name: SPAdes
  role: Baseline genome assembler applied to RNA-Seq without modification; its runtime
    and RAM serve as the reference point rnaSPAdes is compared against
  repo: github.com/ablab/spades
- name: Trinity
  role: Benchmarked de novo transcriptome assembler; exceeds 100 GB peak RAM on the
    Arabidopsis dataset
- name: SOAPdenovo-Trans
  role: Benchmarked assembler; expected to be at least 3x faster than the others with
    under 30 GB RAM on both datasets
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/gigascience/giz100
    title: 'rnaSPAdes: a <i>de novo</i> transcriptome assembler and its application
      to RNA-Seq data'
  source_content_hash: sha256:5dfde9d0fa158f0e0d39d67ce055e1c14e254b808ff32110616a8eed7836587b
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - SOAPdenovo-Trans
  - SPAdes
  - Trinity
  - rnaSPAdes
  edam_operation: http://edamontology.org/operation_0564
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_3173
  - http://edamontology.org/topic_0196
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 7
---

# wallclock-time-measurement-of-assembly-runs

## Summary

Benchmark protocol for measuring wall-clock execution time and peak RAM consumption of de novo transcriptome assemblers (rnaSPAdes, SPAdes, Trinity, SOAPdenovo-Trans, BinPacker, Bridger) run on the two largest RNA-Seq datasets (Human large and Arabidopsis), reproducing the resource-usage comparison of Table 5 in the rnaSPAdes paper.

## When to use

Use when comparing computational cost (running time and peak RAM) of multiple transcriptome assemblers on large FASTQ RNA-Seq datasets, e.g. to reproduce Table 5-style resource benchmarks before choosing an assembler for big SRA libraries.

## When NOT to use

- The goal is assembly quality comparison (assembled genes/isoforms, misassemblies) rather than resource usage — use rnaQUAST-based evaluation instead of timing benchmarks
- Input is metatranscriptome data, which the rnaSPAdes approach was not designed or evaluated for
- Datasets are small simulated or low-coverage libraries where the large-scale RAM/time differences (tens to hundreds of GB) do not manifest

## Inputs

- Paired-end Illumina RNA-Seq FASTQ reads from NCBI SRA: SRR1957703, SRR1957706 (Human large dataset)
- Paired-end Illumina RNA-Seq FASTQ reads from NCBI SRA: SRR5344669, SRR5344670 (Arabidopsis dataset)
- Executable installations of rnaSPAdes, SPAdes, Trinity, SOAPdenovo-Trans, BinPacker, and Bridger with the paper's benchmarking command lines

## Outputs

- CSV table with one row per assembler per dataset and columns: dataset, assembler, wall-clock time, peak RAM
- Per-run resource logs from system utilities (e.g. `/usr/bin/time -v`) recording elapsed time and maximum resident set size

## How to apply

Download the paired-end reads for the two largest datasets in FASTQ format from NCBI SRA — Human large (SRR1957703, SRR1957706) and Arabidopsis (SRR5344669, SRR5344670). Run each benchmarked assembler (rnaSPAdes, SPAdes, Trinity, SOAPdenovo-Trans, BinPacker, Bridger) on each dataset under comparable conditions using the paper's command lines, wrapping each run with system resource measurement such as `/usr/bin/time -v` to capture wall-clock time and maximum resident set size. Aggregate one row per assembler per dataset into a CSV with columns dataset, assembler, wall-clock time, and peak RAM. Judge the benchmark by the paper's anchors: SOAPdenovo-Trans should be at least ~3x faster than any other assembler with under 30 GB RAM on both datasets, BinPacker, Bridger, and Trinity should exceed 100 GB peak RAM on the Arabidopsis dataset, and rnaSPAdes should come out about twice as fast as original SPAdes with lower RAM consumption. Keep hardware and thread counts identical across assembler runs so timing ratios are meaningful.

## Related tools

- **rnaSPAdes** (Benchmarked transcriptome assembler; measured to run about twice as fast as SPAdes with lower RAM usage) — github.com/ablab/spades
- **SPAdes** (Baseline genome assembler applied to RNA-Seq without modification; its runtime and RAM serve as the reference point rnaSPAdes is compared against) — github.com/ablab/spades
- **Trinity** (Benchmarked de novo transcriptome assembler; exceeds 100 GB peak RAM on the Arabidopsis dataset)
- **SOAPdenovo-Trans** (Benchmarked assembler; expected to be at least 3x faster than the others with under 30 GB RAM on both datasets)

## Examples

```
/usr/bin/time -v rnaspades.py -1 SRR5344669_1.fastq -2 SRR5344669_2.fastq -o rnaspades_arabidopsis
```

## Evaluation signals

- SOAPdenovo-Trans recorded wall-clock time is at least ~3x lower than every other assembler, with peak RAM under 30 GB on both Human large and Arabidopsis datasets
- BinPacker, Bridger, and Trinity peak RAM measurements exceed 100 GB on the Arabidopsis dataset
- rnaSPAdes runs in roughly half the wall-clock time of SPAdes on the same dataset with lower peak RAM
- CSV has exactly one row per (dataset, assembler) pair with non-empty wall-clock time and peak RAM values
- All assembler runs used the same hardware and thread counts, so time ratios are directly comparable

## Limitations

- No single assembler wins across all metrics, so resource efficiency alone does not determine assembler choice; assembler selection may require additional per-project benchmarks
- Timing and RAM values are hardware- and configuration-dependent and must be run under comparable conditions to be reproducible
- rnaSPAdes excludes the BayesHammer error correction module, so its faster runtime is partly attributable to a shorter pipeline, not only algorithmic efficiency

## Evidence

- [discussion] the rnaSPAdes pipeline seems to be about twice as fast and consumes less RAM than usual SPAdes use.: "the rnaSPAdes pipeline seems to be about twice as fast and consumes less RAM than usual SPAdes use."
- [methods] In order to run read error correction, reads should be in FASTQ or BAM format: "In order to run read error correction, reads should be in FASTQ or BAM format"
- [readme] SPAdes package contains assembly pipelines for isolated and single-cell bacterial, as well as metagenomic and transcriptomic data.: "SPAdes package contains assembly pipelines for isolated and single-cell bacterial, as well as metagenomic and transcriptomic data."
