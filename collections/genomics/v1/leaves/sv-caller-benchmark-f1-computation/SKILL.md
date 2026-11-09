---
name: sv-caller-benchmark-f1-computation
description: Use when comparing SV caller accuracy from Truvari GA4GH VCF outputs
  and you need per-size-stratum (50-499 bp, 500-4999 bp, >=5000 bp) precision/recall/F1
  rankings across callers.
when_to_use_negative:
- The comparison VCFs are raw Truvari bench outputs that have not been consolidated
  with the refine step — the article specifies using bench AND refine results consolidated
  into the ga4gh VCFs, so unrefined outputs will not match reported F1 values
- The input records are BND/translocation calls, which lack END-based lengths and
  are excluded from the size-stratified computation (only non-BND records are sized)
- The task is depth-level (10- to 32-fold) accuracy comparison rather than SV size-stratified
  comparison, which requires a different aggregation dimension
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3574
tools:
- name: truvari
  role: Performs the bench/refive comparison producing the GA4GH truth/query VCFs
    whose TP/FP/FN decision annotations are aggregated per size stratum (version 4.2.2
    used with MAFFT refinement)
  repo: library://ctsa/truvari/truvari:4.2.2
- name: sawfish
  role: SV caller under evaluation; its per-stratum F1 is compared against Sniffles2
    and pbsv
  repo: https://github.com/PacificBiosciences/sawfish
- name: pbsv
  role: Comparison SV caller benchmarked in the size-stratified F1 analysis
  repo: https://github.com/PacificBiosciences/pbsv
- name: bcftools
  role: Used to filter benchmark VCFs to structural variant records only prior to
    assessment (bcftools view -i 'INFO/SVTYPE!="."')
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioinformatics/btaf136
    title: 'Sawfish: improving long-read structural variant discovery and genotyping
      with local haplotype modeling'
  source_content_hash: sha256:41f990eff81221a0bff51fc0f01cfd1b3ab4e022ec4c1bf9b1615e2788f4ba89
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
yes_rate: 0.7692307692307693
n_claims: 13
---

# sv-caller-benchmark-f1-computation

## Summary

Computes size-stratified precision, recall, and F1 scores for long-read SV callers (sawfish, Sniffles2, pbsv) by aggregating Truvari bench/refine TP, FP, and FN annotations from consolidated GA4GH VCFs on the GIAB T2T-HG002-Q100 benchmark, reproducing the article's finding that sawfish ranks first in every SV size stratum.

## When to use

Use when comparing SV caller accuracy from Truvari GA4GH VCF outputs and you need per-size-stratum (50-499 bp, 500-4999 bp, >=5000 bp) precision/recall/F1 rankings across callers.

## When NOT to use

- The comparison VCFs are raw Truvari bench outputs that have not been consolidated with the refine step — the article specifies using bench AND refine results consolidated into the ga4gh VCFs, so unrefined outputs will not match reported F1 values
- The input records are BND/translocation calls, which lack END-based lengths and are excluded from the size-stratified computation (only non-BND records are sized)
- The task is depth-level (10- to 32-fold) accuracy comparison rather than SV size-stratified comparison, which requires a different aggregation dimension

## Inputs

- Truvari 4.2.2 bench/refine consolidated GA4GH truth and query VCFs (ga4gh_with_refine_truth.vcf.gz, ga4gh_with_refine_query.vcf.gz) for sawfish, pbsv, and Sniffles2 from the 32x HG002 assessment (Zenodo doi:10.5281/zenodo.14898462)
- Truvari per-variant decision annotations (TP/FP/FN) carried in the GA4GH VCFs
- Variant coordinates (POS/END) and SVTYPE annotations for non-BND records

## Outputs

- size_stratified_f1_rankings.csv: per-stratum caller-by-metric F1 table mirroring the layout of Supplementary Table S2
- Per-stratum precision, recall, and F1 values with caller rankings for the 50-499 bp, 500-4999 bp, and >=5000 bp bins

## How to apply

