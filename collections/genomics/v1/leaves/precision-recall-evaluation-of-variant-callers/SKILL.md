---
name: precision-recall-evaluation-of-variant-callers
description: Use when a newly generated SV call set (e.g., from tumor/control long-read
  data) must be quantified as true/false positives against a published high-confidence
  somatic SV truth set to report precision and recall.
when_to_use_negative:
- The caller output is germline-only or single-sample, with no matched control — the
  somatic truth-set comparison does not model germline contamination.
- The truth set is not high-confidence or contains events ≤100 bp that were not filtered,
  since short indels are excluded from both benchmarks by design.
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_0102
- http://edamontology.org/topic_0622
tools:
- name: nanomonsv
  role: Caller under evaluation; run as parse followed by get on tumor/control long-read
    BAM pairs to produce the somatic SV call set
  repo: https://github.com/friend1ws/nanomonsv
- name: minimap2
  role: Aligns COLO829/COLO829BL ONT and PacBio FASTQ reads to GRCh38 before SV calling
  repo: https://github.com/lh3/minimap2
- name: samtools
  role: Sorts and indexes the aligned tumor and control BAMs
  repo: https://github.com/samtools/samtools
- name: liftOver
  role: Converts the Valle-Inclan truth set to GRCh38 coordinates
- name: SURVIVOR
  role: Tool set for comparing/merging SV call sets across samples, applicable to
    the breakpoint-overlap matching step
  repo: https://github.com/fritzsedlazeck/SURVIVOR
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/nar/gkad526
    title: Precise characterization of somatic complex structural variations from
      tumor/control paired long-read sequencing data with nanomonsv
  source_content_hash: sha256:ee1fd1c9ae5e7eb9585b9204a6ff4e8b6f928f572afb2ca2d6bb3d7a3e29dae7
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/friend1ws/nanomonsv
  tools:
  - PacBio
  - SURVIVOR
  - liftOver
  - minimap2
  - nanomonsv
  - samtools
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_0102
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# precision-recall-evaluation-of-variant-callers

## Summary

Benchmarks somatic SV caller output against curated high-confidence truth sets by breakpoint overlap to compute precision and recall, as done for nanomonsv on COLO829 against the Arora and Valle-Inclan somatic SV benchmarks.

## When to use

Use when a newly generated SV call set (e.g., from tumor/control long-read data) must be quantified as true/false positives against a published high-confidence somatic SV truth set to report precision and recall.

## When NOT to use

- The caller output is germline-only or single-sample, with no matched control — the somatic truth-set comparison does not model germline contamination.
- The truth set is not high-confidence or contains events ≤100 bp that were not filtered, since short indels are excluded from both benchmarks by design.

## Inputs

- Somatic SV call set from the caller under evaluation (e.g., nanomonsv.result.txt / .vcf from ONT and PacBio tumor/control pairs)
- Curated truth set in bedpe format (Arora high-confidence somatic SV bedpe)
- Curated truth set in VCF format (Valle-Inclan truthset_somaticSVs_COLO829.vcf, converted to GRCh38 with liftOver)

## Outputs

- Per-benchmark precision and recall values (benchmark_precision_recall.csv)
- Counts of true positives, false positives (novel calls), and truth-set size (e.g., 43/49 TP-precision, 43/75 TP-recall)

## How to apply

Run the caller under evaluation (nanomonsv parse followed by nanomonsv get) on tumor/control BAM pairs (e.g., COLO829/COLO829BL ONT and PacBio reads aligned to GRCh38 with minimap2), producing call sets such as nanomonsv.result.txt/.vcf. Harmonize coordinates and formats: convert the Valle-Inclan truthset_somaticSVs_COLO829.vcf (Zenodo record 3988185) to GRCh38 with liftOver, obtain the Arora bedpe from the NY Genome NovaSeq resource, and in both truth sets remove insertions and deletions of ≤100 bp. Intersect the call set with each truth set by breakpoint overlap, counting matches as true positives (43/75 and 46/58 for Arora/Valle-Inclan respectively) and treating novel, unmatched calls (6 for Arora, 4 for Valle-Inclan) as false positives. Compute precision as TP/(TP+FP) and recall as TP/truth-set-size (worst-case values 87.8% and 91.8% precision, 57.3% and 77.6% recall) and write per-benchmark values to a CSV such as benchmark_precision_recall.csv.

## Related tools

- **nanomonsv** (Caller under evaluation; run as parse followed by get on tumor/control long-read BAM pairs to produce the somatic SV call set) — https://github.com/friend1ws/nanomonsv
- **minimap2** (Aligns COLO829/COLO829BL ONT and PacBio FASTQ reads to GRCh38 before SV calling) — https://github.com/lh3/minimap2
- **samtools** (Sorts and indexes the aligned tumor and control BAMs) — https://github.com/samtools/samtools
- **liftOver** (Converts the Valle-Inclan truth set to GRCh38 coordinates)
- **SURVIVOR** (Tool set for comparing/merging SV call sets across samples, applicable to the breakpoint-overlap matching step) — https://github.com/fritzsedlazeck/SURVIVOR

## Examples

```
minimap2 -ax map-ont GRCh38.fa ERR2752451.fq | samtools sort -o tumor.bam && nanomonsv parse tumor.bam control.bam colo829_t; nanomonsv get --prefix colo829_t colo829_somatic.vcf
```

## Evaluation signals

- Reproduced precision matches 87.8% (43/49) on the Arora benchmark and 91.8% (45/49) on the Valle-Inclan benchmark as reported in Figure 3E.
- Reproduced recall matches 57.3% (43/75) and 77.6% (46/58) as reported in Figure 3E.
- Novel caller SVs (6 for Arora, 4 for Valle-Inclan) are explicitly enumerated as false positives rather than silently dropped.
- Both truth sets exclude insertions and deletions of ≤100 bp before comparison; the Valle-Inclan set has been liftOver-converted to GRCh38.

## Limitations

- Caller-specific novel SVs are treated as false positives, but some may be true somatic events absent from the benchmark (underestimating precision).
- Detection of subclonal SVs present in a minority of cells is still challenging with a modest sequencing depth.
- There has been little evaluation on the resolution of breakpoints of SVs using noisy long-read sequencing data; the current reference-genome-based framework may not capture long repeat expansion events.

## Evidence

- [full_text] worst-case precision and recall reproduced: [evidence span withheld — anchor status: not_found]
- [abstract] precision/recall vs existing methods: "we demonstrate that Canonical SV module can identify somatic SVs that can be captured by short-read technologies with higher precision and recall than existing methods"
- [intro] paired tumor/control calling premise: "identify somatic SVs with single-nucleotide resolution jointly using both tumor and control long-read sequencing data"
- [readme] tool description: "nanomonsv is a software for detecting somatic structural variations from paired (tumor and matched control) cancer genome sequence data"
