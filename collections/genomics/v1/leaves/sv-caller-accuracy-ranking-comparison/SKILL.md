---
name: sv-caller-accuracy-ranking-comparison
description: Use when comparing accuracy of long-read SV callers against a GIAB truth
  set and you need F1 rankings stratified by SV length (50-499 bp, 500-4999 bp, >=5000
  bp).
when_to_use_negative:
- Evaluating the GIAB CMRG SV benchmark (HG002_GRCh38_CMRG_SV_v1.00), where truvari
  bench alone is used without the bench/refine/ga4gh consolidation
- Depth-stratified or joint-genotyping pedigree-concordance comparisons rather than
  SV size-stratified accuracy
- Calls outside benchmark-qualified confident regions or BND/translocation records,
  which have no END-POS length for stratification
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3577
tools:
- name: sawfish
  role: SV/CNV caller under benchmark; expected to rank first by F1 in every size
    stratum
  repo: https://github.com/PacificBiosciences/sawfish
- name: truvari 4.2.2
  role: bench/refine/ga4gh comparison engine producing TP/FP/FN decision annotations,
    with MAFFT for refinement
  repo: library://ctsa/truvari/truvari:4.2.2
- name: MAFFT
  role: Alignment refinement engine used by truvari refine in the T2T assessment
- name: pbsv
  role: Comparison caller; next-best in the 50-499 bp stratum
  repo: https://github.com/PacificBiosciences/pbsv
- name: bcftools
  role: Pre-filters the benchmark VCF to structural variant records (INFO/SVTYPE !=
    '.')
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioinformatics/btaf136
    title: 'Sawfish: improving long-read structural variant discovery and genotyping
      with local haplotype modeling'
  source_content_hash: sha256:4b05df52516ce8e0967e5a5ee4e13e3a405789e85e511a818b96ce7a33428f42
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/PacificBiosciences/sawfish
  tools:
  - MAFFT
  - bcftools
  - pbsv
  - sawfish
  - truvari 4.2.2
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3577
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# sv-caller-accuracy-ranking-comparison

## Summary

Benchmark SV callers (sawfish, Sniffles2, pbsv) on the GIAB T2T HG002 benchmark by computing per-size-stratum precision, recall, and F1 from Truvari ga4gh consolidation outputs, showing sawfish leads in every SV size group.

## When to use

Use when comparing accuracy of long-read SV callers against a GIAB truth set and you need F1 rankings stratified by SV length (50-499 bp, 500-4999 bp, >=5000 bp).

## When NOT to use

- Evaluating the GIAB CMRG SV benchmark (HG002_GRCh38_CMRG_SV_v1.00), where truvari bench alone is used without the bench/refine/ga4gh consolidation
- Depth-stratified or joint-genotyping pedigree-concordance comparisons rather than SV size-stratified accuracy
- Calls outside benchmark-qualified confident regions or BND/translocation records, which have no END-POS length for stratification

## Inputs

- Consolidated Truvari ga4gh truth VCFs (ga4gh_with_refine_truth.vcf.gz) for sawfish, pbsv, and Sniffles2 from 32x HG002 assessment results
- Consolidated Truvari ga4gh query VCFs (ga4gh_with_refine_query.vcf.gz) for each caller
- GIAB draft T2T SV benchmark V0.019-20241113 (GRCh38_HG2-T2TQ100-V1.1_stvar), SVTYPE-filtered and restricted to autosomes

## Outputs

- Per-stratum caller-by-metric F1 table (size_stratified_f1_rankings.csv), mirroring Supplementary Table S2 layout
- Per-stratum TP/FP/FN counts, precision, and recall per caller

## How to apply

Load the consolidated Truvari ga4gh truth and query VCFs (ga4gh_with_refine_truth.vcf.gz and ga4gh_with_refine_query.vcf.gz) for each caller from the 32x assessment results in the Zenodo deposit (doi:10.5281/zenodo.14898462). Compute each benchmark-qualified variant's length as END - POS for non-BND records using the --dup-to-ins convention so duplications are sized as insertions, and assign TP, FP, and FN calls to 50-499 bp, 500-4999 bp, or >=5000 bp strata. Aggregate per-stratum TP/FP/FN counts from the Truvari decision annotations, compute precision, recall, and F1 (harmonic mean) per stratum, and rank callers within each bin. The T2T assessment must use truvari 4.2.2 with MAFFT refinement via the bench/refine workflow with ga4gh consolidation, not truvari bench alone (which is reserved for the CMRG assessment).

## Related tools

- **sawfish** (SV/CNV caller under benchmark; expected to rank first by F1 in every size stratum) — https://github.com/PacificBiosciences/sawfish
- **truvari 4.2.2** (bench/refine/ga4gh comparison engine producing TP/FP/FN decision annotations, with MAFFT for refinement) — library://ctsa/truvari/truvari:4.2.2
- **MAFFT** (Alignment refinement engine used by truvari refine in the T2T assessment)
- **pbsv** (Comparison caller; next-best in the 50-499 bp stratum) — https://github.com/PacificBiosciences/pbsv
- **bcftools** (Pre-filters the benchmark VCF to structural variant records (INFO/SVTYPE != '.'))

## Examples

```
bcftools view -i 'INFO/SVTYPE!="."' -Oz -o GRCh38_HG2-T2TQ100-V1.1_stvar.svtype.vcf.gz GRCh38_HG2-T2TQ100-V1.1_stvar.vcf.gz
```

## Evaluation signals

- Computed F1 ranking places sawfish first in all three size strata (50-499 bp, 500-4999 bp, >=5000 bp)
- Sawfish's 50-499 bp lead over pbsv reproduces 0.5 percentage points within rounding tolerance
- Sawfish's >=5000 bp leads reproduce 2.6 points over Sniffles2 and 7.0 points over pbsv within rounding tolerance
- Sizing uses END - POS on non-BND records with the --dup-to-ins convention, so duplications are counted in insertion size bins
- Stratum assignment partitions all TP, FP, and FN calls exhaustively across the three size bins

## Limitations

- Findings apply to the GIAB draft T2T SV benchmark on HiFi data at 32x; they do not automatically transfer to other callers, chemistries, or truth sets
- BND records lack an END-POS length and are not assignable to the size strata
- Requires the bench/refine workflow with MAFFT refinement and ga4gh consolidation; plain truvari bench output is insufficient for the T2T benchmark comparison

## Evidence

- [abstract] Assessment against the draft Genome in a Bottle (GIAB) SV benchmark from the T2T-HG002-Q100 diploid assembly shows that sawfish has the highest accuracy: "Assessment against the draft Genome in a Bottle (GIAB) SV benchmark from the T2T-HG002-Q100 diploid assembly shows that sawfish has the highest accuracy"
- [methods] All assessment below uses truvari 4.2.2 with MAFFT for refinement.: "All assessment below uses truvari 4.2.2 with MAFFT for refinement."
- [methods] it is important to use the bench and refine command, with the bench/refine results consolidated as described further below.: "it is important to use the bench and refine command, with the bench/refine results consolidated as described further below."
- [methods] more information on assessment script details from the App Note are also available on Zenodo: "more information on assessment script details from the App Note are also [available on Zenodo](https://doi.org/10.5281/zenodo.14898462)."
- [methods] The benchmark files are modified to select for structural variants only as follows: bcftools view -i 'INFO/SVTYPE!="."': "The benchmark files are modified to select for structural variants only as follows: bcftools view -i 'INFO/SVTYPE!="."'"
