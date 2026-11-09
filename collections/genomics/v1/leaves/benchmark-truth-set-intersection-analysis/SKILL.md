---
name: benchmark-truth-set-intersection-analysis
description: Use when benchmarking a somatic SV caller against published high-confidence
  truth sets on COLO829, needing precision/recall from breakpoint-overlap matching
  of call sets to truth VCFs/bedpe files.
when_to_use_negative:
- The goal is germline SV calling or single-sample SV detection rather than comparing
  a somatic caller's output to a matched-normal-controlled somatic truth set
- Truth sets contain many short insertions/deletions ≤100 bp that the caller's SV
  definition does not cover — these must be filtered out first or precision/recall
  will be distorted
- Caller output and truth sets are in incompatible genome builds and no liftOver step
  is possible (e.g. no chain file), making breakpoint-overlap intersection invalid
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3175
tools:
- name: nanomonsv
  role: Produces the somatic SV call set (nanomonsv.result.txt/.vcf) that is benchmarked
    against truth sets
  repo: https://github.com/friend1ws/nanomonsv
- name: liftOver
  role: Converts the Valle-Inclan truth set VCF to GRCh38 coordinates before intersection
- name: minimap2
  role: Aligns tumor and control ONT/PacBio reads to GRCh38 to generate BAMs from
    which caller output is derived
  repo: https://github.com/lh3/minimap2
- name: samtools
  role: Sorts and indexes the minimap2-aligned BAMs prior to calling
- name: SURVIVOR
  role: Tool set for merging and comparing SVs within and among samples, useful for
    SV call set comparison
  repo: https://github.com/fritzsedlazeck/SURVIVOR
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/nar/gkad526
    title: Precise characterization of somatic complex structural variations from
      tumor/control paired long-read sequencing data with nanomonsv
  source_content_hash: sha256:10c436cbd243e12be7d3cfc17316cb58fd4bdef9e925b864688a402256a75e00
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
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3175
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# benchmark-truth-set-intersection-analysis

## Summary

Evaluates somatic SV caller performance by intersecting caller output (e.g. nanomonsv.result.txt/.vcf from COLO829 ONT data) with high-confidence truth sets (Arora bedpe, Valle-Inclan truthset_somaticSVs_COLO829.vcf) via breakpoint overlap to compute precision and recall.

## When to use

Use when benchmarking a somatic SV caller against published high-confidence truth sets on COLO829, needing precision/recall from breakpoint-overlap matching of call sets to truth VCFs/bedpe files.

## When NOT to use

- The goal is germline SV calling or single-sample SV detection rather than comparing a somatic caller's output to a matched-normal-controlled somatic truth set
- Truth sets contain many short insertions/deletions ≤100 bp that the caller's SV definition does not cover — these must be filtered out first or precision/recall will be distorted
- Caller output and truth sets are in incompatible genome builds and no liftOver step is possible (e.g. no chain file), making breakpoint-overlap intersection invalid

## Inputs

- nanomonsv somatic SV call set (nanomonsv.result.txt / .vcf) from tumor/control paired long-read data
- Arora high-confidence somatic SV bedpe (NY Genome high-coverage NovaSeq resource)
- Valle-Inclan truthset_somaticSVs_COLO829.vcf (Zenodo record 3988185, hg38-to-GRCh38 lifted)
- Tumor/control BAMs aligned to GRCh38 with minimap2 (ONT ERR2752451/ERR2752452; PacBio ERR2808247/ERR2808248, ENA PRJEB27698)

## Outputs

- benchmark_precision_recall.csv with per-benchmark precision and recall values
- True positive counts from breakpoint-overlap intersection (e.g. 43/49 for both Arora and Valle-Inclan)
- False positive counts of caller-novel SVs (6 for Arora, 4 for Valle-Inclan)

## How to apply

Run the caller (nanomonsv parse then get) on tumor/control long-read BAMs — e.g. COLO829 ONT (ERR2752451/ERR2752452) and PacBio (ERR2808247/ERR2808248) from ENA PRJEB27698 aligned to GRCh38 with minimap2 — to obtain somatic SV call sets. Download the Arora high-confidence somatic SV bedpe (NY Genome high-coverage NovaSeq resource) and the Valle-Inclan truthset_somaticSVs_COLO829.vcf from Zenodo record 3988185, lifting the latter to GRCh38 with liftOver. In both truth sets remove insertions and deletions of ≤100 bp, then intersect caller calls with each truth set by breakpoint overlap. Count overlapping calls as true positives, treat novel caller SVs (6 for Arora, 4 for Valle-Inclan) as false positives, and compute precision and recall per benchmark (worst-case 87.8% (43/49) and 91.8% (45/49) precision; 57.3% (43/75) and 77.6% (46/58) recall), writing results to benchmark_precision_recall.csv.

