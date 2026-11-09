---
name: truth-set-concordance-evaluation
description: Use when nanopore >50-kb CNV call sets (lra/minimap2/NGMLR pipelines)
  must be polished and benchmarked against a well-characterized truth set such as
  NA19240/Chaisson, using coverage support as the filtering criterion.
when_to_use_negative:
- The call set contains SV types other than deletions and duplications (e.g., INV,
  INS, BND) — disCoverage's coverage test can only be applied to DEL/DUP
- No putative SV list exists and de novo calling is the goal — disCoverage is not
  a variant caller and needs putative SV positions from another technique or multi-sample
  analysis
- Coverage in the sample is too non-uniform (local peaks/valleys, unresolved regions)
  for the coverage-difference test to discriminate true from false calls
edam_operation: http://edamontology.org/operation_0284
edam_topics:
- http://edamontology.org/topic_0625
tools:
- name: disCoverage
  role: Runs the coverage-difference statistical test on each DEL/DUP against the
    NA19240 BAM to flag coverage support and filter calls not present in the truth
    set
  repo: https://github.com/javiercguard/disCoverage
- name: Mosdepth
  role: Invoked by disCoverage to compute mean per-region coverage for every queried
    SV interval
  repo: https://github.com/brentp/mosdepth
- name: nanopore_pipeline_21
  role: Pipeline producing the lra/minimap2/NGMLR >50-kb DEL/DUP call sets that are
    evaluated against the truth set
  repo: https://github.com/javiercguard/nanopore_pipeline_21
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1016/j.jare.2022.10.012
    title: Improvement of large copy number variant detection by whole genome nanopore
      sequencing
  source_content_hash: sha256:04d5def0215429f38ff4c3e2c621cd3a541f7d9ddce3c75edacc9cb6fbd13a43
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
  - nanopore_pipeline_21
  edam_operation: http://edamontology.org/operation_0284
  edam_topics:
  - http://edamontology.org/topic_0625
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# truth-set-concordance-evaluation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Evaluates >50-kb DEL/DUP nanopore SV call sets against a gold-standard truth set (NA19240 with the Chaisson et al. set) by running disCoverage's coverage-difference test per aligner call set, discarding calls lacking coverage support and checking how many truth-set SVs are coverage-supported.

## When to use

Use when nanopore >50-kb CNV call sets (lra/minimap2/NGMLR pipelines) must be polished and benchmarked against a well-characterized truth set such as NA19240/Chaisson, using coverage support as the filtering criterion.

## When NOT to use

- The call set contains SV types other than deletions and duplications (e.g., INV, INS, BND) — disCoverage's coverage test can only be applied to DEL/DUP
- No putative SV list exists and de novo calling is the goal — disCoverage is not a variant caller and needs putative SV positions from another technique or multi-sample analysis
- Coverage in the sample is too non-uniform (local peaks/valleys, unresolved regions) for the coverage-difference test to discriminate true from false calls

## Inputs

- Merged >50-kb DEL/DUP SV call sets in VCF format, one per aligner (lra, minimap2, NGMLR)
- Chaisson et al. truth-set SVs for NA19240 (122 SVs, converted to 4-column BED)
- NA19240 nanopore (PromethION) alignment BAM
- Cached per-region coverage files (<P1.label.per-region.bed.gz>) for --skip-mosdepth reruns

## Outputs

- Per-call retained/discarded status with coverage-support flags for every SV, separately for lra, minimap2, and NGMLR
- Coverage-support status for the 122 truth-set SVs
- Per-region coverage files (<prefix.label.per-region.bed.gz>) from Mosdepth
- Counts of discarded false positives (83/115/114) and specificity gains per aligner

## How to apply

Convert each aligner's merged >50-kb DEL/DUP VCF call set and the Chaisson et al. truth-set SVs into 4-column BED (chr, start, end, SVTYPE); disCoverage accepts .vcf/.vcf.gz directly. Run disCoverage once per aligner (lra, minimap2, NGMLR) with the NA19240 nanopore BAM (-bam), the SV BED (-bed), an output directory (-o), an analysis label (-i), and at least 4 threads (-t), which invokes Mosdepth to compute per-region coverage for every queried SV. Apply the statistical coverage-difference test with a p-value threshold smaller than 0.05 to flag each deletion/duplication as coverage-supported or not; SV types other than DEL/DUP are ignored. Discard calls lacking coverage support, recording retained/discarded status with support flags separately for each aligner, plus coverage-support status for the 122 truth-set SVs. For repeat runs over the three aligners, reuse cached coverage with --skip-mosdepth, passing the prior per-region coverage files (<P1.label.per-region.bed.gz>) via -cov. Judge success by reproducing the reference results: 83/115/114 discarded calls absent from the gold standard (lra/minimap2/NGMLR) and 55/61 of 122 truth-set SVs coverage-supported.

## Related tools

- **disCoverage** (Runs the coverage-difference statistical test on each DEL/DUP against the NA19240 BAM to flag coverage support and filter calls not present in the truth set) — https://github.com/javiercguard/disCoverage
- **Mosdepth** (Invoked by disCoverage to compute mean per-region coverage for every queried SV interval) — https://github.com/brentp/mosdepth
- **nanopore_pipeline_21** (Pipeline producing the lra/minimap2/NGMLR >50-kb DEL/DUP call sets that are evaluated against the truth set) — https://github.com/javiercguard/nanopore_pipeline_21

## Examples

```
disCoverage -bam NA19240.nanopore.bam -bed lra_calls_50kb.bed -o discoverage_out -i NA19240_lra -t 4
```

## Evaluation signals

- Discarded calls absent from the gold standard reproduce 83/115/114 for the lra/minimap2/NGMLR >50-kb CNV call sets
- 55 of 61 checked (122 total) Chaisson et al. truth-set SVs are flagged coverage-supported
- Specificity increases of ~56%/62% are observed for the minimap2/NGMLR call sets after filtering
- Rerunning with --skip-mosdepth and -cov <P1.label.per-region.bed.gz> yields identical support flags to the full run
- Every DEL/DUP record in the output carries an explicit coverage-support flag; non-DEL/DUP SVs are ignored

## Limitations

- The p-value threshold 0.05 was not confirmed as optimal; further studies are required
- Coverage-based filtering applies only to deletions and duplications, not all SV types
- Coverage is not uniform in a sample, and local peaks and valleys occur without pointing to an SV, limiting the test
- disCoverage is not a variant caller and requires putative SV positions as input

## Evidence
