---
name: semisynthetic-trio-variant-simulation
description: 'Use when benchmarking causal variant prioritization or filtering tools
  against ground truth: you need a trio WES dataset where causal variants, disease
  phenotype, and inheritance mode are known by construction.'
when_to_use_negative:
- The goal is real clinical diagnosis rather than benchmarking — the article notes
  the benchmark relies on semisynthetic trios rather than fully real disease cohorts
- The analysis targets CNVs — CNV analysis is currently being integrated in eDiVA
  and is not yet available for this spike-in workflow
- Input data already contains known causal variants confirmed by segregation, making
  spike-in simulation unnecessary
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: eDiVA
  role: 'Pipeline executing the semisynthetic benchmark: eDiVA-Predict (variant calling),
    eDiVA-Annotate + eDiVA-Score (annotation and random-forest ranking), eDiVA-Prioritize
    (inheritance- and HPO-based shortlisting)'
  repo: https://github.com/mbosio85/ediva
- name: ClinVar
  role: Source of the thousands of known pathogenic variants spiked into the trio
    WES data to create disease cases with known ground truth
- name: 1000 Genomes Project CEPH trio
  role: Real WES data (NA12878, NA12891, NA12892) providing the realistic sequencing
    background into which pathogenic variants are introduced
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1002/humu.23772
    title: eDiVA—Classification and prioritization of pathogenic variants for clinical
      diagnostics
  source_content_hash: sha256:eaf3ff0f21bce36360ed497ab0e946aed9350defb367ad90c6de840d599f51a5
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mbosio85/ediva
  tools:
  - 1000 Genomes Project CEPH trio
  - ClinVar
  - eDiVA
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 12
---

# semisynthetic-trio-variant-simulation

## Summary

Constructs a highly realistic benchmark by spiking thousands of known pathogenic ClinVar variants into real WES data from the 1000 Genomes CEPH parent-child trio (NA12878, NA12891, NA12892), generating 6,811 semisynthetic disease trios with known causal variants for evaluating variant prioritization pipelines.

## When to use

Use when benchmarking causal variant prioritization or filtering tools against ground truth: you need a trio WES dataset where causal variants, disease phenotype, and inheritance mode are known by construction.

## When NOT to use

- The goal is real clinical diagnosis rather than benchmarking — the article notes the benchmark relies on semisynthetic trios rather than fully real disease cohorts
- The analysis targets CNVs — CNV analysis is currently being integrated in eDiVA and is not yet available for this spike-in workflow
- Input data already contains known causal variants confirmed by segregation, making spike-in simulation unnecessary

## Inputs

- WES fastq files (read1/read2) from the 1000 Genomes CEPH trio NA12878, NA12891, NA12892
- ClinVar pathogenic variant list to spike into the trios
- FAMILY_INFO file (concatenated sample_info.txt with AFFECTED column, 0/1)
- Pedigree structure (trio) and chosen inheritance modes (dominant_denovo, recessive, compound)

## Outputs

- 6,811 semisynthetic disease trios with joint multisample VCFs (combined.variants.supplement.vcf) and pedigree.tree files
- Ranked variant tables per trio (combined.variants.supplement.ranked.csv)
- Filtered candidate shortlists per inheritance mode (e.g. combined.variants.supplement.filtered.dominant_denovo.csv)
- Benchmark metrics: recall of the spiked causal variant and average false positives per inheritance mode

## How to apply

Run eDiVA-Predict per trio sample (bwa-mem alignment, GATK best-practices variant calling) on real WES fastq from the 1000 Genomes CEPH trio, then spike ClinVar pathogenic variants into each trio to simulate disease cases. Join samples into joint multisample calls (eDiVA_prepare_for_annotation) to obtain combined.variants.supplement.vcf and a pedigree.tree per trio. Annotate and score with eDiVA-Annotate + eDiVA-Score (random forest, 1000 trees, five-fold cross validation) to get combined.variants.supplement.ranked.csv, then run eDiVA-Prioritize per inheritance mode — recessive homozygous (3,353 variants), dominant de novo (2,592), recessive compound heterozygous (866) — using the lenient default filter setting (Table S1). Score candidates by whether the spiked causal variant appears in the filtered shortlist (recall) and by average false positives, e.g. comparing HPO-based prioritization against an empty HPO list.

## Related tools

- **eDiVA** (Pipeline executing the semisynthetic benchmark: eDiVA-Predict (variant calling), eDiVA-Annotate + eDiVA-Score (annotation and random-forest ranking), eDiVA-Prioritize (inheritance- and HPO-based shortlisting)) — https://github.com/mbosio85/ediva
- **ClinVar** (Source of the thousands of known pathogenic variants spiked into the trio WES data to create disease cases with known ground truth)
- **1000 Genomes Project CEPH trio** (Real WES data (NA12878, NA12891, NA12892) providing the realistic sequencing background into which pathogenic variants are introduced)

## Examples

```
nextflow run ~/ediva/edivatools-code/Predict/eDiVA-Predict.nf --NAME NA12878 --READ1 fastq.read1.gz --READ2 fastq.read2.gz --AFFECTED 1 --CPU 2 --OUTF output_folder/ -w work_folder/
```

## Evaluation signals

- The number of regenerated semisynthetic trios matches ~6,811 and each trio carries a pedigree.tree plus combined.variants.supplement.vcf
- Per-mode spike-in counts are recovered as reported: recessive homozygous 3,353 variants, dominant de novo 2,592, recessive compound heterozygous 866
- Recall check: the spiked ClinVar causal variant appears in the filtered shortlist for each inheritance mode
- HPO-enabled runs should show a two-fold reduction in false positives versus empty-HPO runs across all tested inheritance modes, with at most a mild recall reduction
- Filtered output files (combined.variants.supplement.filtered.<mode>.csv) are sorted by eDiVA-Score and carry an inheritance-compatibility annotation column

## Limitations

- The benchmark relies on semisynthetic trios rather than fully real disease cohorts, so findings may not transfer to real clinical cases
- Incomplete maps of known genotype-phenotype (or gene-phenotype) relations in public databases lead to a mild reduction in recall when HPO-based prioritization is enabled
- CNV analysis is not yet available in eDiVA, so spiked causal variants are limited to point mutations/indels within the exome kit BED regions

## Evidence

- [discussion] a highly realistic benchmark test combining real WES data of a parent-child trio with thousands of pathogenic ClinVar variants to generate 6,811 semisynthetic disease trios: "a highly realistic benchmark test combining real WES data of a parent–child trio with thousands of pathogenic ClinVar variants to generate 6,811 semisynthetic disease trios"
- [intro] semisynthetic benchmark study introducing thousands of known disease variants: "We demonstrate that eDiVA outperforms competing approaches in a semisynthetic benchmark study introducing thousands of known disease variants"
- [readme] This step describes how to launch eDiVA predict on all new samples coming out of  a sequencing machine.: "This step describes how to launch eDiVA predict on all new samples coming out of  a sequencing machine."
- [methods] By default, eDiVA applies a lenient filter setting defined in Table S1: "By default, eDiVA applies a lenient filter setting defined in Table S1"
- [readme] All variants annotated with an extra column explaining if they are compatible with the inheritance pattern provided: "All variants annotated with an extra column explaining if they are compatible with the inheritance pattern provided"
