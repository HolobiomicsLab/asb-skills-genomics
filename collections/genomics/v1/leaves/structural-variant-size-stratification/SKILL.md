---
name: structural-variant-size-stratification
description: Use when Truvari bench/refine ga4gh VCFs from a long-read SV caller comparison
  are available and you need to know whether a caller's F1 advantage holds within
  specific SV length bins, not just genome-wide.
when_to_use_negative:
- Input is a raw caller VCF rather than Truvari ga4gh-annotated truth/query VCFs with
  TP/FP/FN decision annotations — no per-stratum F1 can be computed without benchmark
  decisions
- The goal is a genome-wide (aggregate) accuracy comparison only; stratification is
  unnecessary and adds size-binning bookkeeping
- The comparison uses a benchmark engine other than the paper's bench/refine consolidation
  (e.g. hap-eval), whose match semantics differ from Truvari 4.2.2 decisions
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: truvari
  role: Produces the ga4gh TP/FP/FN decision annotations via the bench and refine
    workflow whose results are stratified by SV size
  repo: library://ctsa/truvari/truvari:4.2.2
- name: sawfish
  role: SV caller under evaluation; its per-stratum F1 is compared against Sniffles2
    and pbsv
  repo: https://github.com/PacificBiosciences/sawfish
- name: pbsv
  role: Comparison caller; second-best at 50-499 bp and >=5000 bp strata in the reported
    results
  repo: https://github.com/PacificBiosciences/pbsv
- name: bcftools
  role: Pre-filters benchmark/caller VCFs to SVTYPE-declared structural variant records
    before benchmarking
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioinformatics/btaf136
    title: 'Sawfish: improving long-read structural variant discovery and genotyping
      with local haplotype modeling'
  source_content_hash: sha256:314acc734fb2f6896f923fa02d7c02fa291d1346b287421c8ee82848c294038d
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
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# structural-variant-size-stratification

## Summary

Benchmark SV caller accuracy as a function of variant size by binning Truvari-benchmarked TP, FP, and FN calls into 50-499 bp, 500-4999 bp, and >=5000 bp strata and computing per-stratum precision, recall, and F1 to reveal whether a caller (e.g. sawfish) leads across all size groups rather than only in aggregate.

## When to use

Use when Truvari bench/refine ga4gh VCFs from a long-read SV caller comparison are available and you need to know whether a caller's F1 advantage holds within specific SV length bins, not just genome-wide.

## When NOT to use

- Input is a raw caller VCF rather than Truvari ga4gh-annotated truth/query VCFs with TP/FP/FN decision annotations — no per-stratum F1 can be computed without benchmark decisions
- The goal is a genome-wide (aggregate) accuracy comparison only; stratification is unnecessary and adds size-binning bookkeeping
- The comparison uses a benchmark engine other than the paper's bench/refine consolidation (e.g. hap-eval), whose match semantics differ from Truvari 4.2.2 decisions

## Inputs

- Consolidated Truvari ga4gh truth VCF (e.g. ga4gh_with_refine_truth.vcf.gz) per caller
- Consolidated Truvari ga4gh query VCF (e.g. ga4gh_with_refine_query.vcf.gz) per caller (sawfish, pbsv, Sniffles2)
- 32x HG002 GIAB T2T benchmark assessment results from Zenodo doi:10.5281/zenodo.14898462 (truvari 4.2.2 with MAFFT refinement)

## Outputs

- size_stratified_f1_rankings.csv: per-stratum (50-499 bp, 500-4999 bp, >=5000 bp) caller-by-metric F1 table mirroring Supplementary Table S2
- Per-stratum TP/FP/FN counts, precision, recall, and F1 for each caller

## How to apply

