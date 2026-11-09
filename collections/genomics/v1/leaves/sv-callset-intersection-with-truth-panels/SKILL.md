---
name: sv-callset-intersection-with-truth-panels
description: Use when a filtered SV VCF or BED callset from long-read (MinION/nanopore)
  calling must be quantified for recall and precision against an established reference
  SV panel on a defined region such as NA12878 chromosome 1.
when_to_use_negative:
- Input callset has not passed the post-calling filtering step — intersecting raw
  calls with truth panels produces sensitivity/precision values incomparable with
  the published benchmarks
- Truth and query callsets use inconsistent SV type or breakpoint representations
  (e.g., BND vs. type-specific records), since matching requires agreement on SV type
  and breakpoint proximity
- The goal is de novo SV discovery or phasing rather than benchmarking against an
  established truth panel — the article's novelty assessment (e.g., retrotransposon
  insertions missed by short reads) falls outside this intersection workflow
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: NanoSV
  role: Source of the benchmarked long-read SV callset and of the post-calling filtering
    protocol applied to all callers before intersection
  repo: https://github.com/mroosmalen/nanosv
- name: Lumpy
  role: Comparative SV caller run on the same aligned BAM to generate a comparable
    chromosome 1 VCF for precision/sensitivity comparison
- name: Sniffles
  role: Comparative SV caller run on the same aligned BAM with caller-appropriate
    parameters for the benchmark
- name: bedtools
  role: Intersects the filtered callsets with the 1KG and PacBio truth sets to label
    true positives
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-017-01343-4
    title: Mapping and phasing of structural variation in patient genomes using nanopore
      sequencing
  source_content_hash: sha256:3b5e95f50171f2b25b3a3017edb63727569ed31a021bac1681aadcacc8a3245d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mroosmalen/nanosv
  tools:
  - Lumpy
  - NanoSV
  - Sniffles
  - bedtools
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# SV callset intersection with truth panels

## Summary

Benchmarks structural-variant callsets (e.g., NanoSV, Lumpy, Sniffles VCFs derived from MinION nanopore reads) against public truth panels such as the 1000 Genomes (1KG) Illumina and PacBio (PB) SV call sets by intersecting calls and matching events by SV type and breakpoint proximity, then computing per-caller sensitivity and precision.

## When to use

Use when a filtered SV VCF or BED callset from long-read (MinION/nanopore) calling must be quantified for recall and precision against an established reference SV panel on a defined region such as NA12878 chromosome 1.

## When NOT to use

- Input callset has not passed the post-calling filtering step — intersecting raw calls with truth panels produces sensitivity/precision values incomparable with the published benchmarks
- Truth and query callsets use inconsistent SV type or breakpoint representations (e.g., BND vs. type-specific records), since matching requires agreement on SV type and breakpoint proximity
- The goal is de novo SV discovery or phasing rather than benchmarking against an established truth panel — the article's novelty assessment (e.g., retrotransposon insertions missed by short reads) falls outside this intersection workflow

## Inputs

- Coordinate-sorted, filtered SV callset VCF (NanoSV output; comparably produced Lumpy and Sniffles VCFs) restricted to the benchmark region (e.g., NA12878 chromosome 1)
- Public 1000 Genomes (1KG) Illumina SV truth callset for the sample/region
- Public PacBio (PB) SV truth callset for NA12878
- Post-calling filtering parameters from the NanoSV protocol

## Outputs

- Per-caller sensitivity and precision table (sv_benchmark_na12878_chr1.tsv) covering NanoSV, Lumpy, Sniffles and the combined callset
- Per-event intersection results indicating which truth-panel SVs (1KG or PB) were recovered by each caller

## How to apply

Convert both the caller-derived callset and the truth panels (1KG Illumina and PacBio SV sets) to compatible intervals retaining SV type and breakpoints, then intersect them with bedtools while requiring agreement on SV type and proximity of breakpoints. Compute sensitivity as the fraction of truth-panel SVs recovered by the caller (e.g., NanoSV recovered 78% of 1KG SVs, 131/168, and 88% of PB SVs, 292/332, on NA12878 chr1) and precision as the fraction of caller calls matching truth (NanoSV 54%, Lumpy 2%, Sniffles 50% in the reported benchmark). Apply the NanoSV protocol's post-calling filtering to all three callsets before intersection so comparisons are like-for-like; the paper notes that extremely stringent filtering is deliberately traded against sensitivity, yielding ~75% estimated sensitivity genome-wide. Write per-caller and combined sensitivity/precision summaries to a TSV and check the values against the published benchmarks before trusting the run.

## Related tools

- **NanoSV** (Source of the benchmarked long-read SV callset and of the post-calling filtering protocol applied to all callers before intersection) — https://github.com/mroosmalen/nanosv
- **Lumpy** (Comparative SV caller run on the same aligned BAM to generate a comparable chromosome 1 VCF for precision/sensitivity comparison)
- **Sniffles** (Comparative SV caller run on the same aligned BAM with caller-appropriate parameters for the benchmark)
- **bedtools** (Intersects the filtered callsets with the 1KG and PacBio truth sets to label true positives)

## Examples

```
bedtools intersect -u -a 1KG_chr1_SVs.bed -b NanoSV_chr1_filtered.bed > NanoSV_chr1_1KG_tp.bed
```

## Evaluation signals

- Sensitivity against 1KG truth on NA12878 chr1 nanopore data should approach 78% (131 of 168 SVs) for NanoSV
- Sensitivity against PacBio truth should approach 88% (292 of 332 SVs) for NanoSV
- Precision should approximate NanoSV 54%, Lumpy 15%/32% context (reported 2% for Lumpy) and Sniffles 50% for the same BAM and filtering
- Every truth-panel event counted as recovered must match the caller's event by SV type and breakpoint proximity — verify no type-mismatched overlaps were counted
- The sensitivity/precision TSV must include per-caller and combined callset rows, all computed on identically filtered callsets

## Limitations

- Extremely stringent post-calling filtering is intentionally conservative, lowering the estimated sensitivity (~75% genome-wide) in exchange for state-of-the-art precision (>95% in the paper's own pipeline benchmark)
- Benchmarking is limited by medium coverage and intrinsic nanopore sequencing biases, so truth-panel recall underestimates achievable sensitivity at higher depths
- Comparison is region-specific (NA12878 chromosome 1) against 1KG and PB reference sets; results do not automatically generalize genome-wide or to other samples

## Evidence

- [results] task_002 benchmark finding: [evidence span withheld — anchor status: not_found]
- [other] workflow intersection criterion: [evidence span withheld — anchor status: not_found]
- [discussion] filtering vs sensitivity tradeoff: "we employ extremely stringent filtering that results in a good estimated sensitivity (~75%)"
- [discussion] pipeline precision benchmark: [evidence span withheld — anchor status: not_found]
- [discussion] truth panel used: "as well as comparison to 1KG variant sites"
