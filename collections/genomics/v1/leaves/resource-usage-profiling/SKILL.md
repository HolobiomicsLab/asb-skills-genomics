---
name: resource-usage-profiling
description: Use when benchmarking long-read correction or polishing methods on real
  datasets and you need to record wall-clock runtime and peak RAM alongside corrected-read
  quality metrics to compare scalability.
when_to_use_negative:
- When evaluating only simulated datasets where the paper's scaling claims (e.g.,
  ultra-long ONT reads up to 1.5 Mbp) do not apply
- When runtime/memory are not comparable because tools are run with different core
  counts or thread settings
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_0102
- http://edamontology.org/topic_0622
tools:
- name: CONSENT
  role: Self-correction tool being profiled; run with --type PB or --type ONT and
    default parameters; --nproc controls parallelism used during timing
  repo: https://github.com/morispi/CONSENT
- name: Canu
  role: Competing self-correction tool run on the same datasets with its recommended
    real-read correction recipe to compare runtime and memory
- name: FLAS
  role: Competing self-correction tool run on the same datasets to compare runtime
    and memory
- name: MECAT
  role: Competing self-correction tool run on the same datasets to compare runtime
    and memory
- name: minimap2
  role: Alignment engine underlying ELECTOR's reference-alignment module used to compute
    quality metrics reported alongside runtime/memory
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41598-020-80757-5
    title: Scalable long read self-correction and assembly polishing with multiple
      sequence alignment
  source_content_hash: sha256:4bc091ce41f78f20f8700a1d3531eb4051aa0353f6feaa52429bd10419b34b9d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/morispi/CONSENT
  tools:
  - CONSENT
  - Canu
  - FLAS
  - MECAT
  - minimap2
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_0102
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6923076923076923
n_claims: 13
---

# Resource-usage profiling

## Summary

Profiling the wall-clock runtime and peak memory consumption of long-read self-correction tools (CONSENT, Canu, FLAS, MECAT) on real PacBio/ONT datasets so that correction quality results can be interpreted alongside scalability, as in the CONSENT benchmark's Tables measuring runtime and memory.

## When to use

Use when benchmarking long-read correction or polishing methods on real datasets and you need to record wall-clock runtime and peak RAM alongside corrected-read quality metrics to compare scalability.

## When NOT to use

- When evaluating only simulated datasets where the paper's scaling claims (e.g., ultra-long ONT reads up to 1.5 Mbp) do not apply
- When runtime/memory are not comparable because tools are run with different core counts or thread settings

## Inputs

- Real long-read FASTA/FASTQ datasets: S. cerevisiae PacBio reads (SRR9617898), D. melanogaster ONT reads (SRX3676783), H. sapiens chr 1 ONT reads (PRJEB23027)
- Correction tool executables: CONSENT, Canu, FLAS, MECAT
- Timing/memory measurement wrapper (e.g., /usr/bin/time -v or equivalent)

## Outputs

- Per-tool, per-dataset wall-clock runtime measurements
- Per-tool, per-dataset peak memory (RAM) measurements
- A benchmark table combining runtime and memory with corrected-read quality metrics (reads, bases, N50, aligned-read proportion, identity, genome coverage)

## How to apply

Run each correction tool (e.g., CONSENT via ./CONSENT-correct --in longReads --out result.fasta --type PB or --type ONT) on the same real long-read datasets — S. cerevisiae PacBio (SRR9617898), D. melanogaster ONT (SRX3676783), and H. sapiens chr 1 ONT (PRJEB23027) — with recommended default parameters (CONSENT defaults: windowSize 500, minSupport 4, maxSupport 150, merSize 9, solid 4, anchorSupport 8, minAnchors 2, windowOverlap 50). Wrap each run with time and memory measurement so wall-clock runtime and peak memory are recorded per tool per dataset. Use identical core counts across tools (CONSENT's --nproc defaults to the number of cores) so comparisons are fair. Report runtime and memory next to ELECTOR/minimap2-derived quality metrics (corrected reads, bases, N50, alignment identity, genome coverage) in a single table. Interpret jointly: CONSENT may have higher correction-step runtime yet still correct the most reads with the largest genome coverage.

## Related tools

- **CONSENT** (Self-correction tool being profiled; run with --type PB or --type ONT and default parameters; --nproc controls parallelism used during timing) — https://github.com/morispi/CONSENT
- **Canu** (Competing self-correction tool run on the same datasets with its recommended real-read correction recipe to compare runtime and memory)
- **FLAS** (Competing self-correction tool run on the same datasets to compare runtime and memory)
- **MECAT** (Competing self-correction tool run on the same datasets to compare runtime and memory)
- **minimap2** (Alignment engine underlying ELECTOR's reference-alignment module used to compute quality metrics reported alongside runtime/memory)

## Examples

```
/usr/bin/time -v ./CONSENT-correct --in longReads.fastq --out result.fasta --type ONT --nproc 4
```

## Evaluation signals

- Each tool-dataset combination in the benchmark table has both a runtime value and a peak-memory value, matching the eight Table 2 metrics
- All runs used the same datasets (SRR9617898, SRX3676783, PRJEB23027) and equivalent core counts, so runtime differences reflect tool efficiency
- CONSENT's measured runtime is interpretable against the article's claim that it is 'almost four times faster than Canu' on human chr 1 and processes a full human dataset in 10 days
- Runtime and memory values are consistent with the quality metrics produced from the same corrected-reads files (e.g., CONSENT outputting the most reads/bases and largest genome coverage)
- CONSENT runs used its documented defaults (windowSize 500, minSupport 4, maxSupport 150, merSize 9, solid 4, anchorSupport 8, minAnchors 2, windowOverlap 50) so timings reflect reproducible configurations

## Limitations

- CONSENT's correction step runtime tends to be higher than other state-of-the-art methods
- Correcting a full human dataset requires about 5 TB of disk space, driven mainly by the overlaps file
- Window size and minimum number of anchors parameters significantly impact runtime; the optimal runtime/quality compromise has not been established
- CONSENT is the only method that efficiently scales to ultra-long reads, so competing tools may fail or require excessive resources on ultra-long ONT data

## Evidence

- [discussion] CONSENT's correction-step runtime is higher than other state-of-the-art methods, with context: "However, the runtime of CONSENT’s correction step tends to be higher than that of other state-of-the-art methods."
- [discussion] 5 TB disk space for full human dataset: "The 5 TB disk space requirement comes from the overlaps file, which can easily be stored on a external hard drive."
- [abstract] CONSENT scales to full human dataset in 10 days: "CONSENT is the only method that efficiently scales to ultra-long reads, and allows to process a full human dataset, containing reads reaching up to 1.5 Mbp, in 10 days."
- [discussion] CONSENT almost four times faster than Canu on human chr 1: "on the human chr 1 dataset, CONSENT is almost four times faster than Canu"
- [readme] README: nproc and minimap index options affect runtime: "--nproc INT, -j INT:           Number of processes to run in parallel (default: number of cores)."