Load the consolidated Truvari ga4gh truth and query VCFs (e.g. ga4gh_with_refine_truth.vcf.gz and ga4gh_with_refine_query.vcf.gz) for each caller from the 32x HG002 T2T assessment results in the Zenodo deposit (doi:10.5281/zenodo.14898462). Size each benchmark-qualified variant as END - POS for non-BND records using the --dup-to-ins convention (duplications sized as insertions), and assign every TP, FP, and FN call to one of three strata: 50-499 bp, 500-4999 bp, and >=5000 bp. Aggregate per-stratum TP/FP/FN counts from the Truvari decision annotations, compute precision, recall, and their harmonic mean (F1) per stratum, and rank callers within each bin. The rationale is that the paper's headline claim — sawfish has the highest accuracy across every tested SV size group — is only verifiable with this stratification, which in the paper yields sawfish leads of 0.5 percentage points over pbsv at 50-499 bp and 2.6 points over Sniffles2 / 7.0 points over pbsv at >=5000 bp. Write the caller-by-metric F1 table (mirroring Supplementary Table S2) to size_stratified_f1_rankings.csv.

## Related tools

- **truvari** (Produces the ga4gh TP/FP/FN decision annotations via the bench and refine workflow whose results are stratified by SV size) — library://ctsa/truvari/truvari:4.2.2
- **sawfish** (SV caller under evaluation; its per-stratum F1 is compared against Sniffles2 and pbsv) — https://github.com/PacificBiosciences/sawfish
- **pbsv** (Comparison caller; second-best at 50-499 bp and >=5000 bp strata in the reported results) — https://github.com/PacificBiosciences/pbsv
- **bcftools** (Pre-filters benchmark/caller VCFs to SVTYPE-declared structural variant records before benchmarking)

## Examples

```
bcftools view -i 'INFO/SVTYPE!="."' -Oz -o GRCh38_HG2-T2TQ100-V1.1_stvar.svtype.vcf.gz
```

## Evaluation signals

- Computed F1 ranking places sawfish first in all three size strata (50-499 bp, 500-4999 bp, >=5000 bp)
- Reproduces the reported leads within rounding tolerance: 0.5 percentage points vs pbsv at 50-499 bp; 2.6 points vs Sniffles2 and 7.0 points vs pbsv at >=5000 bp
- Every benchmark-qualified variant is assigned to exactly one size stratum with no records dropped (BNDs excluded from sizing per the END - POS non-BND rule; DUPs sized as insertions via --dup-to-ins)
- Per-stratum F1 values are consistent with the aggregate TP/FP/FN counts (weighted recombination of strata approximately recovers overall F1)
- Output CSV matches the Supplementary Table S2 layout (caller x metric within each size bin)

## Limitations

- Sizing uses END - POS for non-BND records only, so BND/translocation records fall outside the length-stratified comparison
- Results depend on the paper's specific consolidation: the T2T assessment requires truvari bench plus refine with MAFFT, not bench alone (unlike the CMRG assessment); using bench alone changes the match decisions and thus per-stratum counts
- Benchmark restricted to autosomes (confident regions filtered with grep -v chr[XY]) and to SVTYPE-declared records, so sex chromosomes and undeclared-SVTYPE records are not represented in the strata

## Evidence

- [abstract] Assessment against the draft Genome in a Bottle (GIAB) SV benchmark from the T2T-HG002-Q100 diploid assembly shows that sawfish has the highest accuracy: "Assessment against the draft Genome in a Bottle (GIAB) SV benchmark from the T2T-HG002-Q100 diploid assembly shows that sawfish has the highest accuracy"
- [methods] All assessment below uses truvari 4.2.2 with MAFFT for refinement.: "All assessment below uses truvari 4.2.2 with MAFFT for refinement."
- [methods] it is important to use the bench and refine command, with the bench/refine results consolidated as described further below: "it is important to use the bench and refine command, with the bench/refine results consolidated as described further below"
- [results] Sawfish has the highest F1-score in every SV size stratum, with a modest 0.5% lead over the next-best method pbsv for 50-499 bp SVs: [evidence span withheld — anchor status: not_found]
- [readme] Sawfish is a joint structural variant (SV) and copy number variant (CNV) caller for mapped HiFi sequencing reads.: "Sawfish is a joint structural variant (SV) and copy number variant (CNV) caller for mapped HiFi sequencing reads."
