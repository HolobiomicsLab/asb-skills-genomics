---
name: giab-sv-truthset-comparison
description: Use when one or more SV caller VCFs need precision/recall/F1 assessment
  against a GIAB truth VCF (e.g. HG002), especially for deletion/insertion calls in
  Tier 1 high-confidence regions.
when_to_use_negative:
- The input is raw alignment data (BAM/CRAM) with no VCF call set — SV calling (e.g.
  dysgu run/call) must happen first
- Evaluating small indels below the truth set's size scope (the paper compares indel
  calling with strelka/gatk down to 30 bp using other benchmarks such as syndip/CHM1-CHM13),
  since GIAB SV truth sets target SVs/indels >=50 bp
- Somatic or tumour call sets where a GIAB germline truth set is not an appropriate
  reference
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3175
tools:
- name: svbench
  role: Distance-based benchmarking tool comparing caller query VCFs against the GIAB
    truth VCF to compute precision, recall and F1
  repo: https://github.com/kcleal/svbench
- name: dysgu
  role: SV/indel caller producing the query VCFs under test; supports --mode presets
    for PacBio and ONT and a filter command with model-probability thresholds
  repo: https://github.com/kcleal/dysgu
- name: SVIM
  role: Long-read SV caller producing a comparable query VCF for the benchmark
- name: nanovar
  role: Long-read SV caller producing a comparable query VCF for the benchmark
- name: Sniffles
  role: Long-read SV caller producing a comparable query VCF for the benchmark
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/nar/gkac039
    title: 'Dysgu: efficient structural variant calling using short or long reads'
  source_content_hash: sha256:c83681a0e262cb9969a8b5f824a0d44997c8f173341b797551ca81cfb35821d0
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/kcleal/svbench
  tools:
  - SVIM
  - Sniffles
  - dysgu
  - nanovar
  - svbench
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3175
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# giab-sv-truthset-comparison

## Summary

Benchmark structural variant caller VCFs (e.g. dysgu, SVIM, NanoVar, Sniffles) against the GIAB HG002 Ashkenazi son truth set using svbench, restricting to high-confidence Tier 1 regions and event classes such as deletions and insertions to compute precision, recall and F1.

## When to use

Use when one or more SV caller VCFs need precision/recall/F1 assessment against a GIAB truth VCF (e.g. HG002), especially for deletion/insertion calls in Tier 1 high-confidence regions.

## When NOT to use

- The input is raw alignment data (BAM/CRAM) with no VCF call set — SV calling (e.g. dysgu run/call) must happen first
- Evaluating small indels below the truth set's size scope (the paper compares indel calling with strelka/gatk down to 30 bp using other benchmarks such as syndip/CHM1-CHM13), since GIAB SV truth sets target SVs/indels >=50 bp
- Somatic or tumour call sets where a GIAB germline truth set is not an appropriate reference

## Inputs

- GIAB HG002 SV benchmark truth VCF (e.g. v0.6 Tier 1 high-confidence regions)
- Caller query VCFs (dysgu PacBio, dysgu ONT, SVIM, NanoVar, Sniffles output)
- Tier 1 / include-regions BED file
- PacBio HiFi and ONT alignment BAM/CRAM files (e.g. SRA SRR10188368, SRR10188369) used to generate the query calls

## Outputs

- Per-caller, per-event-class precision, recall and F1 scores
- TP/FP/FN (and duplicate TP) counts per caller, optionally as TP/FP/FN variant lists via --tp/--fp/--fn flags
- Comparison table (e.g. lr_sv_comparison_table.csv) reproducing the paper's Table 5

## How to apply

