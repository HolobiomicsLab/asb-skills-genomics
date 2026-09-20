---
name: computational-resource-profiling
description: Use when benchmarking CONSENT (or correction/polishing alternatives like
  Canu, Daccord, FLAS, MECAT, RACON) on PacBio/ONT datasets and runtime, peak RAM,
  or disk usage must be reported next to accuracy metrics.
when_to_use_negative:
- When the dataset is too small to differentiate tool scalability — e.g. the 10x simulated
  PacBio example dataset — resource profiling yields numbers that do not extrapolate
  to full-scale benchmarks like C. elegans or human.
- When only correctness metrics are needed and no runtime/memory comparison between
  correctors or polishers is intended.
- When profiling read quality alone (error rate, recall, precision via ELECTOR or
  QUAST-LG) without a resource dimension is the goal.
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: CONSENT
  role: Subject tool being profiled — record runtime and peak RAM for CONSENT-correct
    (--type PB or ONT) and CONSENT-polish runs
  repo: https://github.com/morispi/CONSENT
- name: Canu
  role: Self-correction baseline whose runtime and memory are recorded for the PacBio
    benchmark table
- name: Daccord
  role: Baseline profiled to expose its 3x-11x higher memory consumption and failure
    to scale to C. elegans
- name: RACON
  role: State-of-the-art polishing baseline; its runtime/memory (e.g. 5 min 55 sec
    on E. coli 60x) is compared against CONSENT-polish
- name: Flye
  role: Assembler used in the correct-then-assemble human pipeline whose combined
    runtime/memory footprint is measured
provenance:
  source_task_ids:
  - task_002
  - task_005
  - task_007
  source_papers:
  - doi: 10.1038/s41598-020-80757-5
    title: Scalable long read self-correction and assembly polishing with multiple
      sequence alignment
  source_content_hash: sha256:749f63efdf60d33333785cf55bf97f6169571cab941d66a9c8aeef7582acec86
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/morispi/CONSENT
  tools:
  - CONSENT
  - Canu
  - Daccord
  - ELECTOR
  - FLAS
  - Flye
  - MECAT
  - Miniasm
  - ONT
  - RACON
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6818181818181818
n_claims: 22
---

# computational-resource-profiling

## Summary

Profiles wall-clock runtime, number of cores, peak RAM, and (where noted) disk footprint of long-read correctors and polishers — CONSENT, Canu, Daccord, FLAS, MECAT, RACON, Flye — alongside quality metrics, so resource scalability of self-correction and polishing pipelines can be quantified and compared.

## When to use

Use when benchmarking CONSENT (or correction/polishing alternatives like Canu, Daccord, FLAS, MECAT, RACON) on PacBio/ONT datasets and runtime, peak RAM, or disk usage must be reported next to accuracy metrics.

## When NOT to use

- When the dataset is too small to differentiate tool scalability — e.g. the 10x simulated PacBio example dataset — resource profiling yields numbers that do not extrapolate to full-scale benchmarks like C. elegans or human.
- When only correctness metrics are needed and no runtime/memory comparison between correctors or polishers is intended.
- When profiling read quality alone (error rate, recall, precision via ELECTOR or QUAST-LG) without a resource dimension is the goal.

## Inputs

- PacBio or ONT long-read dataset in FASTA/FASTQ (e.g. E. coli, S. cerevisiae, C. elegans 30x/60x simulated sets; full human ultra-long ONT set)
- Raw de novo assembly (e.g. from Miniasm) plus the long reads for polishing-mode profiling
- Corrector/polisher CLI runs with a pinned core count (e.g. 4 cores, CONSENT --nproc)

## Outputs

- Per-run resource metrics: wall-clock runtime, number of cores, peak RAM (and disk for temporary overlaps files)
- Aggregated benchmark table in the layout of Table 1 (error rate, recall, precision, runtime, memory per tool per dataset)
- Polishing and pipeline comparison tables (e.g. polishing_benchmark_metrics.tsv, consent_pipeline_comparison.tsv)

## How to apply

