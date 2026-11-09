---
name: structural-variant-coverage-analysis
description: Use when you have putative >50 kb DEL/DUP calls from a nanopore variant-calling
  pipeline (e.g., Sniffles/CuteSV/SVIM/NanoVar with lra/minimap2/NGMLR alignments)
  and need to filter false positives or check coverage support via read-depth analysis.
when_to_use_negative:
- SV types other than deletions and duplications (e.g., insertions, inversions, translocations)
  — coverage analysis cannot validate these, and disCoverage ignores them
- 'When putative SV positions do not exist yet: disCoverage is not a variant caller
  and needs positions from another technique or multi-sample analysis'
- Regions with highly non-uniform coverage, strong local peaks/valleys, or unresolved
  genome segments where a coverage shift does not reliably indicate a CNV
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_3175
- http://edamontology.org/topic_0622
tools:
- name: disCoverage
  role: 'Performs the coverage-support statistical test: parses the BED/VCF of DEL/DUP
    calls, orchestrates Mosdepth, computes the >=40% relative coverage shift, applies
    the Welch t-test (p < 0.05), and writes support/discard labels'
  repo: https://github.com/javiercguard/disCoverage
- name: Mosdepth
  role: Computes per-base/per-region read depth from the nanopore BAM over each SV
    interval extended by 1-Mb flanking regions
  repo: https://github.com/brentp/mosdepth
- name: minimap2
  role: One of the aligners whose DEL/DUP call sets (alongside lra and NGMLR) are
    the input targets for coverage filtering
provenance:
  source_task_ids:
  - task_005
  - task_004
  source_papers:
  - doi: 10.1016/j.jare.2022.10.012
    title: Improvement of large copy number variant detection by whole genome nanopore
      sequencing
  source_content_hash: sha256:fe5fe3792e5b125899b2bbffc645967cc5381e45434c5675bd7efcbbae99f561
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/javiercguard/disCoverage
  repo_url: https://github.com/javiercguard/disCoverage
  tools:
  - Mosdepth
  - disCoverage
  - minimap2
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_3175
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# structural-variant-coverage-analysis

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

A coverage-based statistical refinement of structural variant calls (deletions/duplications) using the disCoverage tool: per-SV coverage is computed with Mosdepth over the SV interval plus 1-Mb flanks and tested with Welch's t-test to label calls as supported or discarded.

## When to use

Use when you have putative >50 kb DEL/DUP calls from a nanopore variant-calling pipeline (e.g., Sniffles/CuteSV/SVIM/NanoVar with lra/minimap2/NGMLR alignments) and need to filter false positives or check coverage support via read-depth analysis.

## When NOT to use

- SV types other than deletions and duplications (e.g., insertions, inversions, translocations) — coverage analysis cannot validate these, and disCoverage ignores them
- When putative SV positions do not exist yet: disCoverage is not a variant caller and needs positions from another technique or multi-sample analysis
- Regions with highly non-uniform coverage, strong local peaks/valleys, or unresolved genome segments where a coverage shift does not reliably indicate a CNV

## Inputs

- VCF (.vcf or .vcf.gz) or 4-column BED (chr, start, end, SVTYPE) of putative SVs, restricted to DEL/DUP
- Nanopore alignment BAM/CRAM (e.g., NA19240 PromethION sequencing data)
- Optional cached Mosdepth per-region coverage file (<prefix.label.per-region.bed.gz>) for --skip-mosdepth reruns

## Outputs

- One labelled support/discard record per input SV (sv_support_records.tsv) including coverage-shift and p-value support flags
- Per-region coverage files from Mosdepth (e.g., <prefix.label.per-region.bed.gz>) reusable via -cov

## How to apply

Convert the SV call set (VCF or 4-column BED with chr, start, end, SVTYPE) — disCoverage accepts .vcf/.vcf.gz directly and ignores SV types other than DEL and DUP. Run disCoverage on the nanopore BAM (-bam), the SV BED (-bed), an output directory (-o), an analysis label (-i), and >=4 threads (-t); this invokes Mosdepth to compute per-base coverage over each SV's coordinates extended by 1-Mb flanking regions. For each SV, compute the relative coverage shift between the SV interval and its surroundings, then evaluate support with a two-tailed Student's t-test (Welch's approximation) in R, labelling the SV supported when the coverage shift is >=40% and p < 0.05. When repeating across aligners (lra, minimap2, NGMLR on the NA19240 call sets), reuse cached coverage via --skip-mosdepth with -cov pointing at the prior per-region files (<P1.label.per-region.bed.gz>).

## Related tools

- **disCoverage** (Performs the coverage-support statistical test: parses the BED/VCF of DEL/DUP calls, orchestrates Mosdepth, computes the >=40% relative coverage shift, applies the Welch t-test (p < 0.05), and writes support/discard labels) — https://github.com/javiercguard/disCoverage
- **Mosdepth** (Computes per-base/per-region read depth from the nanopore BAM over each SV interval extended by 1-Mb flanking regions) — https://github.com/brentp/mosdepth
- **minimap2** (One of the aligners whose DEL/DUP call sets (alongside lra and NGMLR) are the input targets for coverage filtering)

## Examples

```
disCoverage -i myanalysis -t 8 -o outdir -bam sample.bam -bed svs.bed; disCoverage -i myanalysis2 -t 8 -o outdir2 -bam sample.bam -bed svs2.bed --skip-mosdepth -cov outdir/P1.myanalysis.per-region.bed.gz
```

## Evaluation signals

- sv_support_records.tsv exists in the output directory and contains exactly one labelled row per input SV
- Every SV labelled supported satisfies the >=40% relative coverage shift and p < 0.05 (Welch two-tailed t-test) criteria
- Rerunning on the NA19240 >50-kb call sets should reproduce discarded counts of 83/115/114 calls absent from the gold standard for lra/minimap2/NGMLR
- Approximately 55/61 of the 122 Chaisson et al. truth-set SVs should be coverage-supported on NA19240
- --skip-mosdepth reruns using -cov produce identical support labels as full runs that recompute coverage

## Limitations

- Coverage is not uniform in a sample; local peaks and valleys are frequently observed without pointing to an SV, so coverage shifts can mislabel calls near coverage artifacts
- The p < 0.05 threshold was not confirmed as optimal — the authors note further studies are required to confirm its optimality
- Applicable only to deletions and duplications; other SV types cannot be checked by coverage analysis
- disCoverage is not a variant caller and requires putative SV positions from an external source (another technique or multi-sample analysis)

## Evidence

- [other] disCoverage compares the mean coverage of each putative SV with its surroundings using a threshold of 40% shift in coverage, and checks statistical significance with Student's t-test (Welch's approximation) implemented in R.: [evidence span withheld — anchor status: not_found]
- [other] disCoverage filtering discarded 83, 115, and 114 SVs not present in the gold standard from the lra, minimap2, and NGMLR >50-kb CNV call sets respectively.: [evidence span withheld — anchor status: not_found]
- [other] For repeat runs over the three aligners, reuse cached coverage with --skip-mosdepth, passing the prior per-region coverage files (<P1.label.per-region.bed.gz>) via -cov to avoid recomputation.: [evidence span withheld — anchor status: not_found]