Obtain the caller VCFs to be evaluated and the GIAB HG002 benchmark truth set (v0.6 for long-read work; GIAB v1.1 is also used in the CMRG/GIAB benchmark repo). Run svbench with the truth VCF as the reference and each caller VCF as a query, e.g. `svbench --include tier1.bed --pass-only reference.vcf query.vcf`, which matches SVs within a default +/-250 bp breakpoint slop and supports minimum size thresholds (`--min-size-ref`, `--min-size-query`, default query min 30 bp). Restrict the comparison to Tier 1 high-confidence regions via the include BED and, per the paper's long-read workflow, restrict event classes to deletions and insertions before tabulating F1, precision and recall per caller per read type. For PacBio-derived dysgu calls apply the paper's quality filter (maximum model probability >=0.5; >=0.35 for ONT combinations) before evaluation. Write the resulting per-caller metrics to a comparison table (e.g. lr_sv_comparison_table.csv) and sanity-check TP/FP/FN counts against the caller's raw call counts.

## Related tools

- **svbench** (Distance-based benchmarking tool comparing caller query VCFs against the GIAB truth VCF to compute precision, recall and F1) — https://github.com/kcleal/svbench
- **dysgu** (SV/indel caller producing the query VCFs under test; supports --mode presets for PacBio and ONT and a filter command with model-probability thresholds) — https://github.com/kcleal/dysgu
- **SVIM** (Long-read SV caller producing a comparable query VCF for the benchmark)
- **nanovar** (Long-read SV caller producing a comparable query VCF for the benchmark)
- **Sniffles** (Long-read SV caller producing a comparable query VCF for the benchmark)

## Examples

```
svbench --include tier1.bed --pass-only HG002_v0.6_trio.vcf dysgu_pacbio.vcf sniffles.vcf svim.vcf nanovar.vcf
```

## Evaluation signals

- svbench reports matching TP/FP/FN counts whose TP + FN should equal the number of benchmark truth variants inside the include regions; TPs should not exceed caller VCF record counts
- F1, precision and recall values should fall in [0, 1] and reproduce paper-scale results, e.g. dysgu F1 of 0.945 (deletions) and 0.934 (insertions) on HG002 Tier 1 long-read calls
- Metrics are restricted to deletion and insertion event classes; duplications/inversions appearing in the output indicate event-class filtering was not applied
- Matching is governed by svbench's +/-250 bp breakpoint slop by default; changing --slop should visibly change TP counts, confirming the include-region and slop settings took effect
- Comparison table rows should exist for every caller x read-type combination (dysgu PacBio, dysgu ONT, SVIM, NanoVar, Sniffles) with no missing cells

## Limitations

- No software versions are stated for any compared tool in the article, complicating exact reproduction
- svbench matching is slop-based (default 250 bp) and duplicate TPs can inflate sensitivity unless --no-duplicates is used
- Benchmark scope is limited to the truth set's covered regions and event classes; truth sets such as GIAB cover high-confidence Tier 1 regions only, so genome-wide performance cannot be directly assessed
- Manually/curated truth sets carry observer bias and may exclude hard-to-call variant classes (e.g. novel sequence insertions are better assessed with assembly-based benchmarks)

## Evidence

- [results] filtering calls with a maximum model probability ≥0.5 for PacBio, or ≥0.35 for ONT combinations (Table 7): "filtering calls with a maximum model probability ≥0.5 for PacBio, or ≥0.35 for ONT combinations (Table 7)"
- [discussion] Primarily we assessed the HG002 benchmark, analysing in detail high-confidence Tier 1 regions: "Primarily we assessed the HG002 benchmark, analysing in detail high-confidence Tier 1 regions, as well as all genomic regions."
- [supplementary] Svbench is also released under the MIT license and can be found at https://github.com/kcleal/svbench.: "Svbench is also released under the MIT license and can be found at https://github.com/kcleal/svbench."
- [readme] Usage: svbench [OPTIONS] REFERENCE_VCF QUERY_VCFS...: [evidence span withheld — anchor status: external_doc]
- [readme] --slop INTEGER            Add intervals +/- slop around breakpoints [default: 250]: "--slop INTEGER            Add intervals +/- slop around breakpoints
                                [default: 250]"
