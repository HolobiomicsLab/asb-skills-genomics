---
name: ga4gh-vcf-decision-field-aggregation
description: Use when consolidated Truvari bench/refine ga4gh truth and query VCFs
  (e.g., ga4gh_with_refine_truth.vcf.gz, ga4gh_with_refine_query.vcf.gz) are available
  and per-size-stratum caller accuracy (F1) comparisons are needed.
when_to_use_negative:
- The assessment uses only the plain truvari bench command (e.g., the CMRG SV benchmark)
  without the bench/refine consolidation step, so ga4gh decision-annotated VCFs do
  not exist
- The goal is overall (unstratified) caller accuracy rather than size-stratified precision/recall/F1
  comparisons
- Input VCFs are raw caller outputs rather than Truvari-consolidated ga4gh VCFs with
  decision annotations, so TP/FP/FN status is not defined
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3574
tools:
- name: truvari
  role: bench/refine with ga4gh consolidation produces the decision-annotated truth
    and query VCFs whose TP/FP/FN fields are aggregated; version 4.2.2 used with MAFFT
    for refinement
  repo: library://ctsa/truvari/truvari:4.2.2
- name: sawfish
  role: SV caller under evaluation; its ga4gh VCF decision counts are aggregated and
    compared against competitors
  repo: https://github.com/PacificBiosciences/sawfish
- name: pbsv
  role: comparison SV caller whose per-stratum F1 scores are ranked against sawfish
  repo: https://github.com/PacificBiosciences/pbsv
- name: bcftools
  role: used to pre-filter benchmark VCFs to SVTYPE records before assessment
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioinformatics/btaf136
    title: 'Sawfish: improving long-read structural variant discovery and genotyping
      with local haplotype modeling'
  source_content_hash: sha256:4d277422f47c6eaeab7c390542c67de7aa6b0d49f44848956998229f4466d40b
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - MAFFT
  - bcftools
  - pbsv
  - sawfish
  - truvari
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3574
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# ga4gh-vcf-decision-field-aggregation

## Summary

Aggregate Truvari decision annotations (TP/FP/FN) from consolidated ga4gh VCF outputs of SV callers, stratify benchmark-qualified variants by SV size, and compute per-stratum precision, recall, and F1 to rank callers (e.g., sawfish vs. pbsv vs. Sniffles2) as in the sawfish GIAB T2T-HG002 benchmark.

## When to use

Use when consolidated Truvari bench/refine ga4gh truth and query VCFs (e.g., ga4gh_with_refine_truth.vcf.gz, ga4gh_with_refine_query.vcf.gz) are available and per-size-stratum caller accuracy (F1) comparisons are needed.

## When NOT to use

- The assessment uses only the plain truvari bench command (e.g., the CMRG SV benchmark) without the bench/refine consolidation step, so ga4gh decision-annotated VCFs do not exist
- The goal is overall (unstratified) caller accuracy rather than size-stratified precision/recall/F1 comparisons
- Input VCFs are raw caller outputs rather than Truvari-consolidated ga4gh VCFs with decision annotations, so TP/FP/FN status is not defined

## Inputs

- Consolidated Truvari ga4gh truth VCFs (e.g., ga4gh_with_refine_truth.vcf.gz) for sawfish, pbsv, and Sniffles2 from the 32x HG002 assessment
- Consolidated Truvari ga4gh query VCFs (e.g., ga4gh_with_refine_query.vcf.gz) for each caller
- Truvari decision annotations (TP/FP/FN) encoded in the ga4gh VCF records

## Outputs

- Per-stratum caller-by-metric F1 table (size_stratified_f1_rankings.csv), mirroring Supplementary Table S2
- Per-stratum TP, FP, FN counts and derived precision/recall/F1 values for each caller

## How to apply

Load the consolidated Truvari ga4gh truth and query VCFs for each caller (sawfish, pbsv, Sniffles2) from the 32x HG002 assessment results in the Zenodo deposit (doi:10.5281/zenodo.14898462). Compute each benchmark-qualified variant's length as END - POS for non-BND records, applying the --dup-to-ins convention so duplications are sized as insertions, and assign each TP, FP, and FN call to one of three strata: 50-499 bp, 500-4999 bp, and >=5000 bp. Aggregate per-stratum TP/FP/FN counts from the Truvari decision annotations in the ga4gh VCFs, then compute per-stratum precision, recall, and F1 (harmonic mean of precision and recall) for each caller and rank callers within each bin. Write the caller-by-metric F1 table to size_stratified_f1_rankings.csv, mirroring the layout of Supplementary Table S2.

## Related tools

- **truvari** (bench/refine with ga4gh consolidation produces the decision-annotated truth and query VCFs whose TP/FP/FN fields are aggregated; version 4.2.2 used with MAFFT for refinement) — library://ctsa/truvari/truvari:4.2.2
- **sawfish** (SV caller under evaluation; its ga4gh VCF decision counts are aggregated and compared against competitors) — https://github.com/PacificBiosciences/sawfish
- **pbsv** (comparison SV caller whose per-stratum F1 scores are ranked against sawfish) — https://github.com/PacificBiosciences/pbsv
- **bcftools** (used to pre-filter benchmark VCFs to SVTYPE records before assessment)

## Examples

```
truvari bench -b truth.vcf.gz -c query.vcf.gz -f human_GRCh38_no_alt_analysis_set.fasta --dup-to-ins -o bench_out && truvari ga4gh bench_out -o ga4gh_with_refine
```

## Evaluation signals

- Computed F1 ranking places sawfish first in all three size strata (50-499 bp, 500-4999 bp, >=5000 bp)
- Sawfish's 50-499 bp F1 lead over pbsv reproduces the reported 0.5 percentage points within rounding tolerance
- Sawfish's >=5000 bp F1 leads reproduce the reported 2.6 points over Sniffles2 and 7.0 points over pbsv within rounding tolerance
- Per-stratum TP+FN counts are consistent between truth and query ga4gh VCFs for each caller (matching true variant totals)
- Output size_stratified_f1_rankings.csv layout matches the caller-by-metric structure of Supplementary Table S2

## Limitations

- Requires the bench/refine ga4gh consolidation; the CMRG assessment uses truvari bench alone and does not yield these decision-annotated ga4gh VCFs
- Sizing uses END - POS for non-BND records with the --dup-to-ins convention, so duplications are sized as insertions and BND records are excluded from size strata
- VACmap is still unsupported in the Truvari workflow, limiting some alignment-based comparisons

## Evidence

- [methods] All assessment below uses truvari 4.2.2 with MAFFT for refinement.: "All assessment below uses truvari 4.2.2 with MAFFT for refinement."
- [methods] bench/refine results consolidated as ga4gh VCFs: "it is important to use the bench and refine command, with the bench/refine results consolidated as described further below."
- [abstract] sawfish highest accuracy across all SV size groups: "sawfish has the highest accuracy among state-of-the-art long-read SV callers across every tested SV size group."
- [methods] bcftools SVTYPE filtering of benchmark VCF: "bcftools view -i 'INFO/SVTYPE!="."' -Oz -o GRCh38_HG2-T2TQ100-V1.1_stvar.svtype.vcf.gz"
