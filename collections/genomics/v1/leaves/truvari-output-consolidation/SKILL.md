---
name: truvari-output-consolidation
description: Use when comparing SV caller accuracy against the GIAB T2T-HG002-Q100
  truth set and you need per-call Truvari decision labels from consolidated ga4gh
  VCFs, especially for size-stratified F1 rankings.
when_to_use_negative:
- Assessing the GIAB CMRG SV benchmark (HG002_GRCh38_CMRG_SV_v1.00), which uses the
  truvari bench command alone without bench/refine consolidation
- Comparing accuracy by sequencing depth or pedigree concordance rather than SV size
  strata, which requires different aggregation of the Truvari outputs
- Benchmarking with hap-eval or a different comparison engine whose output VCFs do
  not carry Truvari decision annotations
edam_operation: http://edamontology.org/operation_2928
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3175
tools:
- name: truvari
  role: bench/refine benchmarking of caller VCFs against the GIAB T2T SV truth set
    with MAFFT refinement; consolidation of outputs into ga4gh VCFs with decision
    annotations
  repo: https://cloud.sylabs.io/library/ctsa/truvari/truvari
- name: MAFFT
  role: sequence refinement used within truvari refine for the T2T SV assessment
- name: bcftools
  role: pre-filtering of the benchmark VCF to SVTYPE-defined structural variant records
    before consolidation
- name: sawfish
  role: query SV caller being benchmarked (v0.12.9); its consolidated Truvari results
    supply the per-stratum F1 values being validated
  repo: https://github.com/PacificBiosciences/sawfish
- name: pbsv
  role: comparison caller whose consolidated Truvari results provide the F1 baseline
    in the 50-499 bp and >=5000 bp strata
  repo: https://github.com/PacificBiosciences/pbsv
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioinformatics/btaf136
    title: 'Sawfish: improving long-read structural variant discovery and genotyping
      with local haplotype modeling'
  source_content_hash: sha256:7f322ade3f84ab1ff416e2c8d8d94a5f737434a5edf14aace89e86941bd5ae9d
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - MAFFT
  - bcftools
  - pbsv
  - sawfish
  - truvari
  edam_operation: http://edamontology.org/operation_2928
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3175
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# truvari-output-consolidation

## Summary

Consolidate Truvari bench/refine benchmarking outputs into GA4GH intermediate VCFs (e.g. ga4gh_with_refine_truth.vcf.gz and ga4gh_with_refine_query.vcf.gz) carrying per-call decision annotations (TP/FP/FN), then stratify SVs by size (50-499 bp, 500-4999 bp, >=5000 bp) to compute caller precision, recall, and F1 for sawfish, pbsv, and Sniffles2 on the 32x HG002 T2T benchmark.

## When to use

Use when comparing SV caller accuracy against the GIAB T2T-HG002-Q100 truth set and you need per-call Truvari decision labels from consolidated ga4gh VCFs, especially for size-stratified F1 rankings.

## When NOT to use

- Assessing the GIAB CMRG SV benchmark (HG002_GRCh38_CMRG_SV_v1.00), which uses the truvari bench command alone without bench/refine consolidation
- Comparing accuracy by sequencing depth or pedigree concordance rather than SV size strata, which requires different aggregation of the Truvari outputs
- Benchmarking with hap-eval or a different comparison engine whose output VCFs do not carry Truvari decision annotations

## Inputs

- Truvari 4.2.2 bench and refine output directories for sawfish, pbsv, and Sniffles2 on the 32x HG002 GIAB draft T2T SV benchmark (GRCh38_HG2-T2TQ100-V1.1_stvar)
- Consolidated ga4gh truth and query VCFs (ga4gh_with_refine_truth.vcf.gz, ga4gh_with_refine_query.vcf.gz) with Truvari decision annotations (TP/FP/FN)
- Autosome-restricted confident regions BED derived by excluding chr[XY]
- MAFFT-aligned refinement results from truvari refine

## Outputs

- size_stratified_f1_rankings.csv: per-stratum caller-by-metric F1 table mirroring Supplementary Table S2
- Per-stratum TP/FP/FN counts and precision, recall, and F1 for each caller in the 50-499 bp, 500-4999 bp, and >=5000 bp bins

## How to apply

