---
name: cross-tool-correction-benchmarking
description: Use when comparing self-correction tools on real long-read datasets (PacBio
  S. cerevisiae, ONT D. melanogaster, ONT H. sapiens chr 1) and need a reproducible
  table of correction quality and resource metrics.
when_to_use_negative:
- Simulated datasets only — the article's real-data benchmark (Table 2) targets real
  PacBio and ONT reads; simulated-read comparisons use different evaluation machinery
  (ELECTOR's simulation module)
- Assembly polishing evaluation — this skill benchmarks self-correction only, not
  CONSENT's polishing feature
- Datasets lacking a reference genome, since genome coverage and alignment identity
  require aligning corrected reads back to a reference
edam_operation: http://edamontology.org/operation_3182
edam_topics:
- http://edamontology.org/topic_0769
- http://edamontology.org/topic_0622
tools:
- name: CONSENT
  role: Self-correction tool under benchmark; run with ./CONSENT-correct using --type
    PB or --type ONT on the three real datasets
  repo: https://github.com/morispi/CONSENT
- name: minimap2
  role: Aligner underlying ELECTOR's reference-alignment module, used to align corrected
    reads to reference genomes with index split every 500M bases
- name: ELECTOR
  role: Evaluation framework that aligns each tool's corrected reads back to the reference
    and extracts alignment statistics (identity, coverage, N50)
- name: Canu
  role: Competing self-correction tool run with its recommended real-read correction
    recipe
- name: FLAS
  role: Competing self-correction tool run with its recommended real-read correction
    recipe
- name: MECAT
  role: Competing self-correction tool run with its recommended real-read correction
    recipe
- name: Daccord
  role: Competing self-correction tool included in the real-data benchmark
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41598-020-80757-5
    title: Scalable long read self-correction and assembly polishing with multiple
      sequence alignment
  source_content_hash: sha256:84381f794507a817fc9a3a92d87de1b1803008fd5cba84d6d3236194aa10e9dd
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
  - MECAT
  - minimap2
  edam_operation: http://edamontology.org/operation_3182
  edam_topics:
  - http://edamontology.org/topic_0769
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 15
---

# cross-tool-correction-benchmarking

## Summary

Benchmark long-read self-correction methods (CONSENT vs. Canu, Daccord, FLAS, MECAT) on real PacBio and ONT datasets by running each correction recipe, aligning corrected reads back to the reference with ELECTOR/minimap2, and compiling Table 2 metrics (corrected reads, bases, N50, aligned-read proportion, identity, genome coverage, runtime, memory).

## When to use

Use when comparing self-correction tools on real long-read datasets (PacBio S. cerevisiae, ONT D. melanogaster, ONT H. sapiens chr 1) and need a reproducible table of correction quality and resource metrics.

## When NOT to use

- Simulated datasets only — the article's real-data benchmark (Table 2) targets real PacBio and ONT reads; simulated-read comparisons use different evaluation machinery (ELECTOR's simulation module)
- Assembly polishing evaluation — this skill benchmarks self-correction only, not CONSENT's polishing feature
- Datasets lacking a reference genome, since genome coverage and alignment identity require aligning corrected reads back to a reference

## Inputs

- S. cerevisiae PacBio long reads (SRR9617898) in FASTA/FASTQ
- D. melanogaster ONT long reads (SRX3676783) in FASTA/FASTQ
- H. sapiens chromosome 1 ONT long reads (PRJEB23027) in FASTA/FASTQ
- Reference genomes for each of the three organisms
- Corrected-reads FASTA files produced by CONSENT, Canu, Daccord, FLAS, and MECAT

## Outputs

- Corrected-reads FASTA per tool per dataset
- ELECTOR/minimap2 reference alignments of corrected reads
- Table 2-style benchmark table: reads corrected, total bases, N50, aligned-reads proportion, alignment identity, genome coverage, runtime, and peak memory per tool per dataset

## How to apply

Download the three real datasets (S. cerevisiae PacBio reads SRR9617898, D. melanogaster ONT reads SRX3676783, H. sapiens chr 1 ONT reads PRJEB23027) with their reference genomes. Run CONSENT self-correction on each with ./CONSENT-correct --in longReads --out result.fasta, passing --type PB for PacBio and --type ONT for Nanopore data, keeping defaults (windowSize 500, minSupport 4, maxSupport 150, merSize 9, solid 4, anchorSupport 8, minAnchors 2, windowOverlap 50). Run the competing self-correction tools Canu, FLAS, and MECAT with their recommended real-read correction recipes, recording wall-clock runtime and peak memory per run. Align each tool's corrected reads to the matching reference with ELECTOR's reference-alignment module (minimap2-based, index split every 500M bases by default), then extract corrected-read counts, total bases, N50, aligned-read proportion, alignment identity, and genome coverage into a single Table 2-style benchmark table.

## Related tools

- **CONSENT** (Self-correction tool under benchmark; run with ./CONSENT-correct using --type PB or --type ONT on the three real datasets) — https://github.com/morispi/CONSENT
- **minimap2** (Aligner underlying ELECTOR's reference-alignment module, used to align corrected reads to reference genomes with index split every 500M bases)
- **ELECTOR** (Evaluation framework that aligns each tool's corrected reads back to the reference and extracts alignment statistics (identity, coverage, N50))
- **Canu** (Competing self-correction tool run with its recommended real-read correction recipe)
- **FLAS** (Competing self-correction tool run with its recommended real-read correction recipe)
- **MECAT** (Competing self-correction tool run with its recommended real-read correction recipe)
- **Daccord** (Competing self-correction tool included in the real-data benchmark)

## Examples

```
./CONSENT-correct --in longReads.fasta --out result.fasta --type ONT
```

## Evaluation signals

- The reproduced table reports all eight Table 2 metrics (reads corrected, bases, N50, aligned-reads proportion, alignment identity, genome coverage, runtime, memory) for every tool on all three datasets
- Each reported value is computable from the produced corrected-reads FASTA files and ELECTOR alignment output
- CONSENT is expected to correct the largest number of reads, output the most bases, and reach the largest genome coverage across all three real datasets
- Runtime and peak memory are recorded per tool run and CONSENT is expected to be nearly 4x faster than Canu on the human chr 1 dataset
- CONSENT defaults are preserved (windowSize 500, minSupport 4, maxSupport 150, merSize 9, solid 4, anchorSupport 8, minAnchors 2, windowOverlap 50) so runs are reproducible

## Limitations

- The runtime of CONSENT's correction step tends to be higher than that of other state-of-the-art methods, though it is faster than Canu on human chr 1
- CONSENT requires ~5 TB of disk space for a full human dataset, driven by the temporary overlaps file (Alignments_dateTimeStamp.paf)
- Only chromosome 1 of H. sapiens is benchmarked in the real-data comparison; larger, more complex organisms were not corrected
- CONSENT has been developed and tested on x86-64 GNU/Linux only; requires minimap2 on the PATH

## Evidence

- [abstract] CONSENT compares well to the state-of-the-art, and performs better on real Oxford Nanopore data.: "CONSENT compares well to the state-of-the-art, and performs better on real Oxford Nanopore data."
- [discussion] human chr 1 speed comparison: "on the human chr 1 dataset, CONSENT is almost four times faster than Canu"
- [readme] CONSENT correction invocation: "`./CONSENT-correct --in longReads.fast[a|q] --out result.fasta --type readsTechnology`"
- [readme] minimap index split parameter: "--minimapIndex INT, -m INT:    Split minimap2 index every INT input bases (default: 500M)."
