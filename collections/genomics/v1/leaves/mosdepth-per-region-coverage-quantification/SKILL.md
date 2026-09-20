---
name: mosdepth-per-region-coverage-quantification
description: Use when you have a BAM/CRAM of nanopore reads and a set of putative
  SV intervals (4-column BED or VCF of DEL/DUP calls) and need mean per-region depth
  per SV as input to a coverage-support filter.
when_to_use_negative:
- The SV set contains types other than deletions and duplications (e.g., INS, BND,
  INV) — coverage-support analysis can only be applied to DEL/DUP
- You have no putative SV positions yet — disCoverage/Mosdepth is not a variant caller
  and needs regions supplied from variant calling or another technique
- You need per-base or genome-wide depth profiles rather than mean depth over discrete
  SV regions
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: mosdepth
  role: Computes mean per-region depth for each SV interval in the BED (via --by),
    producing the per-region coverage file disCoverage tests
  repo: https://github.com/brentp/mosdepth
- name: disCoverage
  role: Orchestrates Mosdepth on the SV BED/BAM inputs, caches per-region coverage,
    and applies the coverage-difference statistical test
  repo: https://github.com/javiercguard/disCoverage
- name: mamba
  role: Recommended installer to build and create the disCoverage conda environment
    used to run Mosdepth-backed filtering
  repo: https://github.com/mamba-org/mamba
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1016/j.jare.2022.10.012
    title: Improvement of large copy number variant detection by whole genome nanopore
      sequencing
  source_content_hash: sha256:291fc612586222749b6cbbd5dc62094751882436a57c8afe558f33dd9cfa3d23
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/javiercguard/disCoverage
  repo_url: https://github.com/brentp/mosdepth
  tools:
  - disCoverage
  - mamba
  - minimap2
  - mosdepth
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# mosdepth-per-region-coverage-quantification

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Compute mean per-region read depth over candidate structural variant intervals from a nanopore BAM using Mosdepth (invoked by disCoverage), producing the per-region coverage files needed for coverage-support testing of DEL/DUP calls.

## When to use

Use when you have a BAM/CRAM of nanopore reads and a set of putative SV intervals (4-column BED or VCF of DEL/DUP calls) and need mean per-region depth per SV as input to a coverage-support filter.

## When NOT to use

- The SV set contains types other than deletions and duplications (e.g., INS, BND, INV) — coverage-support analysis can only be applied to DEL/DUP
- You have no putative SV positions yet — disCoverage/Mosdepth is not a variant caller and needs regions supplied from variant calling or another technique
- You need per-base or genome-wide depth profiles rather than mean depth over discrete SV regions

## Inputs

- Nanopore whole-genome alignment BAM/CRAM (e.g., NA19240 PromethION reads aligned with lra, minimap2, or NGMLR)
- 4-column BED of SV regions (chr, start, end, SVTYPE), or a .vcf/.vcf.gz of DEL/DUP calls that disCoverage converts
- Output directory and analysis label (-o, -i); thread count (-t, >= 4)
- Optional cached per-region coverage file (<prefix>.per-region.bed.gz) passed via -cov with --skip-mosdepth

## Outputs

- Per-region mean coverage file (<P1.label.per-region.bed.gz>) with one entry per queried SV region
- Cached coverage files reusable across aligner call sets via -cov
- Per-call coverage-support status flags for each DEL/DUP after the disCoverage statistical test

## How to apply

Convert the merged >50-kb DEL/DUP call sets (e.g., lra/minimap2/NGMLR Sniffles output) into 4-column BED format (chr, start, end, SVTYPE); disCoverage can generate these BEDs directly from .vcf or .vcf.gz inputs. Run disCoverage once per aligner's call set passing the nanopore BAM (-bam), the SV BED (-bed), an output directory (-o), an analysis label (-i), and at least 4 threads (-t); disCoverage invokes Mosdepth with the BED as its --by regions file so Mosdepth reports mean per-region coverage for every queried SV. For repeat runs over other aligners' call sets on the same sample, avoid recomputation by passing the prior per-region coverage file (<P1.label.per-region.bed.gz>) via -cov together with --skip-mosdepth. The per-region depth values then feed disCoverage's statistical coverage-difference test (p < 0.05) that flags each deletion/duplication as coverage-supported or not.

## Related tools

- **mosdepth** (Computes mean per-region depth for each SV interval in the BED (via --by), producing the per-region coverage file disCoverage tests) — https://github.com/brentp/mosdepth
- **disCoverage** (Orchestrates Mosdepth on the SV BED/BAM inputs, caches per-region coverage, and applies the coverage-difference statistical test) — https://github.com/javiercguard/disCoverage
- **mamba** (Recommended installer to build and create the disCoverage conda environment used to run Mosdepth-backed filtering) — https://github.com/mamba-org/mamba

## Examples

```
disCoverage -bam NA19240.nanopore.bam -bed lra_cnvs_50kb.bed -o discoverage_out/ -i lra -t 4
```

## Evaluation signals

- A <prefix>.per-region.bed.gz file is produced containing one coverage entry per queried SV region (Mosdepth propagates the 4th-column BED name to its output)
- Re-running with --skip-mosdepth and the cached -cov file yields identical coverage-support flags to a fresh Mosdepth computation
- On the NA19240 benchmark, filtering on the resulting per-region coverage discards 83/115/114 calls absent from the Chaisson et al. truth set for lra/minimap2/NGMLR
- 55/61 of the 122 truth-set SVs are scored as coverage-supported at the p < 0.05 threshold
- Calls flagged as supported show coverage consistent with SVTYPE (reduced depth for DEL, increased for DUP) rather than flat regional coverage

## Limitations

- Coverage is not uniform across a genome; local peaks and valleys occur without pointing to an SV, and shift boundaries may be unclear
- Only deletions and duplications can be assessed; other SV types are ignored by the coverage-support test
- The p < 0.05 threshold has not been confirmed as optimal and requires further study
- Unresolved genome regions limit coverage interpretation

## Evidence

- [other] Run disCoverage once per aligner's call set with the NA19240 nanopore BAM (-bam), the BED of SVs (-bed), an output directory (-o), an analysis label (-i), and at least 4 threads (-t), which invokes Mosdepth: [evidence span withheld — anchor status: not_found]
- [readme] mean per-region given a BED file of regions: [evidence span withheld — anchor status: external_doc]
- [other] reuse cached coverage with --skip-mosdepth: [evidence span withheld — anchor status: not_found]