Run the 32x HG002 assessment as in the App Note: map HiFi reads with pbmm2 1.13.1, call SVs with sawfish 0.12.9, pbsv, and Sniffles2, then assess all callers with truvari 4.2.2 using the bench and refine commands with MAFFT refinement, consolidating the results into ga4gh_with_refine_truth.vcf.gz and ga4gh_with_refine_query.vcf.gz per caller. Before benchmarking, restrict the truth set to structural variants with 'bcftools view -i 'INFO/SVTYPE!="."'' and restrict the confident-region BED to autosomes with 'grep -v chr[XY]'. Compute each benchmark-qualified variant's length as END - POS for non-BND records using the --dup-to-ins convention (duplications sized as insertions), and bin TP, FP, and FN calls into 50-499 bp, 500-4999 bp, and >=5000 bp strata. Aggregate per-stratum counts from the Truvari decision annotations, compute per-stratum precision, recall, and F1 (harmonic mean), and write the caller-by-metric table (e.g. size_stratified_f1_rankings.csv, mirroring Supplementary Table S2). Validate by checking that sawfish ranks first in all three strata, reproducing reported leads of 0.5 percentage points over pbsv at 50-499 bp and 2.6 and 7.0 points over Sniffles2 and pbsv, respectively, at >=5000 bp within rounding tolerance.

## Related tools

- **truvari** (bench/refine benchmarking of caller VCFs against the GIAB T2T SV truth set with MAFFT refinement; consolidation of outputs into ga4gh VCFs with decision annotations) — https://cloud.sylabs.io/library/ctsa/truvari/truvari
- **MAFFT** (sequence refinement used within truvari refine for the T2T SV assessment)
- **bcftools** (pre-filtering of the benchmark VCF to SVTYPE-defined structural variant records before consolidation)
- **sawfish** (query SV caller being benchmarked (v0.12.9); its consolidated Truvari results supply the per-stratum F1 values being validated) — https://github.com/PacificBiosciences/sawfish
- **pbsv** (comparison caller whose consolidated Truvari results provide the F1 baseline in the 50-499 bp and >=5000 bp strata) — https://github.com/PacificBiosciences/pbsv

## Examples

```
truvari bench -b GRCh38_HG2-T2TQ100-V1.1_stvar.svtype.vcf.gz -c caller.vcf.gz -f human_GRCh38_no_alt_analysis_set.fasta --include-bed confident_regions.autosomes.bed -o bench_out && truvari refine -m -o refine_out bench_out && python scripts/size_stratified_f1.py --ga4gh-truth refine_out/ga4gh_with_refine_truth.vcf.gz --ga4gh-query refine_out/ga4gh_with_refine_query.vcf.gz --dup-to-ins -o size_stratified_f1_rankings.csv
```

## Evaluation signals

- Computed F1 ranking places sawfish first in all three size strata (50-499 bp, 500-4999 bp, >=5000 bp)
- Computed sawfish lead over pbsv in the 50-499 bp stratum matches the reported 0.5 percentage points within rounding tolerance
- Computed sawfish leads in the >=5000 bp stratum match the reported 2.6 points over Sniffles2 and 7.0 points over pbsv within rounding tolerance
- Per-call TP/FP/FN counts in each stratum sum, per caller, to the overall Truvari bench/refine call totals for that caller
- Variant lengths are computed as END - POS for non-BND records with the --dup-to-ins convention applied consistently across all three callers

## Limitations

- Sizing must exclude BND records and treat duplications as insertions (--dup-to-ins convention); inconsistent sizing conventions will break stratum assignment
- The T2T assessment requires bench plus refine with MAFFT — plain truvari bench alone is only appropriate for the CMRG benchmark, not this workflow
- The confident regions must be restricted to autosomes; including chr[XY] would change the TP/FP/FN counts relative to the published benchmark
- Assessment scripts and data live in the Zenodo deposit (doi:10.5281/zenodo.14898462); exact consolidation behavior depends on that versioned deposit

## Evidence

- [methods] All assessment below uses truvari 4.2.2 with MAFFT for refinement.: "All assessment below uses truvari 4.2.2 with MAFFT for refinement."
- [methods] bench and refine consolidation: "it is important to use the bench and refine command, with the bench/refine results consolidated as described further below."
- [methods] SVTYPE filtering: "The benchmark files are modified to select for structural variants only as follows: bcftools view -i 'INFO/SVTYPE!="."'"
- [methods] autosome restriction: "The confident regions are restricted to autosomes as follows: grep -v chr[XY]"
- [abstract] highest accuracy across size strata: "sawfish has the highest accuracy among state-of-the-art long-read SV callers across every tested SV size group."
- [methods] assessment scripts on Zenodo: "more information on assessment script details from the App Note are also [available on Zenodo](https://doi.org/10.5281/zenodo.14898462)."
