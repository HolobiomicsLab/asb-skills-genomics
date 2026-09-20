---
name: vcf-callset-precision-recall-computation
description: Use when you have ≥2 caller VCFs and a benchmark truth VCF (e.g. GIAB
  HG002) plus a confidence-region BED, and need per-event-class precision/recall/F1
  to rank SV callers or reproduce a benchmark table.
when_to_use_negative:
- The question is small-variant (SNP/short-indel) accuracy, e.g. the syndip/CHM-eval
  small-variant benchmark — CHM-eval, not svbench, targets that task
- You only have a single call set and no benchmark truth VCF, so precision/recall
  against a reference is undefined
- Comparison must be genotype-concordance or insertion-sequence-accuracy oriented
  (e.g. full-length insertion sequences in PE data, which the paper explicitly leaves
  unresolved)
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: svbench
  role: Computes TP/FP/FN and precision/recall/F1 of query VCFs against a benchmark
    VCF with interval slop matching
  repo: https://github.com/kcleal/svbench
- name: dysgu
  role: Source of query VCFs; its model-probability filter (≥0.5 PacBio, ≥0.35 ONT)
    must be applied before evaluation
  repo: https://github.com/kcleal/dysgu
- name: SVIM
  role: Long-read caller producing a comparable query VCF on the same HG002 PacBio/ONT
    alignments
- name: nanovar
  role: Long-read caller producing a comparable query VCF on the same HG002 PacBio/ONT
    alignments
- name: Sniffles
  role: Long-read caller producing a comparable query VCF on the same HG002 PacBio/ONT
    alignments
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/nar/gkac039
    title: 'Dysgu: efficient structural variant calling using short or long reads'
  source_content_hash: sha256:2af0e8b25068de798715eb3cae4ce5fe32d5cc3a1018fd22797ba4df1b10bbc5
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
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 12
---

# vcf-callset-precision-recall-computation

## Summary

Computes precision, recall and F1 for SV/indel VCF call sets against a benchmark truth set using svbench, the evaluator released with the dysgu paper, so competing callers (dysgu, SVIM, NanoVar, Sniffles, Manta) can be compared on the same GIAB HG002 benchmark restricted to deletion and insertion event classes.

## When to use

Use when you have ≥2 caller VCFs and a benchmark truth VCF (e.g. GIAB HG002) plus a confidence-region BED, and need per-event-class precision/recall/F1 to rank SV callers or reproduce a benchmark table.

## When NOT to use

- The question is small-variant (SNP/short-indel) accuracy, e.g. the syndip/CHM-eval small-variant benchmark — CHM-eval, not svbench, targets that task
- You only have a single call set and no benchmark truth VCF, so precision/recall against a reference is undefined
- Comparison must be genotype-concordance or insertion-sequence-accuracy oriented (e.g. full-length insertion sequences in PE data, which the paper explicitly leaves unresolved)

## Inputs

- Benchmark truth VCF (e.g. GIAB HG002 v0.6 SV benchmark set)
- Query VCFs from each SV caller (dysgu PacBio/ONT, SVIM, NanoVar, Sniffles) filtered with dysgu max model probability ≥0.5 (PacBio) / ≥0.35 (ONT)
- High-confidence regions BED (GIAB Tier 1) supplied via --include

## Outputs

- Per-caller, per-event-class precision, recall and F1 table (lr_sv_comparison_table.csv)
- Optional TP/FP/FN variant lists via --tp/--fp/--fn for manual inspection

## How to apply

Run svbench with the benchmark VCF (e.g. GIAB v0.6 HG002) as REFERENCE_VCF and each caller's VCF as query, using --include with the Tier 1 BED to restrict to high-confidence regions, --pass-only to assess only PASS variants, and the default --slop 250 bp breakpoint tolerance. Call sets must first be filtered comparably — for dysgu, keep calls with maximum model probability ≥0.5 for PacBio and ≥0.35 for ONT — and restrict analysis to deletion and insertion event classes with --min-size-query 30 to match the ≥30 bp indel regime. Use --fp/--fn/--tp to emit per-class false positives, false negatives and true positives, and tabulate F1 = 2PR/(P+R) per caller per read type (PacBio HiFi vs ONT). Rationale: matching the paper's settings (Tier 1 regions, probability filters, deletion/insertion classes) makes the resulting table directly comparable to published Table 5 values such as dysgu F1 0.945 (deletions) and 0.934 (insertions) on HG002 long-read calls.

## Related tools

- **svbench** (Computes TP/FP/FN and precision/recall/F1 of query VCFs against a benchmark VCF with interval slop matching) — https://github.com/kcleal/svbench
- **dysgu** (Source of query VCFs; its model-probability filter (≥0.5 PacBio, ≥0.35 ONT) must be applied before evaluation) — https://github.com/kcleal/dysgu
- **SVIM** (Long-read caller producing a comparable query VCF on the same HG002 PacBio/ONT alignments)
- **nanovar** (Long-read caller producing a comparable query VCF on the same HG002 PacBio/ONT alignments)
- **Sniffles** (Long-read caller producing a comparable query VCF on the same HG002 PacBio/ONT alignments)

## Examples

```
svbench --pass-only --include tier1.bed --slop 250 GIAB_HG002_v0.6.vcf.gz dysgu_pacbio.vcf.gz dysgu_ont.vcf.gz svim.vcf.gz nanovar.vcf.gz sniffles.vcf.gz
```

## Evaluation signals

- F1 values satisfy F1 = 2·precision·recall/(precision+recall) for every caller×event-class row
- Reproduction of paper magnitudes: dysgu highest F1 on HG002 Tier 1 long-read calls, 0.945 for deletions and 0.934 for insertions, with SVIM close behind (0.935/0.928)
- All caller VCFs evaluated against the identical reference VCF, --include BED and --slop (250 bp) so TP/FP/FN are comparable across callers
- dysgu query sets reflect the stated probability filters (max model probability ≥0.5 PacBio, ≥0.35 ONT), otherwise counts are inflated by low-quality calls
- Per-class counts (--tp/--fp/--fn) are integers with TP + FN equal to benchmark event count within the included regions

## Limitations

- Matching depends on the --slop default of 250 bp and --min-size-query of 30; changing either shifts precision/recall and breaks comparability with the paper
- Software versions of dysgu and the other callers are not stated in the paper, and dysgu insertion size accuracy is reported as 'data not shown'
- Duplicate true positives are quantified separately (--no-duplicates/--dtp); the paper notes duplicated true-positive events as an unresolved issue, particularly for insertions in PE data

## Evidence

- [results] filtering calls with a maximum model probability ≥0.5 for PacBio, or ≥0.35 for ONT combinations: "filtering calls with a maximum model probability ≥0.5 for PacBio, or ≥0.35 for ONT combinations"
- [results] On HG002 Tier 1 long-read calls, dysgu achieved the highest F1 score for deletions (0.945) and insertions (0.934): [evidence span withheld — anchor status: not_found]
- [readme] Usage: svbench [OPTIONS] REFERENCE_VCF QUERY_VCFS...: [evidence span withheld — anchor status: external_doc]
- [readme] Add intervals +/- slop around breakpoints: [evidence span withheld — anchor status: external_doc]
- [discussion] Primarily we assessed the HG002 benchmark, analysing in detail high-confidence Tier 1 regions, as well as all genomic regions.: "Primarily we assessed the HG002 benchmark, analysing in detail high-confidence Tier 1 regions, as well as all genomic regions."