## Related tools

- **nanomonsv** (Produces the somatic SV call set (nanomonsv.result.txt/.vcf) that is benchmarked against truth sets) — https://github.com/friend1ws/nanomonsv
- **liftOver** (Converts the Valle-Inclan truth set VCF to GRCh38 coordinates before intersection)
- **minimap2** (Aligns tumor and control ONT/PacBio reads to GRCh38 to generate BAMs from which caller output is derived) — https://github.com/lh3/minimap2
- **samtools** (Sorts and indexes the minimap2-aligned BAMs prior to calling)
- **SURVIVOR** (Tool set for merging and comparing SVs within and among samples, useful for SV call set comparison) — https://github.com/fritzsedlazeck/SURVIVOR

## Examples

```
minimap2 -ax map-ont GRCh38.fa tumor.fq control.fq | samtools sort -o aln.bam; nanomonsv parse; nanomonsv get; python intersect_calls_vs_truth.py --calls nanomonsv.result.vcf --truth truthset_somaticSVs_COLO829.vcf --out benchmark_precision_recall.csv
```

## Evaluation signals

- Reproduced precision values match 87.8% (43/49) against Arora and 91.8% (45/49) against Valle-Inclan, as reported in Figure 3E
- Reproduced recall values match 57.3% (43/75) against Arora and 77.6% (46/58) against Valle-Inclan
- True positive counts are 43 for both benchmarks (49 nanomonsv calls), and false positives from novel nanomonsv SVs are 6 (Arora) and 4 (Valle-Inclan)
- Truth sets used are the Arora benchmark with 75 SVs and the Valle-Inclan benchmark with 58 SVs after removing insertions/deletions of ≤100 bp
- Valle-Inclan truth set VCF is confirmed to be in GRCh38 coordinates after liftOver before intersection

## Limitations

- Caller-novel SVs (not present in either truth set) are conservatively treated as false positives, yielding 'worst-case' precision — some may be real somatic SVs missed by the truth sets
- Truth sets are restricted to SVs capturable by short-read technologies; SVs in highly repetitive regions (e.g. centromeres, LINE1-mediated rearrangements) may be under-represented in benchmarks
- Somatic VNTR/microsatellite repeat expansion events are not addressed; the reference-genome-based framework may not capture long repeat expansion events, limiting what such benchmarks can assess
- Benchmarking against COLO829 truth sets does not evaluate single breakend SVs or mobile element insertions detectable only by long reads

## Evidence

- [results] Against the Arora and Valle-Inclan benchmarks (75 and 58 SVs respectively), nanomonsv on COLO829 ONT data achieved worst-case precision of 87.8% (43/49) and 91.8% (45/49), and recall of 57.3% (43/75) and 77.6% (46/58).: [evidence span withheld — anchor status: not_found]
- [methods] Download the Valle-Inclan truthset_somaticSVs_COLO829.vcf from Zenodo record 3988185, converting the latter to GRCh38 coordinates with liftOver, and in both truth sets remove insertions and deletions of ≤100 bp.: [evidence span withheld — anchor status: not_found]
- [methods] Intersect nanomonsv calls with each truth set by breakpoint overlap; count true positives, and treat novel nanomonsv SVs (6 for Arora, 4 for Valle-Inclan) as false positives to compute precision and recall.: [evidence span withheld — anchor status: not_found]
- [abstract] The Canonical SV module identifies somatic SVs captured by short-read technologies with higher precision and recall than existing methods.: "we demonstrate that Canonical SV module can identify somatic SVs that can be captured by short-read technologies with higher precision and recall than existing methods"
- [readme] nanomonsv is a software for detecting somatic structural variations from paired (tumor and matched control) cancer genome sequence data.: "nanomonsv is a software for detecting somatic structural variations from paired (tumor and matched control) cancer genome sequence data."