Load the consolidated Truvari ga4gh truth and query VCFs (e.g. ga4gh_with_refine_truth.vcf.gz and ga4gh_with_refine_query.vcf.gz) for each caller from the 32x HG002 assessment results in the Zenodo deposit (doi:10.5281/zenodo.14898462). Compute each benchmark-qualified variant's length as END - POS for non-BND records, applying the --dup-to-ins convention so duplications are sized as insertions, and assign each TP, FP, and FN call (per Truvari decision annotations) to one of three size strata: 50-499 bp, 500-4999 bp, and >=5000 bp. Aggregate per-stratum counts, then compute precision, recall, and their harmonic mean (F1) for each caller and rank callers within each bin. Validation: confirm sawfish ranks first in all three strata, reproducing the reported leads (0.5 points over pbsv at 50-499 bp; 2.6 points over Sniffles2 and 7.0 points over pbsv at >=5000 bp) within rounding tolerance.

## Related tools

- **truvari** (Performs the bench/refive comparison producing the GA4GH truth/query VCFs whose TP/FP/FN decision annotations are aggregated per size stratum (version 4.2.2 used with MAFFT refinement)) — library://ctsa/truvari/truvari:4.2.2
- **sawfish** (SV caller under evaluation; its per-stratum F1 is compared against Sniffles2 and pbsv) — https://github.com/PacificBiosciences/sawfish
- **pbsv** (Comparison SV caller benchmarked in the size-stratified F1 analysis) — https://github.com/PacificBiosciences/pbsv
- **bcftools** (Used to filter benchmark VCFs to structural variant records only prior to assessment (bcftools view -i 'INFO/SVTYPE!="."'))

## Examples

```
truvari bench -b truth.vcf.gz -c query.vcf.gz -f human_GRCh38_no_alt_analysis_set.fasta --dup-to-ins -o bench_out && truvari refine bench_out && truvari ga4gh bench_out/ -o ga4gh_with_refine
```

## Evaluation signals

- Computed F1 ranking places sawfish first in all three size strata (50-499 bp, 500-4999 bp, >=5000 bp), consistent with the article's finding that sawfish has the highest accuracy across every tested SV size group
- The sawfish F1 lead in the 50-499 bp bin is approximately 0.5 percentage points over pbsv, within rounding tolerance
- The sawfish F1 lead in the >=5000 bp bin is approximately 2.6 points over Sniffles2 and 7.0 points over pbsv, within rounding tolerance
- Per-stratum TP, FP, and FN counts sum consistently with the Truvari decision annotations in the ga4gh VCFs for each caller
- Output table size_stratified_f1_rankings.csv matches the caller-by-metric layout of Supplementary Table S2

## Limitations

- SV sizing requires the --dup-to-ins convention (duplications treated as insertions) and excludes BND records, so size strata only cover non-BND variants sized as END - POS
- Results are specific to the GIAB draft T2T SV benchmark (V0.019-20241113, GRCh38_HG2-T2TQ100-V1.1_stvar) restricted to autosomes with Truvari 4.2.2 and MAFFT refinement; other benchmarks (e.g. the CMRG GRCh38 assessment) use truvari bench alone and will not reproduce these values
- Reported leads are subject to rounding tolerance when reproduced

## Evidence

- [abstract] Assessment against the draft Genome in a Bottle (GIAB) SV benchmark from the T2T-HG002-Q100 diploid assembly shows that sawfish has the highest accuracy: "Assessment against the draft Genome in a Bottle (GIAB) SV benchmark from the T2T-HG002-Q100 diploid assembly shows that sawfish has the highest accuracy"
- [abstract] sawfish has the highest accuracy among state-of-the-art long-read SV callers across every tested SV size group.: "sawfish has the highest accuracy among state-of-the-art long-read SV callers across every tested SV size group."
- [methods] All assessment below uses truvari 4.2.2 with MAFFT for refinement.: "All assessment below uses truvari 4.2.2 with MAFFT for refinement."
- [methods] it is important to use the bench and refine command, with the bench/refine results consolidated as described further below.: "it is important to use the bench and refine command, with the bench/refine results consolidated as described further below."
- [methods] more information on assessment script details from the App Note are also [available on Zenodo](https://doi.org/10.5281/zenodo.14898462).: "more information on assessment script details from the App Note are also [available on Zenodo](https://doi.org/10.5281/zenodo.14898462)."