For every corrector or polishing run, record wall-clock time, core count, and peak RAM together with the quality evaluation. Use the README's example dataset as a calibration point: CONSENT-correct should take ~2 min using up to 750 MB RAM on 4 cores, and CONSENT-polish ~15 sec and at most 150 MB RAM on 4 cores; deviations indicate misconfiguration. Apply this uniformly across the compared tools (Canu, Daccord, FLAS, MECAT for correction; RACON for polishing) on the same six simulated PacBio datasets (E. coli, S. cerevisiae, C. elegans at 30x/60x) and real ONT datasets, then aggregate into a Table 1-style benchmark table. For pipeline-level profiling on the full human ONT dataset, track total time and memory across whole pipelines — correction + Flye assembly vs. Miniasm assembly + CONSENT-polish — to test the paper's claim that assemble-then-polish runs in 7.5 days with <400 GB RAM versus 17 days and >1 TB for correct-then-assemble. Account for disk separately: the overlaps file for a full human dataset drives a ~5 TB temporary storage requirement.

## Related tools

- **CONSENT** (Subject tool being profiled — record runtime and peak RAM for CONSENT-correct (--type PB or ONT) and CONSENT-polish runs) — https://github.com/morispi/CONSENT
- **Canu** (Self-correction baseline whose runtime and memory are recorded for the PacBio benchmark table)
- **Daccord** (Baseline profiled to expose its 3x-11x higher memory consumption and failure to scale to C. elegans)
- **RACON** (State-of-the-art polishing baseline; its runtime/memory (e.g. 5 min 55 sec on E. coli 60x) is compared against CONSENT-polish)
- **Flye** (Assembler used in the correct-then-assemble human pipeline whose combined runtime/memory footprint is measured)

## Examples

```
./CONSENT-correct --in example/reads.fasta --out example/correctedReads.fasta --type PB
```

## Evaluation signals

- Example dataset calibration: CONSENT-correct completes in ~2 min with ≤750 MB RAM on 4 cores; CONSENT-polish in ~15 sec with ≤150 MB RAM on 4 cores.
- Reproduced Table 1 runtime and memory values for CONSENT on simulated PacBio datasets fall within the published ranges alongside ~0.31%/0.18%/0.39% post-correction error rates.
- Polishing comparison shows CONSENT-polish consistently faster than RACON (e.g. 30 sec vs 5 min 55 sec on E. coli 60x; 2x-38x overall) with ≤4x less memory.
- Full human dataset pipeline comparison reproduces correction+assembly at >17 days and >1 TB RAM versus assembly+polishing at 7.5 days and <400 GB RAM.
- Disk accounting shows the overlaps file (~5 TB for full human) is stored on a separate/external volume so peak-RAM and disk metrics are not conflated.

## Limitations

- CONSENT's correction step runtime tends to be higher than other state-of-the-art methods, so time profiling must be paired with quality metrics to be meaningful.
- Full human dataset profiling requires ~5 TB of disk for the temporary overlaps file (Alignments_dateTimeStamp.paf), which is easily overlooked on constrained systems.
- Scalability gaps are dataset-dependent: Daccord fails to scale to C. elegans, and windowSize/minAnchors parameter choices shift the runtime-quality trade-off without an established optimum.

## Evidence

- [readme] This should take about 2 min and use up to 750 MB of RAM, using 4 cores.: "This should take about 2 min and use up to 750 MB of RAM, using 4 cores."
- [readme] This should take about 15 sec and use at most 150 MB of RAM, using 4 cores.: "This should take about 15 sec and use at most 150 MB of RAM, using 4 cores."
- [abstract] CONSENT is 2-38x times faster than other polishing tools, while providing comparable results.: "CONSENT is 2-38x times faster than other polishing tools, while providing comparable results."
- [abstract] on a human dataset, assembling the raw data and polishing the assembly is less resource consuming than correcting and then assembling the reads: "on a human dataset, assembling the raw data and polishing the assembly is less resource consuming than correcting and then assembling the reads, while providing better results."
- [discussion] the runtime of CONSENT’s correction step tends to be higher than that of other state-of-the-art methods.: "However, the runtime of CONSENT’s correction step tends to be higher than that of other state-of-the-art methods."
